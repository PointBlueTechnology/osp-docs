# 3. Configuration Reference

OSP has **no admin UI of its own**. Its behaviour is controlled by four things:

| Layer | Where | Edited with |
|---|---|---|
| **Runtime properties** | `/opt/netiq/idm/apps/tomcat/conf/ism-configuration.properties` (path given by `-Dcom.netiq.ism.config`) | ConfigUpdate (`configupdate.sh`), `netiq-configutil.jar`, or a text editor |
| Secrets for `ENCRYPT` values | `tomcat/conf/encrypt-keys.pkcs12` + `tomcat/conf/ism-sensitive.properties` | ConfigUpdate / `CryptUtil` |
| JVM system properties | `tomcat/bin/setenv.sh` (`CATALINA_OPTS`) | text editor |
| Built-in OSP configuration ("ext-context") | `/opt/netiq/idm/apps/osp/lib/osp-conf-edir.jar` (or `-ad`, `-fileonly`), selected by `-Dinternal.osp.framework.ext-context-file` | not meant to be edited; it consumes the properties above |

> In the IDM container layout, OSP and the Identity Applications each have **their own** `ism-configuration.properties`:
> - OSP: `/config/osp/tomcat/conf/…`
> - UA: `/config/userapp/tomcat/conf/…`
> - Reporting: `/config/reporting/…`
>
> Several keys appear in more than one file, for example the client IDs and secrets and `com.netiq.idm.osp.url.host`. **The values must agree.** A client secret changed only in OSP's file breaks that client.

## 3.1 File format and value protection

`ism-configuration.properties` is a Java properties file with `key = value` lines. `${other.key}` references are expanded, as in `${com.netiq.idm.osp.url.host}/osp/a/idm/auth/oauth2`.

Sensitive values carry a companion key that says how they are stored:

```properties
com.netiq.idmdash.clientPass._attr_obscurity = ENCRYPT
com.netiq.idmdash.clientPass = <ciphertext>
```

| `_attr_obscurity` | Meaning |
|---|---|
| `ENCRYPT` | Encrypted with the master key in `encrypt-keys.pkcs12`. The keystore's passwords are themselves obscured in `ism-sensitive.properties`, located through the JVM flags `-Dcom.microfocus.ism.master-keystore`, `-Dcom.microfocus.ism.master-storetype=pkcs12`, `-Dcom.microfocus.ism.password-supplier=com.netiq.ism.obfuscate.supplier.ObscuredFileSupplier` and `-Dcom.microfocus.ism.master-pass-file`. |
| `NONE` | Plaintext |
| (other) | Legacy obscuring |

To produce an `ENCRYPT` value by hand, use the installer utility the IDM scripts use:

```bash
/opt/netiq/common/jre/bin/java -cp /idm/common/packages/utils/idm_install_utils.jar:/opt/netiq/idm/apps/configupdate/* \
  com.netiq.installer.utils.CryptUtil encrypt '<plaintext>' \
  -keystore /opt/netiq/idm/apps/tomcat/conf/encrypt-keys.pkcs12 -storepass <MASTER_KEYSTORE_PWD> -keypass <MASTER_KEYSTORE_PWD>
```

Note that `startosp.sh` deletes `/idm/common/lib` after first-run configuration. The `idm_install_utils.jar` path above lives under `/idm/common/packages/utils` and survives.

Simpler alternatives:
- Let ConfigUpdate do the encryption.
- Write the value with `_attr_obscurity = NONE` (plaintext; protected only by file mode `600`).

## 3.2 Runtime properties read by OSP

### How OSP reads the file

1. `ospcfg.xml` in the ext-context jar names `com.netiq.osp.uaconfig.PropertiesSupplier` (in `uaconfig-osp-7.1.0-4.jar`) as the property loader.
2. Keys starting with `com.netiq.idm.osp.` or `internal.osp.` pass through unchanged. `ENCRYPT` values are decrypted on demand.
3. *Generator* classes then **compute** additional keys and XML fragments from the user-facing keys: clients, keys, SAML2, Kerberos, SSPR, CEF, whitelists, attribute maps, HTTP interfaces.
4. The result fills the `${key:default}` placeholders in `ospcfg.xml`, `idm/tenantcfg.xml` and `idm/services/authcfg.xml`. The complete list of placeholders is in [Appendix B](B-runtime-property-placeholders.md). Resolution order: properties file, then Java `-D` property, then environment variable, then the default.

