# 2. Container Image and Kubernetes Deployment

This chapter explains how the OSP container image is built, what happens when it starts, where state lives, and how the vendor Helm chart deploys it. Everything here comes from the image `osp:idm-4.10.2-591` (in `Identity_Manager_4.10.2_Containers.tar.gz`), the `identity-manager-1.7.0` Helm chart, and the Ansible role shipped in the same tarball.

## 2.1 Image anatomy

| Item | Value |
|---|---|
| Image tag (tarball) | `osp:idm-4.10.2-591` (build 591) |
| Image tag (running in lab) | `osp:idm-4.10.2.0200-81` (4.10.2 Patch 2) |
| Base | Red Hat UBI 9 minimal (`ubi9-minimal` 9.6), OpenText-hardened (`com.opentext.hardened.tag=redhat:9.202510.6594312-minimal`) |
| Entrypoint | `/bin/bash start.sh` |
| Healthcheck (Docker) | `ps -eaf \| grep -i "server.xml start"`, which only checks that the Tomcat process exists |
| Default env | `debug=n` (set `debug=y` to run the start scripts with `set -x`) |
| OSP | `netiq-osp-7.1.0-0.noarch` RPM, WAR `osp.war` 7.1.0 (built 2025-06-12, branch `origin/osp_7.1.0`, commit `48a39272`) |
| Servlet container | Apache Tomcat 10.1.48 in the tarball, 10.1.55 in the lab patch image (`-Dinternal.osp.container.plugin.server-version`) |
| JRE | `/opt/netiq/common/jre` (bundled; JDK 17 class level) |
| Run-as user | `novlua` (uid/gid 998); root only during configuration |

### Layers

The image has 8 layers. Only four of them matter:

| Layer | Content |
|---|---|
| `6874c0a7…` (567 MB) | The **whole IDM installer tree** at `/idm` plus the **pre-installed** OSP/Tomcat/JRE under `/opt/netiq` |
| `1c2ad5ca…`, `174b93d4…`, `ceca7597…`, `9caa3b29…` | `startosp.sh`, `start.sh`, `commonfunctions.sh` (the last layer re-adds all three) |

OSP is installed when the image is built, but **configured on the first container start**. The `/idm` installer stays in the image for that purpose. `startosp.sh` deletes `/idm/common/lib/` after configuration ("to mitigate vulnerabilities reported during runtime").

### Filesystem layout (inside the container)

```
/opt/netiq/common/jre/                      bundled JRE
/opt/netiq/idm/apps/osp/
    bin/configutil.sh, bin/lib/             OSP configuration utility launcher
    lib/netiq-configutil.jar                configutil (GUI/console/script)
    lib/osp-conf-edir.jar                   <- active "ext-context" (eDirectory + file auth)
    lib/osp-conf-ad.jar                     alternative: Active Directory + file
    lib/osp-conf-fileonly.jar               alternative: file-only
    osp-extras/l10n-resources/osp-custom-resource.jar   branding / string overrides
    osp-extras/schema/osp.sch               eDirectory schema extension
    osp-extras/sentinel-collector/          Sentinel collector for OSP audit events
    conf/            -> /config/osp/osp/conf   (symlink created at start)
    osp.jks          -> /config/osp/osp/osp.jks
/opt/netiq/idm/apps/tomcat/                 CATALINA_BASE
    webapps/osp.war
    conf/            -> /config/osp/tomcat/conf
    logs/            -> /config/osp/tomcat/logs
    cache/           -> /config/osp/tomcat/cache    (CEF audit cache)
    bin/setenv.sh    -> /config/osp/tomcat/bin/setenv.sh
    bin/startUA.sh, bin/shutdownUA.sh       start/stop wrappers (also used by UA/RPT images)
/opt/netiq/idm/apps/configupdate/           configupdate.sh (the IDM "ConfigUpdate" tool)
    configupdate.sh.properties -> /config/osp/configupdate/configupdate.sh.properties
/opt/netiq/idm/tomcat/                      a second, vanilla Tomcat tree (ROOT webapp only; unused by OSP)
/idm/                                       installer (osp/, common/, user_application/, orchestration/…)
/var/opt/netiq/idm/log/ -> /config/osp/log  idmconfigure.log
```

