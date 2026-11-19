# 6. Operations

## Applying configuration changes

1. Edit OSP's `ism-configuration.properties`, either with ConfigUpdate or directly. Back it up first.
2. If the change involves a client ID or secret, or the OSP URL, make the **same** change in the consuming app's `ism-configuration.properties` (UA, Reporting, SSPR or Forms).
3. Restart OSP:
   - Kubernetes: `kubectl -n idm rollout restart deploy/osp`
   - Other installs: `systemctl restart netiq-tomcat`
4. Verify:
   - `curl https://<host>/osp/a/idm/auth/app/ping` returns `pong`.
   - Discovery returns the expected issuer.

OSP can also restart itself through the admin API (`POST /osp/s/restart` with an administrator bearer token). ConfigUpdate's NAM auto-setup does this. A pod restart is simpler and reloads everything.

## Health checks

| Check | URL / command | Notes |
|---|---|---|
| Liveness | `GET /osp/a/idm/auth/app/ping` → `pong` | No auth |
| Admin liveness | `GET /osp/s/ping`, `GET /osp/s/livenessCheck` | No auth |
| Health detail | `GET /osp/s/health`, `/osp/s/idm/health` | Admin |
| Discovery | `GET /osp/a/idm/auth/oauth2/.well-known/openid-configuration` | No auth. The best "OSP is really serving" check. |

The Helm chart's readiness probe only runs `cat catalina.out`, and its `helm test` uses plain HTTP against a TLS port. Neither tells you OSP works. If you maintain your own values, consider an `httpGet` probe on `/osp/a/idm/auth/app/ping` (scheme HTTPS, port 8543).

## Logging

| What | Where / how |
|---|---|
| Tomcat console | `/opt/netiq/idm/apps/tomcat/logs/catalina.out` (container stdout follows it). The previous run is appended to `catalina-back.out`. |
| OSP log | Logger `osp-idm`, in `-Dcom.netiq.idm.osp.logging.file.dir` (`tomcat/logs`). Ten files of 10 MB, zipped on rotation. |
| Level (static) | `-Dcom.netiq.idm.osp.logging.level=WARN` in `setenv.sh`. Use `DEBUG` or `TRACE` when troubleshooting. At DEBUG, the property translator also prints every effective property, with secrets masked. |
| Exceptions | `-Dcom.netiq.idm.exception.log.level=…` |
| Level (runtime) | `GET`/`PUT /osp/s/loglevel`, `/osp/s/verboseLogging`, `/osp/s/revertLogging` (admin) |
| Access log | `tomcat/logs/localhost_access_log.<date>.txt` (common format) |
| First-run configuration | `/config/osp/log/idmconfigure.log` |

## Key and certificate rotation

OSP uses **one RSA-2048 key** (alias `osp` in `/opt/netiq/idm/apps/osp/osp.jks`, PKCS#12, valid 730 days) for all of the following:
- signing JWT access tokens and id_tokens
- decrypting JWE
- SAML SP signing and encryption

On idm254 it expires **2028-09-07**.

Rotation options:

1. **Built-in rollover:** `POST /osp/s/idm/rollkeys` (admin). OSP generates new keys and keeps the old ones for validation. Remove them later with `/osp/s/idm/deleterolledkeys`.
   - Set `com.netiq.idm.osp.login.saml2.rolledKeysInMetadata=true` if a SAML IdP must see both keys during the changeover.
   - Behaviour after a restart (are rolled keys persisted, or regenerated from `osp.jks`?) has not been verified. Test in a lab first.
2. **Replace the keystore:**
   1. Generate a new key, preferably with the ConfigUpdate `1:n` layout (separate `osp-signing`, `osp-encryption` and `osp-tls` aliases):

      ```bash
      keytool -genkeypair -keyalg RSA -keysize 3072 -alias osp -validity 730 \
        -storetype pkcs12 -keystore /config/osp/osp/osp.jks.new -dname "cn=idm254.pointbluetech.com"
      ```

   2. Swap the file. If the password changed, update `com.netiq.idm.osp.oauth-keystore.pwd` / `oauth-key.pwd`.
   3. Restart OSP.
   4. Re-import OSP's SAML metadata at the IdP, if SAML is in use.
   5. Clients pick up the new key through `jwks`. The IDM apps validate tokens through OSP, so no app changes are needed.

**Other certificates to watch:**

| Certificate | Location | Renewal |
|---|---|---|
| Tomcat TLS | `tomcat.ks`, alias `ospservercert` in Kubernetes | From secret `<release>-osp-server-certificate`, imported at pod start |
| Ingress certificate in the truststore | `idm.jks`, alias `ingresscert` | Refreshed only at pod start. After cert-manager renews the ingress certificate, restart OSP, UA and Reporting so server-to-server calls keep trusting it. |
| eDirectory CA | `idm.jks` | Needed for LDAPS |

## Backup

Back up the following:
- the whole `/config/osp` tree;
- the matching `/config/userapp/tomcat/conf/ism-configuration.properties` and `encrypt-keys.pkcs12`;
- the `MASTER_KEYSTORE_PWD` and keystore passwords, which are stored in the Kubernetes secrets.

Without `encrypt-keys.pkcs12` and its password, the `ENCRYPT`ed values cannot be decrypted.

## Common errors

| Error | Meaning |
|---|---|
| `XDAS_OUT_ENTITY_NON_EXISTANT` "Endpoint not found" | Wrong path (for example `oauth2/getkeys`; the right one is `oauth2/jwks`) |
| `XDAS_OUT_POLICY_VIOLATION` "Not allowed" | Wrong HTTP method (for example GET on `token`) |
| `XDAS_OUT_INVALID_INPUT` "No handler defined for protocol" | Protocol not enabled (SAML) |
| `{"error":"invalid_request"}` on `grant` / `userinfo` | Missing parameters or bearer token |
| `failure [temporary session]` from `app/activity` | No OSP session cookie; normal for an anonymous request |
| Login works but the IDM app loops back to login | `com.netiq.idm.osp.url.host` or the client `redirect.url` points at an internal name (`osp:8543`, `identityapplications:8543`), or the client secrets differ between OSP and the app |
| Container exits with "Configured data layer has been run with version later than…" | The image is older than the volume's `version.properties`. Use the matching or a newer image. |