**Legacy User Application keys take precedence** whenever `DirectoryService/realms/jndi/params/AUTHORITY` is present, which is the normal IDM case. For the following settings, the UA keys win over the `com.netiq.idm.osp.*` equivalents:
- users container
- login attribute
- LDAP host, port and SSL
- search scope
- truststore
- admin name and password (`com.novell.idm.ldap.admin.user` / `.pass`)

If an OSP setting "doesn't take", check for a UA-style key in the same file.

**Debugging the effective configuration:** with `-Dcom.netiq.idm.osp.logging.level=DEBUG`, the translator logs every effective property, with secrets masked.

Defaults below come from `/idm/osp/conf/osp_init.properties` (the first-run template), ConfigUpdate's definitions ([Appendix A](A-configupdate-option-catalog.md)) and the OSP configuration XML ([Appendix B](B-runtime-property-placeholders.md)).

### 3.2.1 Server identity and URLs

| Key | Default / example | Meaning |
|---|---|---|
| `com.netiq.idm.osp.url.host` | `https://<SSO_SERVER_HOST>:<port>` | **Public base URL of OSP.** OSP builds its HTTP interfaces, issuer and SAML entity ID from it. It also accepts the host's IP, hostname and canonical name, plus `localhost` when `localhost-auto-add=true` and the host is local. If unset, the host is derived from `oauth.issuer`, then `client.authserver.url.authorize`, `rbpm.authserver.url.authorize`, `iac.authserver.url.authorize`. In Kubernetes it is rewritten to `https://<ingress host>`. |
| `com.netiq.idm.osp.oauth.issuer` | `${com.netiq.idm.osp.url.host}/osp/a/idm/auth/oauth2` | The issuer **the IDM client apps** use for discovery. OSP itself does **not** take its `iss` from this key. |
| `com.netiq.idm.osp.oauth.issuer-override` | none | **Forces the `iss` value** OSP puts in tokens and discovery. Without it, `iss` = `<interface>/osp/a/idm/auth/oauth2`. |
| `com.netiq.idm.osp.tenant.http-interfaces` | computed | Explicit list of interface URLs OSP answers on (overrides the `url.host` derivation) |
| `com.netiq.idm.osp.localhost-auto-add` | `true` | Also accept `localhost`, and add localhost redirect URI variants |
| `com.netiq.client.authserver.url.authorize` | `${url.host}/osp/a/idm/auth/oauth2/grant` | Authorize URL **used by clients** (legacy; clients now prefer discovery from the issuer) |
| `com.netiq.client.authserver.url.token` | `${url.host}/osp/a/idm/auth/oauth2/getattributes` | Legacy token/attributes URL used by clients |
| `com.netiq.client.authserver.url.logout` | `${url.host}/osp/a/idm/auth/app/logout` | Logout URL used by clients |
| `com.netiq.client.authserver.url.revoke` | `${url.host}/osp/a/idm/auth/oauth2/revoke` | Revocation URL used by clients |
| `com.netiq.client.authserver.url.extend.session` | `${url.host}/osp/a/idm/auth/app/activity` | Keep-alive URL used by clients |
| `com.netiq.ism.reverse-proxy.interface-uri` | none | Public URI when OSP sits behind a reverse proxy (written by ConfigUpdate "Access via reverse proxy") |
| `com.netiq.ism.osp.reverse-proxy.interface-uri` | none | Separate un-proxied interface ("Also allow unproxied access") |
| `com.netiq.client.reverse-proxy.issuer-uri` / `.auth-uri` | none | Front-facing issuer or authorize URI handed to clients when proxied |
| `com.netiq.idm.sslProtocol` | `TLSv1.2`/`TLSv1.3` | TLS protocol for outbound connections |