## 2.2 Start-up sequence

```
start.sh
 ├─ source /commonfunctions.sh         (replica coordination, picks CONF_HOME)
 ├─ symlink default dirs → /config/osp/…  (copy defaults only if volume not yet configured)
 ├─ rotate catalina.out → catalina-back.out
 ├─ trap SIGTERM → shutdownUA.sh (graceful Tomcat stop), clear lock entries
 └─ /startosp.sh &
      ├─ if tomcat/conf/ism-configuration.properties does NOT exist  → FIRST-RUN CONFIGURATION
      │     wait for https://$IDM_ACCESS_VIA_SINGLE_DOMAIN:443 (up to 10 × 1 min)
      │     configureosp:  env > /tmp/silent-<ts>.properties ; append $SECRET_PROPERTY_PATH
      │                    /idm/osp/configure.sh -s -ssc -slc -f <silent file>
      │     ismPropertiesChangeUAandRPT, addopens…, RemoveAJPConnector
      │     kubernetes_configuration osp   (rewrite URLs to ingress host, import TLS certs)
      │     addSimpleTcpCluster            (only if OSP_REPLICA_COUNT > 1)
      │     addismsensitivefile            (create encrypt-keys.pkcs12 + ism-sensitive.properties)
      │     addcommicrofocusismmasterTosetenv, addpropswithconfigupdate
      │     resetencryption                (configupdate.sh --silent with sso-configuration=auto)
      │     copy edited files to /config, back-link, chown novlua, restart Tomcat
      └─ else → RESTART / UPGRADE PATH
            compare /config/osp/version.properties (novell-DXMLengnx version) with /idm/version.properties
            same     → re-create back-links, start Tomcat
            older    → run upgrade functions (same list as above, plus setSSLHostConfig)
            newer data than image → refuse to start ("Configured data layer has been run with version later…")
            below SUPPORTED_DOCKER_IDM_VERSION → refuse to start
      tail -f catalina.out   (or $LOGTOFOLLOW)
```

**Key point:** first-run configuration happens only once per persistent volume. It is keyed on whether `ism-configuration.properties` exists in the volume. To reconfigure from scratch, you have to remove the OSP state under `/config/osp` (back it up first). Changing Helm values and restarting the pod alone will not regenerate the configuration.

### What `/idm/osp/configure.sh` does (first run)

1. Loads the prompt definitions (`/idm/common/conf/prompts.conf`) and reads the answers from the silent file. In Kubernetes that file is the container environment plus the secret properties.
2. `import_vault_certificates`: trusts the eDirectory CA.
3. `configure_props`:
   - Expands `/idm/osp/conf/osp_init.properties`, which holds `add-property GLOBAL …` statements with `___PLACEHOLDER___` tokens. See [03-configuration-reference.md](03-configuration-reference.md).
   - Runs `configinit.sh` (`netiq-configutil.jar -useDb false -script`). It writes `GLOBAL-<n>.properties`, which is then concatenated into `/opt/netiq/idm/apps/osp/conf/global.properties`.
   - Runs `configupdate_idm` and `update_configupdate_script` to populate `configupdate.sh.properties`.
   - Runs `netiq-configutil.jar -script import-configs.script` (`import-external NODE global.properties`). It imports the settings into the active `ism-configuration.properties`.