### 3.2.2 Directory (authentication source)

| Key | Default | Meaning |
|---|---|---|
| `com.netiq.idm.osp.ldap.host` | `$ID_VAULT_HOST` | LDAP server |
| `com.netiq.idm.osp.ldap.port` | `636` | LDAP port |
| `com.netiq.idm.osp.ldap.use-ssl` | `true` | LDAPS |
| `com.netiq.idm.osp.ldap.admin-dn` | `cn=admin,ou=sa,o=system` | Proxy/bind DN used to search users |
| `com.netiq.idm.osp.ldap.admin-pwd` | (`ENCRYPT`) | Bind password. ConfigUpdate always re-encrypts it on save. |
| `com.netiq.idm.osp.as.users-container-dn` | `ou=users,o=data` | Search base for users |
| `com.netiq.idm.osp.as.admins-container-dn` | `ou=sa,o=data` | Search base for administrators |
| `com.netiq.idm.osp.as.scope` | `subtree` | Search scope |
| `com.netiq.idm.osp.as.naming-attr` | `cn` | Attribute matched against the typed user name |
| `com.netiq.idm.osp.as.duplicate-resolution-naming-attr` | `mail` | Attribute used to disambiguate when the naming attribute matches more than one user |
| `com.netiq.idm.as.restrict-to-contexts` | `false` | Restrict authentication sources to the configured contexts |
| `DirectoryService/realms/jndi/params/PROVISION_ROOT` | `cn=User Application Driver,cn=driverset1,o=system` | UA driver DN (used for role and authorization lookups) |

### 3.2.3 Login method and login page

| Key | Default | Meaning |
|---|---|---|
| `com.netiq.idm.osp.login.method` | `np` | `np` (name/password), `krb` (Kerberos), `saml2` (SAML SP). See [05](05-saml-and-federation.md). |
| `com.netiq.idm.osp.login.krb.mapping-attr` | `cn` | Kerberos: eDirectory attribute equal to the Kerberos principal's user name |
| `com.netiq.idm.osp.logout.landing-page` | `false` | Kerberos: show a landing page after logout instead of re-logging in silently |
| `com.netiq.idm.osp.login.saml2.*`, `logout.saml2.*` | | See [05](05-saml-and-federation.md#51-what-is-configurable) |
| `com.netiq.idm.osp.login.captcha.type` | `none` | `none` or reCAPTCHA v2 (set via ConfigUpdate "Enable reCAPTCHA") |
| `com.netiq.idm.osp.login.captcha.attempts-before-required` | none | Failed attempts before reCAPTCHA is required (`0` = always) |
| `com.netiq.idm.osp.login.captcha.recaptcha.public-key` / `.private-key` | none | Google reCAPTCHA site and secret key |
| `com.netiq.idm.osp.login.krb.captcha.*` | | Same settings, applied to the Kerberos name/password fallback |
| `com.netiq.idm.osp.login.sign-in-help-url` | `https://<sspr>/sspr/public` | "Can't sign in?" link target |
| `com.netiq.idm.pwdmgt.provider` | `sspr` | `sspr` or `userapp` (legacy). Affects which links appear on the login page. |
| `com.netiq.idm.sspr.*` (`link-type`, `show-forgotten-password`, `show-forgotten-username`, `show-activate-account`) | | Which SSPR links to show |
| `com.netiq.idm.osp.auth.pwd.expire.show` | `true` | Warn about expiring passwords |
| `com.netiq.idm.osp.auth.pwd.expire.url` | empty | Where to send users with expiring or expired passwords |
| `com.netiq.idm.osp.naaf.*` (`admin-name` = `LOCAL\admin`, `krb-enabled`, `enabled`) | | Advanced Authentication. **Written by the installer and ConfigUpdate but not read by OSP 7.1.0**; NAAF login ships with OSP but is not wired. |
| `com.netiq.idm.osp.np-bg-enabled` | `true` | Enables the `ro-np` contract behind the OAuth **password grant**. It stays on even with SAML or Kerberos. |
| `com.netiq.idm.osp.forgotten-pwd-url` | derived | Read from the UA configuration object in LDAP when admin credentials are available |
| `com.netiq.osp.custom-css-href` / `com.netiq.idm.osp.login.custom.stylesheet.href` | none | Extra stylesheet for the login page |
| `com.netiq.osp.custom.resource.dir` / `.file` (also `internal.osp.framework.custom.resource.*`) | `osp-extras/l10n-resources`, `osp-custom-resource.jar` | Location of the branding override jar |
| `OSP_CUSTOM_NAME` (install-time only) | `Identity Access` | Written by the installer into `osp-custom-resource.jar` as `OIDPENDUSER.LoginProductName`. Not a runtime key. |

### 3.2.4 Sessions and tokens

| Key | Default | Meaning |
|---|---|---|
| `com.netiq.idm.osp.oauth.accessTokenTTL` | `120` s | Access token lifetime (all clients; there is no per-client override) |
| `com.netiq.idm.osp.oauth.authCodeTTL` | `120` s | Authorization code lifetime |
| `com.netiq.idm.osp.oauth.refreshTokenTTL` | `2592000` s (30 d) | Refresh token lifetime for confidential clients |
| `com.netiq.idm.osp.oauth.sessionTokenRevocationTTL` | `172800` s (48 h) | How long refresh tokens stay bound to the OSP session for revocation-on-logout. ConfigUpdate labels it "Refresh token lifetime (hours)". |
| `com.netiq.idm.osp.oauth.public.refreshTokenTTL` | `2700` s | Refresh token lifetime for **public** clients (no secret) |
| `com.netiq.idm.osp.oauth.refreshTokenTTLPercentage` / `.rotateRefreshTokens` | `101` (off) / `false` | Sliding refresh and rotation |
| `com.netiq.idm.osp.oauth.hmacAlgorithm` | `HMACSHA256` | HMAC for internal tokens |
| `com.netiq.idm.session-timeout` | — | **Wins over `sessionTTL`.** OSP session inactivity timeout in seconds; ConfigUpdate shows it in minutes. |
| `com.netiq.idm.osp.sessionTTL` | `2700` s | OSP session inactivity timeout, used when `session-timeout` is absent |
| `com.netiq.oauth.autologout.timeout` | `60` | Used by the IDM client apps, not OSP |
| `com.netiq.idm.osp.oauth.access-token-format.format` | `jwt` | Access token format (RFC 9068 JWT) |
| `com.netiq.idm.osp.oauth.access-token-format.{signatureAlgorithm,keyManagementAlgorithm,contentEncryptionAlgorithm,encryptPrivateData}` | sig default; key mgmt `none` | Set a key-management algorithm to make access tokens **encrypted JWE** |
| `com.netiq.idm.osp.oauth.attr.roles.maxValues` | `5` (code), `1` (installer) | Maximum `roles` values |
| `com.netiq.idm.osp.as.role-attr` | `nrfMemberOf` | LDAP attribute behind `roles` |
| `com.netiq.idm.osp.extra.attrs` | none | Comma-separated LDAP attributes added as same-name claims |
| `com.netiq.idm.osp.oauth.attr.<name>.accessToken` / `.maxValues` | `jwt` / — | Per-claim: include in the access token (`jwt`/`true`/`false`) and the value cap |

TTL floors enforced by the translator:
- `sessionTokenRevocationTTL` is at least `public.refreshTokenTTL`.
- `refreshTokenTTL` is at least `sessionTokenRevocationTTL`.

> **Written by the installer but not read by OSP 7.1.0:**
> - `com.netiq.idm.osp.token.init.cache.size` / `token.max.cache.size`
> - `com.netiq.idm.osp.oauth.{dn,first.name,last.name,initials,email,language,cacheable,expiration,auth.src.id,client,txn}`
>
> Changing them has no effect.

### 3.2.5 Signing, encryption and TLS keys

ConfigUpdate supports three layouts (`com.netiq.idm.osp.edit.oauth.keystore.cardinality`, UI-only):

| Layout | Keys used |
|---|---|
| `1:1` (legacy; **this is what the container installer creates**) | `oauth-keystore.file` = `/opt/netiq/idm/apps/osp/osp.jks`, `.type` = `pkcs12`, `.pwd`, `oauth-key-alias` = `osp`, `oauth-key.pwd`. One RSA-2048 key, valid 730 days. |
| `1:n` (recommended by the UI) | One `oauth-keystore.file` with three aliases: `oauth-signing-key-alias` (`osp-signing`), `oauth-encrypting-key-alias` (`osp-encryption`), `oauth-tls-key-alias` (`osp-tls`), each with `…-key.pwd` |
| `n:1` | Separate files: `oauth-signing-keystore.{file,type,pwd}`, `oauth-encrypting-keystore.*`, `oauth-tls-keystore.*`, plus alias/password per key |

| Key | Default | Meaning |
|---|---|---|
| `com.netiq.idm.osp.oauth-truststore.file` / `.type` / `.pwd` | `/opt/netiq/idm/apps/tomcat/conf/idm.jks`, `PKCS12` | Trust store OSP uses for outbound TLS (LDAP, metadata, NAM) |
| `com.netiq.idm.osp.ssl-keystore.file` / `.type` / `.pwd` | Tomcat keystore | Used by **clients** to trust OSP's TLS certificate |
| `com.netiq.idm.ua.ldap.keystore-pwd` | | Password of `idm.jks` (shared with UA) |

### 3.2.6 OAuth clients

Clients are **not** registered dynamically; `/register` returns 404. **Any** key ending in `.clientID` defines a client, and the text before `.clientID` is its prefix. The full list of sub-keys and a worked example are in [04 §4.5](04-endpoints-and-protocols.md#45-configuring-oauth-clients). The essentials:

| Sub-key | Meaning |
|---|---|
| `<prefix>.clientID` | `client_id` |
| `<prefix>.clientPass` (+ `._attr_obscurity`) | `client_secret`. All built-in clients default to the single `SSO_SERVICE_PWD`. Empty = public client (PKCE required). |
| `<prefix>.redirect.url` | Allowed redirect URI. OSP adds IP, short-name, canonical-name, localhost and default-port variants. |
| `<prefix>.additional.redirect.urls` | Extra redirect URIs (space-separated) |
| `<prefix>.response-types` | Allowed grants: `code`, `token`, `password`, `client_credentials`. **If unset, all grants are allowed.** |
| `<prefix>.landing.url` | Used by the IDM apps; **ignored by OSP** |
| `<prefix>.oic.*`, `.access-token-format.*`, `.logout.url`, `.keyset.*`, refresh overrides | See [04 §4.5](04-endpoints-and-protocols.md#45-configuring-oauth-clients) |

Built-in prefixes:
- `com.netiq.idmdash`, `com.netiq.idmadmin`, `com.netiq.rbpm`, `com.netiq.rbpmrest`, `com.netiq.idmengine`
- `com.netiq.forms`, `com.microfocus.workflow`
- `com.netiq.sspr`
- `com.netiq.rpt`, `com.netiq.rpt.rpt-web`, `com.netiq.idmdcs`, `com.netiq.dcsdrv`
- `com.netiq.edirapi` (Identity Console, when `ID_CONSOLE_USE_OSP=y`)
- `com.netiq.wfconsole`, `com.netiq.workflow`, `com.netiq.ualanding`

Legacy bulk keys `com.netiq.idm.osp.client-ids` and `com.netiq.idm.osp.oauth.sharedsecret` are also understood.

### 3.2.7 Auditing

| Key | Default | Meaning |
|---|---|---|
| `com.netiq.ism.audit.cef.enabled` | `false` | Send CEF events |
| `com.netiq.ism.audit.cef.host` / `.port` / `.protocol` | `localhost` / `6514` / `tcp` | Syslog/CEF destination |
| `com.netiq.ism.audit.cef.use-tls` | `true` (TCP) | TLS to the collector |
| `com.netiq.ism.audit.cef.cache-file-dir` | `/opt/netiq/idm/apps/tomcat/cache` | Spool directory |
| `com.netiq.idm.audit.cef.tls-keystore` / `-password` | none | Client certificate for TLS |
| `-Dcom.netiq.idm.osp.audit.enabled` (JVM) | `false` after install | OSP native (NAudit) auditing; separate from CEF |

### 3.2.8 Miscellaneous

| Key | Meaning |
|---|---|
| `com.netiq.idm.ua.sso-configuration` | `auto`, `nochange`, `manualpkcs8` or `manualpkcs12`. How UA's SSO signing key is managed (`auto` = from OSP). Used by ConfigUpdate. |
| `com.netiq.idm.migrate.secretstore` | Set once the master-key secret store has been migrated |
| `com.netiq.idm.osp.edit.use-ssl` | Remnant UI flag written by the installer template |

### 3.2.9 Runtime-only keys (not offered by ConfigUpdate)

These are honoured by the OSP 7.1.0 runtime but have no ConfigUpdate field. Set them directly in `ism-configuration.properties`. The rest of the 268 placeholders are in [Appendix B](B-runtime-property-placeholders.md).

| Key | Default | Meaning |
|---|---|---|
| `com.netiq.idm.osp.target-white-list.{enabled,mode,uris,uri-patterns}` | mode `auto` | Allowed redirect targets for login/logout (space-separated; `auto` ignores the lists) |
| `com.netiq.idm.osp.origin-white-list.{enabled,mode,origins}` | mode `auto` | Allowed CORS origins |
| `com.netiq.idm.osp.oauth.keyset.{sig,enc}.keystore`, `.keystore-type`, `.keystore-pwd`, `.keystore.key-alias` | none | Additional signing/encryption key sets |
| `com.netiq.idm.osp.oauth.keyset.jwks` / `.jwks_uri` | none | Keys given as JWKS |
| `com.netiq.idm.osp.oauth-truststore.checkCertificateExpiration` | forced `true` | Check certificate expiry in the truststore |
| `com.netiq.idm.osp.fileauthsrc.{path,filename}` (`enabled` computed) | `/opt/netiq/idm/apps/osp/adminusers.txt` | File-based authentication source (IG bootstrap admin) |
| `com.netiq.sspr.logout.inform-at-authentication`, `com.netiq.sspr.logout.url` | | SSPR logout chaining. OSP also derives `com.netiq.idm.osp.sspr.check.url` (`/private/CommandServlet?processAction=checkAll`) and `.status.url` (`/public/rest/status`) from `com.netiq.sspr.redirect.url`. |
| `com.netiq.idm.osp.ldap.use-db-instance-data` (+ `ism.use.global.config`, `ism.global.config.datasource.jndi.{prefix,name}`) | `false` | Keep OSP instance data in the database instead of LDAP |
| `com.netiq.idm.osp.as.bad-name.{min,max}-delay` | | Delay after an unknown user name (anti-enumeration) |
| `com.netiq.idm.osp.np-bg-enabled` | `true` | Password-grant contract (see §3.2.3) |
| `com.netiq.idm.osp.signAuthnRequests`, `requireSignedAssertions`, `saml2.*`, `login.saml2.*` (beyond the ConfigUpdate ones) | | See [05 §5.1](05-saml-and-federation.md#51-what-is-configurable) |
| `com.netiq.idm.osp.login.krb.mapping-class-attr` | `inetOrgPerson` | Kerberos: objectClass of the mapped user |
| `com.netiq.idm.osp.syslog.*` | off | Syslog audit driver |

The translator **computes** these keys; don't set them directly:
- `saml2.enabled`, `krb.enabled`, `np-enabled`
- `cef.{enabled,hostName,port,protocol,cachePath}` (computed from `com.netiq.ism.audit.cef.*`)
- `sspr.*`
- `protocol.http-interfaces`

## 3.3 JVM system properties (`setenv.sh`)

| Property | Value in container | Meaning |
|---|---|---|
| `com.netiq.ism.config` | `/opt/netiq/idm/apps/tomcat/conf/ism-configuration.properties` | Configuration file location |
| `internal.osp.framework.ext-context-file` (also `com.netiq.osp.ext-context-file`) | `/opt/netiq/idm/apps/osp/lib/osp-conf-edir.jar` | Selects the built-in configuration flavour:<br>• `osp-conf-edir.jar`: eDirectory + file.<br>• `osp-conf-ad.jar`: Active Directory + file. Defaults change: naming attribute `sAMAccountName`, roles from `memberOf`, objectClass `User`.<br>• `osp-conf-fileonly.jar`: file users only; no LDAP, Kerberos, SAML or SSPR.<br>The jar overlays `ospcfg.xml`, `idm/tenantcfg.xml` and `idm/services/authcfg.xml` on the WAR. A custom jar here is the only way to use unwired features (see [05 §5.4](05-saml-and-federation.md#54-can-osp-be-an-oauthoidc-client-of-an-external-idp)). |
| `com.netiq.idm.osp.logging.level` | `WARN` | OSP log level (`TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`) |
| `com.netiq.idm.exception.log.level` | (unset) | Level for logged exceptions |
| `com.netiq.idm.osp.logging.file.dir` | `${CATALINA_BASE}/logs` | OSP log directory |
| `com.netiq.idm.osp.client.host` | `osp` (container) | OSP's own host name, used for "local" detection |
| `com.netiq.idm.osp.audit.enabled` | `false` | Native auditing |
| `com.netiq.idm.osp.tenant.logging.naudit.enabled` | (template: `false`) | NAudit per tenant |
| `internal.osp.container.plugin.server-version` | `10.1.55.0` | Tomcat version hint for OSP's container plug-in (osp-tomcat10) |
| `com.microfocus.ism.master-keystore`, `…master-storetype`, `…password-supplier`, `…master-pass-file` | see §3.1 | Master key for `ENCRYPT` values |
| `com.sun.security.enableCRLDP` | `true` (Kubernetes) | CRL distribution point checking |
| `logback.configurationFile` | `conf/logback.xml` | Logback configuration |

## 3.4 Tools

| Tool | Path | Use |
|---|---|---|
| **ConfigUpdate** | `/opt/netiq/idm/apps/configupdate/configupdate.sh` | The supported editor for all of the above. GUI on a desktop; in containers use console mode or `-master_keystore_password=<pw> --silent <file.properties>` to apply a key/value file. Options: `-use_console`, `-edition`, `-sso_apps`, `-force-no-osp`, `-no-nam-oauth`, `-for-fips`, `-master_keystore`, `-tls_version`, `-export`, … |
| **configutil** | `/opt/netiq/idm/apps/osp/bin/configutil.sh` → `netiq-configutil.jar` | OSP's low-level configuration utility. Script commands: `add-property <SCOPE> <key> "<value>" ["ENCRYPT"]`, `import-external NODE <file>`, `set-backup-dir`, `backup GLOBAL`. The installer drives it with `osp_init.properties` and `import-configs.script`. |
| `osp_custom_cfg_util.sh` | `/idm/common/scripts/osp_custom_cfg_util.sh` | Installer helper for custom OSP settings |

## 3.5 Branding and strings

- `/opt/netiq/idm/apps/osp/osp-extras/l10n-resources/osp-custom-resource.jar` is an override jar. It has `resources/`, `css/`, `jsp/` and `images/` folders.
  - `resources/*_custom_resources_<locale>.properties` lists every end-user string (about 190 `#OIDPENDUSER.<n>=…` lines with `@Description`/`@Cause`/`@Action` comments), all commented out.
  - The installer uncomments only `OIDPENDUSER.LoginProductName` (from `OSP_CUSTOM_NAME`).
  - To change other strings, uncomment and edit them (for example `LoginBrowserTitle`, or error texts such as `OIDPENDUSER.120`), rebuild the jar, copy it to `/opt/netiq/idm/apps/tomcat/lib/`, and restart.
- The login page CSS is `webapps/osp/css/uistyles.css` (inside `osp.war`).
- The installer appends `.dialog-header-content { text-align: center; }` to it.