4. `create_osp_keystore`: `keytool -genkey -keyalg RSA -keysize 2048 -alias osp -validity 730 -dname cn=$ID_VAULT_HOST -storetype pkcs12` into `osp.jks`. **This key signs (and encrypts) OAuth tokens. It expires after 2 years.**
5. `modify_server_xml`: sets the HTTPS connector port to `SSO_SERVER_SSL_PORT`, restricts TLS to `TLSv1.2+TLSv1.3`, adds `SSLHostConfig`, and removes AJP.
6. Turns off OSP's native auditing (`-Dcom.netiq.idm.osp.audit.enabled=false`).
7. `changeToCustomName`: writes `OSP_CUSTOM_NAME` into `osp-custom-resource.jar` (`OIDPENDUSER.LoginProductName`) and appends a CSS rule to the WAR.
   - In container mode, it also encrypts every client secret with `CryptUtil` and the master key.
   - It deletes the client blocks of components that are not installed (RPT, SSPR, UA, edirapi).
8. If `CUSTOM_OSP_CERTIFICATE=n`, it builds `tomcat.ks` from a certificate minted by the eDirectory CA (`create_ks_from_kmo`, KMO `dirxml`).
9. Adds the setenv options (opens, CRLDP, logback), imports the base containers LDIF (or `CUSTOM_CONTAINER_LDIF_PATH`), and chmods `server.xml`, `ism-configuration.properties` and `ism-sensitive.properties` to `600`.

## 2.3 Persistent state (`/config`)

All OSP state lives on the shared PVC, mounted at `/config`:

| Path on volume | Purpose |
|---|---|
| `/config/osp/tomcat/conf/ism-configuration.properties` | **The runtime configuration** (`-Dcom.netiq.ism.config`). Its presence means "already configured". |
| `/config/osp/tomcat/conf/ism-sensitive.properties` | Obscured master keystore pass/keypass used to decrypt `ENCRYPT` values |
| `/config/osp/tomcat/conf/encrypt-keys.pkcs12` | Master key store for `_attr_obscurity = ENCRYPT` values |
| `/config/osp/tomcat/conf/tomcat.ks` | HTTPS connector keystore (alias `ospservercert` in Kubernetes) |
| `/config/osp/tomcat/conf/idm.jks` | Trust store: eDirectory CA, ingress certificate (`ingresscert`), server cert |
| `/config/osp/tomcat/conf/server.xml` | Tomcat connectors (HTTPS 8543 in Kubernetes) |
| `/config/osp/tomcat/bin/setenv.sh` | JVM flags (see [03](03-configuration-reference.md#33-jvm-system-properties-setenvsh)) |
| `/config/osp/tomcat/logs/` | `catalina.out`, `localhost_access_log.*.txt`, OSP logs |
| `/config/osp/tomcat/cache/` | CEF audit spool |
| `/config/osp/osp/osp.jks` | **OAuth signing/encryption key** (alias `osp`) |
| `/config/osp/osp/conf/global.properties`, `GLOBAL-*.properties` | Output of configutil script. **Not** rewritten by the Kubernetes URL rewrite; it may hold in-cluster URLs. |
| `/config/osp/configupdate/configupdate.sh.properties` | ConfigUpdate tool settings |
| `/config/osp/log/idmconfigure.log` | First-run configuration log. **Read this first when a fresh deployment fails.** |
| `/config/osp/version.properties` | RPM manifest of the image that last configured the volume; used for upgrade decisions |
| `/config/osp/controlling`, `/config/osp/transaction-lock-file` | Replica coordination (see below) |
| `/config/osp2…/config/ospN` | Per-replica data layers when `OSP_REPLICA_COUNT > 1` |

## 2.4 Multiple replicas

OSP does not use Kubernetes-native leader election. `commonfunctions.sh` coordinates replicas through files on the shared RWX volume:

- Each pod generates a 14-digit random ID and appends it to `/config/osp/transaction-lock-file`. If its ID is first, the pod configures `/config/osp`. Otherwise it waits until the first instance's `catalina.out` shows `Deploying web application archive` and `using server version: <osp rpm version>`, and `osp.jks` exists.
- Additional replicas use `/config/osp2…/config/ospN`. They copy `osp`, `log`, `tomcat` and `version.properties` from the first instance, but **link `tomcat/conf` to the first instance**, so the configuration and keys are shared.
- When `OSP_REPLICA_COUNT > 1`, `addSimpleTcpCluster` adds a Tomcat `SimpleTcpCluster` to `server.xml`, with multicast membership `228.0.0.4:45564`, receiver port 4000 and DeltaManager session replication. **Multicast is usually not available on Kubernetes CNIs.** In practice, session stickiness comes from the ingress cookie affinity annotations in the chart (`nginx.ingress.kubernetes.io/affinity: cookie`).
- The Deployment strategy is `Recreate`.
- On SIGTERM, the pod's own ID is removed from the lock file.

## 2.5 Helm chart (`identity-manager-1.7.0`)

### Objects created for OSP

| Object | Notes |
|---|---|
| `Deployment/osp` | 1 pod: init container `init-osp` (image `identityutils`) and main container `osp` |
| `Service/osp` | `ClusterIP`, port `8543` → `http` (the name is misleading: the port is HTTPS) |
| `HorizontalPodAutoscaler/osp` | Only if `osp.autoscaling.enabled`. Uses the deprecated `autoscaling/v2beta1` API. |
| `Ingress/<release>-identity-manager` | Path `/osp(/|$)(.*)` → `osp:8543`, backend protocol HTTPS, regex paths, cookie affinity |
| `templates/tests/test-osp-connection.yaml` | `helm test` hook: busybox `wget osp:8543`. This is plain HTTP against a TLS port, so the test is not a meaningful health check. |

> Note: the ingress also routes `/ospconfig` and `/oauth`, but they go to **formrenderer** (port 8600), not to OSP. Despite its name, `/ospconfig` is served by the Forms renderer.

### Init container `init-osp`

1. Waits for eDirectory: `/dev/tcp/$ID_VAULT_HOST/$ID_VAULT_HEALTH_CHECK_PORT`.
2. Waits until `$IDM_ACCESS_VIA_SINGLE_DOMAIN:443` answers TLS.
3. Writes `/shared-conf/secret.properties` from the mounted secret (`idm-common-password` or the Azure/AWS secret store) plus `identity-manager-key-store-pwd`.
4. Copies the OSP server certificate and key from secret `<release>-osp-server-certificate`, and the ingress certificate from the ingress TLS secret, into the `shared-osp-cert` emptyDir. The main container sees it as `/config/certificates`.
5. Writes `DATA_CONTAINERS_LDIF` to `/shared-conf/data_container.ldif`.

### Main container

- Env: `SECRET_PROPERTY_PATH=/shared-conf/secret.properties` and `CUSTOM_CONTAINER_LDIF_PATH`, plus about 150 shared `*_` variables rendered from `values.yaml` by `identity-manager.configuration.envs`. Values that come from secrets appear with a `secret___` prefix. `envFrom: <release>-deployment-data` supplies `OSP_REPLICA_COUNT` and the Helm metadata.
- Mounts: PVC → `/config`, emptyDir `shared-osp-cert` → `/config/certificates`, emptyDir `shared-conf` → `/shared-conf`.
- Readiness probe: `cat /opt/netiq/idm/apps/tomcat/logs/catalina.out` (period 10 s, failureThreshold 180). **It only checks that the log file exists, not that OSP serves requests.** There are no liveness or startup probes.
- Resources (chart default): requests 1 CPU / 2 Gi, limits 1.5 CPU / 3 Gi.

### OSP-specific values

```yaml
osp:
  deploy: true
  replicaCount: 1
  configuration:
    OSP_CUSTOM_NAME: "Identity Access"   # product name on the login page
    SSO_SERVICE_PWD:                     # shared client secret for ALL built-in OAuth clients
      secret:
    MASTER_KEYSTORE_PWD:                 # password of encrypt-keys.pkcs12
      secret:
  resources: {...}
```

Other inputs that drive OSP come from the shared configuration:

- `ID_VAULT_*`: LDAP connection and admin DN.
- `USER_CONTAINER` / `ADMIN_CONTAINER`: user search base and admin base.
- `SSO_SERVER_HOST` / `SSO_SERVER_SSL_PORT`: the internal service name and port, `osp` / `8543`.
- `IDM_ACCESS_VIA_SINGLE_DOMAIN`: the external ingress host. **Every internal URL in ism-configuration.properties is rewritten to it.**
- `CUSTOM_OSP_CERTIFICATE`, `COMMON_KEYSTORE_PWD`, `OSP_KEYSTORE_PWD`, `EXTERNAL_SSO_SERVER`, `TOMCAT_HTTP_PORT`, `SSO_SERVER_PORT`.

### URL rewrite to the ingress host (`kubernetes_configuration osp`)

When `KUBERNETES_ORCHESTRATION=y` and `KUBE_INGRESS_ENABLED=true`, `/idm/common/scripts/kube_utils.sh` runs `sed` over `tomcat/conf/ism-configuration.properties`. It replaces:

```
https://$SSO_SERVER_HOST:$SSO_SERVER_SSL_PORT        (e.g. https://osp:8543)
https://$UA_SERVER_HOST:$UA_SERVER_SSL_PORT
https://$FR_SERVER_HOST:$NGINX_HTTPS_PORT
https://$RPT_SERVER_HOSTNAME:$RPT_TOMCAT_HTTPS_PORT
https://$SSPR_SERVER_HOST:$SSPR_SERVER_SSL_PORT
        → https://$IDM_ACCESS_VIA_SINGLE_DOMAIN
```

This is why the issuer, redirect URIs, and `com.netiq.idm.osp.url.host` point at the public host. The rewrite then:

1. Imports `/config/certificates/tls.{crt,key}` into `tomcat.ks` and `idm.jks` as alias `ospservercert`.
2. Imports `/config/certificates/ingress-tls.crt` into `idm.jks` as `ingresscert`. Callers that load `idm.jks` then trust the public certificate on calls through the ingress.

> **Gotcha:** When the ingress certificate is renewed (for example by cert-manager or Let's Encrypt every 90 days), the copy in `idm.jks` is refreshed only when the pod restarts. Calls to OSP use two truststores ([03](03-configuration-reference.md#client-side-properties-that-share-the-comnetiqidmosp-prefix)). The UA and Reporting back-channel trusts OSP with `com.netiq.idm.osp.ssl-keystore.file` (default `tomcat.ks`). OSP's outbound TLS uses `com.netiq.idm.osp.oauth-truststore.file` (`idm.jks`). Server-to-server calls that depend on the `ingresscert` entry then fail TLS validation until those pods are restarted. Let's Encrypt chains to a public root, which the JRE `cacerts` usually trusts. Whether that fallback is used depends on which truststore the caller loaded.

## 2.6 Docker / Ansible deployment (non-Kubernetes)

The tarball also contains `ansible/roles/osp`. It:

1. Loads `IDM_4102_osp.tar.gz`.
2. Copies `silent.properties` and the OSP Tomcat keystore (`tomcat-osp-custom.ks`) into `osp_shared_volume`.
3. Runs the container with `SILENT_INSTALL_FILE=/config/silent.properties`, a fixed IP on a user-defined Docker network, and hostname `SSO_SERVER_HOST`.
4. Waits until `osp/log/idmconfigure.log` contains `Tomcat started`.
5. **Deletes `silent.properties` and the keystore from the volume** afterwards.

The intended readiness URL, commented out in the role, is `https://<osp>:<port>/osp/a/idm/auth/oauth2/.well-known/openid-configuration`.

## 2.7 Upgrades and patches

- The image compares `novell-DXMLengnx` in `/config/osp/version.properties` (volume) with the same entry in `/idm/version.properties` (image).
- A newer image runs the upgrade functions against the existing configuration. It does not re-run `configure.sh`.
- Patch images can ship `/idmpatch/common/scripts/*`. These take precedence over `/idm/...`.
- The OSP version itself (`netiq-osp` RPM) is checked only by replica coordination (`rpm -q netiq-osp`).
- Rolling back to an older image against an upgraded volume is refused.
