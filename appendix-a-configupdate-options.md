# Appendix A. ConfigUpdate Option Catalog (generated)

This is every setting that the IDM **ConfigUpdate** tool (`/opt/netiq/idm/apps/configupdate/configupdate.sh`) can edit for OSP and its OAuth clients. It was generated mechanically from the tool's declarative definition files:
- `auth-defs.xml`
- `sso-defs.xml`
- `audit-defs.xml`
- `rpt-sso-defs.xml`
- `ewf-sso-defs.xml`
- `def/executors/nam/nam-autosaml2-def.xml`
- `def/actions/am-auto-oauth2-def.xml`

All of them are in `ldapconfig_support-6.0.0-4.jar`. The UI labels and help text are joined in from its resource bundle `LdapConfigxRsrc`. See [03-configuration-reference.md](03-configuration-reference.md) for how these settings map to the runtime properties OSP actually reads.

**Reading the table:**
- **UI-only, not persisted**: `*.edit.*` keys exist only inside the ConfigUpdate session. `OspAdapter` translates them into real properties on save. For example, `edit.host`, `edit.port` and `edit.use-ssl` become `com.netiq.idm.osp.url.host`, and `login.captcha.edit.recaptcha-enabled` becomes `login.captcha.type`.
- **Shown when**: the parent switch that must hold for the field to appear, plus the `app` expression for which installed products it applies to:
  - `osp`: OSP is installed. `ua`: Identity Applications. `rpt`: Reporting. `ig`: Identity Governance.
  - `:ae` / `:se`: Advanced or Standard edition. `:sso`: the app is only an SSO client here. `fips`: FIPS mode. `nam:no_oauth`: the "hide NAM OAuth" flag.
- `scope=NODE` values (key and keystore paths) are stored per node, not cluster-wide.

#### auth-defs — Authentication › Authentication Server

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.idm.osp.edit.host` *(UI-only, not persisted)* | string |  |  | OAuth server host identifier |
| `com.netiq.idm.osp.edit.port` *(UI-only, not persisted)* | port |  |  | OAuth server TCP port |
| `com.netiq.idm.osp.edit.is-access-manager` *(UI-only, not persisted)* | checkbox |  | app: !nam:no_oauth | Access Manager is the OAuth provider — Select this if Access Manager is to be the OAuth provider (as opposed to using the bundled OSP). |
| `com.netiq.idm.osp.edit.use-ssl` *(UI-only, not persisted)* | checkbox |  | app: nam:no_oauth | OAuth server uses TLS |
| `com.netiq.idm.osp.edit.is-reverse-proxy` *(UI-only, not persisted)* | checkbox |  | app: nam:no_oauth | Access via reverse proxy — Select this if the OAuth provider is accessed through a reverse proxy. |
| `com.netiq.iac.bootstrapadmin` | user |  | com.netiq.idm.osp.edit.is-access-manager = true | Identity Governance bootstrap admin — Enter or select the LDAP DN of the user to serve as the Identity Governance bootstrap administrator. |
| `com.netiq.idm.osp.ssl-keystore.file` | file |  | com.netiq.idm.osp.edit.is-access-manager = true; app: !fips + ((ua + !ua:sso) \| (ig + !ig:sso) \| (rpt + !rpt:sso)) | Optional TLS truststore file — Enter the full path to a Java keystore file that contains the authentication server trust certificate. |
| `com.netiq.idm.osp.ssl-keystore.pwd` | password |  | com.netiq.idm.osp.edit.is-access-manager = true; app: (ua + !ua:sso) \| (ig + !ig:sso) \| (rpt + !rpt:sso) | Optional TLS truststore password — Enter the TLS truststore password. |
| `com.netiq.idm.osp.ssl-keystore.edit.type` *(UI-only, not persisted)* | combo | `PKCS12` | com.netiq.idm.osp.edit.is-access-manager = true; app: !fips + ((ua + !ua:sso) \| (ig + !ig:sso) \| (rpt + !rpt:sso)) | Optional TLS truststore type — Select the Java keystore type specifier for the TLS truststore. Values: `PKCS12` (PKCS12), `JKS` (JKS), `JCEKS` (JCEKS), `other` (Other) |
| `com.netiq.idm.osp.edit.am-configure-now` *(UI-only, not persisted)* | action |  | com.netiq.idm.osp.edit.is-access-manager = true | Configure Access Manager OAuth — Select to configure the necessary Access Manager OAuth 2.0 definitions. |
| `com.netiq.idm.osp.ssl-keystore.edit.type.other` *(UI-only, not persisted)* | string |  | com.netiq.idm.osp.ssl-keystore.edit.type = other |  |
| `com.netiq.idm.osp.edit.reverse-proxy.domain` *(UI-only, not persisted)* | string |  | com.netiq.idm.osp.edit.is-reverse-proxy = true | Reverse proxy domain — Enter the domain name of the reverse proxy interface through which the OAuth provider is reached. |
| `com.netiq.idm.osp.edit.reverse-proxy.port` *(UI-only, not persisted)* | port |  | com.netiq.idm.osp.edit.is-reverse-proxy = true | Reverse proxy port — Enter the TCP port of the reverse proxy interface through which the OAuth provider is reached. |
| `com.netiq.idm.osp.edit.reverse-proxy.tls` *(UI-only, not persisted)* | checkbox | `true` | com.netiq.idm.osp.edit.is-reverse-proxy = true | Reverse proxy uses TLS — Select this if the reverse proxy interface through which the OAuth provider is reached is using TLS. |
| `com.netiq.idm.osp.edit.reverse-proxy.separate-interface` *(UI-only, not persisted)* | checkbox |  | com.netiq.idm.osp.edit.is-reverse-proxy = true | Also allow unproxied access — Select this if tomcat is configured with a separate interface to which proxied requests are forwarded. |
| `com.netiq.idm.osp.edit.reverse-proxy-interface.host` *(UI-only, not persisted)* | string |  | com.netiq.idm.osp.edit.reverse-proxy.separate-interface = true; app: osp | Host identifier for proxied requests — Enter the name or address of the interface on which requests forwarded by the reverse proxy are received. |
| `com.netiq.idm.osp.edit.reverse-proxy-interface.port` *(UI-only, not persisted)* | port |  | com.netiq.idm.osp.edit.reverse-proxy.separate-interface = true; app: osp | Port for proxied requests — Enter the TCP port of the interface on which requests forwarded by the reverse proxy are received. |
| `com.netiq.idm.osp.edit.reverse-proxy-interface.tls` *(UI-only, not persisted)* | checkbox |  | com.netiq.idm.osp.edit.reverse-proxy.separate-interface = true; app: osp | Proxied requests use TLS — Select this if the reverse proxy forwards requests using TLS. |
| `com.netiq.idm.osp.edit.reverse-proxy.auto-urls` *(UI-only, not persisted)* | checkbox | `true` | com.netiq.idm.osp.edit.reverse-proxy.separate-interface = true; app: (ua + !ua:sso) \| (ig + !ig:sso) \| (rpt + !rpt:sso) | Obtain front-facing OAuth URLs dynamically — Select this if the web applications will obtain the URLs for browser access to the OAuth provider automatically via metadata obtained from the provider through the reverse proxy. |

> `com.netiq.idm.osp.ssl-keystore.*` is a client setting. UA and Reporting's `OAuthServlet` reads `.file` (a default install points it at `tomcat.ks`); OSP does not. The Meaning column is ConfigUpdate's help text. See [03](03-configuration-reference.md#client-side-properties-that-share-the-comnetiqidmosp-prefix).

#### auth-defs — Authentication › Authentication Configuration

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.idm.osp.as.admins-container-dn` | container | `ou=sa,o=data` |  | LDAP DN of Admins Container — LDAP DN of eDirectory container with admin User objects |
| `com.netiq.idm.osp.as.duplicate-resolution-naming-attr` | string | `mail` |  | Duplicate resolution naming attribute |
| `com.netiq.idm.as.restrict-to-contexts` | checkbox | `false` |  | Restrict authentication sources to contexts |
| `com.netiq.idm.session-timeout` | integer |  |  | Session Timeout (minutes) — User application session timeout (minutes) |
| `com.netiq.idm.osp.oauth.accessTokenTTL` | integer | `120` |  | Access token lifetime (seconds) — Validity duration for an OAuth access token in seconds. |
| `com.netiq.idm.osp.oauth.sessionTokenRevocationTTL` | integer | `172800` |  | Refresh token lifetime (hours) — Validity duration for an OAuth refresh token in hours. |
| `com.netiq.idm.osp.oauth.public.refreshTokenTTL` | integer | `2700` | app: ig>=3.6.0 \| rpt>=6.6.0 |  |

#### auth-defs — Authentication › Authentication Keys

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.idm.osp.edit.oauth.keystore.cardinality` *(UI-only, not persisted)* | combo | `1:n` |  | Keystore count — Choose whether to use a single keystore file or multiple keystore files, and whether to supply one key pair (not recommended) or multiple key pairs (recommended) Values: `1:1` (One file, one key pair), `1:n` (One file, multiple key pairs), `n:1` (Multiple files) |
| `com.netiq.idm.osp.oauth-keystore.file` | file | `<path>/osp.pkcs12` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:1; app: !fips | Keystore file — Enter or select the full path to keystore file |
| `com.netiq.idm.osp.oauth-keystore.file` | file | `<path>/osp.bcfks` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:1; app: fips | Keystore file — Enter or select the full path to keystore file |
| `com.netiq.idm.osp.oauth-keystore.edit.type` *(UI-only, not persisted)* | combo | `PKCS12` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:1; app: !fips | Keystore type — Select the Java keystore type specifier Values: `PKCS12` (PKCS12), `JKS` (JKS), `JCEKS` (JCEKS), `other` (Other) |
| `com.netiq.idm.osp.oauth-keystore.pwd` | password |  | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:1 | Keystore password — Enter the password for the keystore |
| `com.netiq.idm.osp.oauth-key-alias` | string | `osp` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:1 | Key pair alias — Enter the alias (the name in the keystore) of the key pair |
| `com.netiq.idm.osp.oauth-key.pwd` | password |  | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:1 | Key pair password — Enter the password of the key pair |
| `com.netiq.idm.osp.oauth-keystore.edit.type.other` *(UI-only, not persisted)* | string |  | com.netiq.idm.osp.oauth-keystore.edit.type = other | Type specifier — Enter the Java keystore type specifier |
| `com.netiq.idm.osp.oauth-signing-key-alias` | string | `osp-signing` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:n | Signing key pair alias — Enter the alias (the name in the keystore) of the signing key pair |
| `com.netiq.idm.osp.oauth-signing-key.pwd` | password |  | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:n | Signing key pair password — Enter the password of the signing key pair |
| `com.netiq.idm.osp.oauth-encrypting-key-alias` | string | `osp-encryption` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:n | Encryption key pair alias — Enter the alias (the name in the keystore) of the encryption key pair |
| `com.netiq.idm.osp.oauth-encrypting-key.pwd` | password |  | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:n | Encryption key pair password — Enter the password of the encryption key pair |
| `com.netiq.idm.osp.oauth-tls-key-alias` | string | `osp-tls` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:n | TLS client key pair alias — Enter the alias (the name in the keystore) of the TLS client key pair |
| `com.netiq.idm.osp.oauth-tls-key.pwd` | password |  | com.netiq.idm.osp.edit.oauth.keystore.cardinality = 1:n | TLS client key pair password — Enter the password of the TLS client key pair |
| `com.netiq.idm.osp.oauth-signing-keystore.file` | file | `<path>/osp-signing.pkcs12` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1; app: !fips | Signing keystore file — Enter or select the full path to the keystore file containing the signing key pair |
| `com.netiq.idm.osp.oauth-signing-keystore.file` | file | `<path>/osp-signing.bcfks` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1; app: fips | Signing keystore file — Enter or select the full path to the keystore file containing the signing key pair |
| `com.netiq.idm.osp.oauth-signing-keystore.edit.type` *(UI-only, not persisted)* | combo | `PKCS12` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1; app: !fips | Signing keystore type — Select the Java keystore type specifier for the signing keystore Values: `PKCS12` (PKCS12), `JKS` (JKS), `JCEKS` (JCEKS), `other` (Other) |
| `com.netiq.idm.osp.oauth-signing-keystore.pwd` | password |  | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1 | Signing keystore password — Enter the password for the signing keystore |
| `com.netiq.idm.osp.oauth-encrypting-keystore.file` | file | `<path>/osp-encrypting.pkcs12` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1; app: !fips | Encryption keystore file — Enter or select the full path to the keystore file containing the encryption key pair |
| `com.netiq.idm.osp.oauth-encrypting-keystore.file` | file | `<path>/osp-encrypting.bcfks` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1; app: fips | Encryption keystore file — Enter or select the full path to the keystore file containing the encryption key pair |
| `com.netiq.idm.osp.oauth-encrypting-keystore.edit.type` *(UI-only, not persisted)* | combo | `PKCS12` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1; app: !fips | Encryption keystore type — Select the Java keystore type specifier for the encryption keystore Values: `PKCS12` (PKCS12), `JKS` (JKS), `JCEKS` (JCEKS), `other` (Other) |
| `com.netiq.idm.osp.oauth-encrypting-keystore.pwd` | password |  | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1 | Encryption keystore password — Enter the password for the encryption keystore |
| `com.netiq.idm.osp.oauth-encrypting-key-alias` | string | `osp-encrypting` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1 | Encryption key pair alias — Enter the alias (the name in the keystore) of the encryption key pair |
| `com.netiq.idm.osp.oauth-tls-keystore.file` | file | `<path>/osp-tls.pkcs12` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1; app: !fips | TLS client keystore file — Enter or select the full path to the keystore file containing the TLS client key pair |
| `com.netiq.idm.osp.oauth-tls-keystore.file` | file | `<path>/osp-tls.bcfks` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1; app: fips | TLS client keystore file — Enter or select the full path to the keystore file containing the TLS client key pair |
| `com.netiq.idm.osp.oauth-tls-keystore.edit.type` *(UI-only, not persisted)* | combo | `PKCS12` | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1; app: !fips | TLS client keystore type — Select the Java keystore type specifier for the TLS client keystore Values: `PKCS12` (PKCS12), `JKS` (JKS), `JCEKS` (JCEKS), `other` (Other) |
| `com.netiq.idm.osp.oauth-tls-keystore.pwd` | password |  | com.netiq.idm.osp.edit.oauth.keystore.cardinality = n:1 | TLS client keystore password — Enter the password for the TLS client keystore |
| `com.netiq.idm.osp.oauth-signing-keystore.edit.type.other` *(UI-only, not persisted)* | string |  | com.netiq.idm.osp.oauth-signing-keystore.edit.type = other |  |
| `com.netiq.idm.osp.oauth-encrypting-keystore.edit.type.other` *(UI-only, not persisted)* | string |  | com.netiq.idm.osp.oauth-encrypting-keystore.edit.type = other |  |
| `com.netiq.idm.osp.oauth-tls-keystore.edit.type.other` *(UI-only, not persisted)* | string |  | com.netiq.idm.osp.oauth-tls-keystore.edit.type = other |  |

#### auth-defs — Authentication › Authentication Method

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.idm.osp.login.method` | combo | `np` |  | Method — The default method to be used for user authentication. Values: `np` (Name/Password), `krb` (Kerberos), `saml2` (SAML 2.0) |
| `com.netiq.idm.osp.login.captcha.edit.recaptcha-enabled` *(UI-only, not persisted)* | checkbox | `false` | com.netiq.idm.osp.login.method = np | Enable reCAPTCHA — Select this item to enable reCAPTCHA on the sign-in page. |
| `com.netiq.idm.osp.login.captcha.attempts-before-required` | integer |  | com.netiq.idm.osp.login.captcha.edit.recaptcha-enabled = true | Number of attempts before required — Enter the number of unsuccessful sign-in attempts before reCAPTCHA is required. Zero means always required. |
| `com.netiq.idm.osp.login.captcha.recaptcha.public-key` | string |  | com.netiq.idm.osp.login.captcha.edit.recaptcha-enabled = true | Site key — Enter the reCAPTCHA site key value obtained from the Google reCAPTCHA site. |
| `com.netiq.idm.osp.login.captcha.recaptcha.private-key` | string |  | com.netiq.idm.osp.login.captcha.edit.recaptcha-enabled = true | Private key — Enter the reCAPTCHA private key value obtained from the Google reCAPTCHA site. |
| `com.netiq.idm.osp.login.krb.mapping-attr` | string | `cn` | com.netiq.idm.osp.login.method = krb | Mapping attribute name — The name of the eDirectory attribute whose value matches the Kerberos username. |
| `com.netiq.idm.osp.login.krb.captcha.edit.recaptcha-enabled` *(UI-only, not persisted)* | checkbox | `false` | com.netiq.idm.osp.login.method = krb | Enable fallback reCAPTCHA — Select this item to enable reCAPTCHA with the fallback name/password when Kerberos cannot be used. |
| `com.netiq.idm.osp.naaf.krb-enabled` | checkbox |  | com.netiq.idm.osp.login.method = krb |  |
| `com.netiq.idm.osp.logout.landing-page` | checkbox | `false` | com.netiq.idm.osp.login.method = krb | Use logout landing page — If Kerberos is used for authentication and logout is selected then use a landing page rather than immediately logging in again. |
| `com.netiq.idm.osp.login.krb.captcha.attempts-before-required` | integer |  | com.netiq.idm.osp.login.krb.captcha.edit.recaptcha-enabled = true |  |
| `com.netiq.idm.osp.login.krb.captcha.recaptcha.public-key` | string |  | com.netiq.idm.osp.login.krb.captcha.edit.recaptcha-enabled = true |  |
| `com.netiq.idm.osp.login.krb.captcha.recaptcha.private-key` | string |  | com.netiq.idm.osp.login.krb.captcha.edit.recaptcha-enabled = true |  |
| `com.netiq.idm.osp.login.saml2.mapping-attr` | string | `mail` | com.netiq.idm.osp.login.method = saml2 | Mapping attribute name — The name of the eDirectory attribute whose value matches the value obtained in the SAML assertion. |
| `com.netiq.idm.osp.logout.saml2.landing-page` | combo | `none` | com.netiq.idm.osp.login.method = saml2 | Landing page — Choose whether or not to use a landing page after logout and, if so, whether to use an internal or external page. Values: `none` (None), `internal` (Internal), `external` (External) |
| `com.netiq.idm.osp.login.saml2.edit.metadata-source` *(UI-only, not persisted)* | combo |  | com.netiq.idm.osp.login.method = saml2 | Metadata source — Select the source of the IDP metadata. Either load metadata from a URL or paste previously-obtained metadata. Values: `url` (URL), `cp` (Copy/paste) |
| `com.netiq.idm.osp.login.saml2.edit.configure-nam` *(UI-only, not persisted)* | checkbox |  | com.netiq.idm.osp.login.method = saml2 | Configure Access Manager on exit — Select this option to automatically configure a SAML Service Provider definition in Access Manager. |
| `com.netiq.idm.osp.logout.saml2.landing-page.url` | uri |  | com.netiq.idm.osp.logout.saml2.landing-page = external | URL — Enter the URL of the external landing page to which the browser will be directed. |
| `com.netiq.idm.osp.login.saml2.metadata-url` | uri |  | com.netiq.idm.osp.login.saml2.edit.metadata-source = url | Metadata URL — The URL from which the SAML 2.0 IDP metadata is obtained. |
| `com.netiq.idm.osp.login.saml2.edit.metadata.load-on-exit` *(UI-only, not persisted)* | checkbox |  | com.netiq.idm.osp.login.saml2.edit.metadata-source = url | Load on save — Select to load the metadata from the URL and save it to the configuration when exiting this application. |
| `com.netiq.idm.osp.login.saml2.metadata` | text_area |  | com.netiq.idm.osp.login.saml2.edit.metadata-source = cp | IDP metadata — The metadata obtained from the SAML 2.0 IDP. |

#### auth-defs — Authentication › Identity Governance Bootstrap Administrator

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.iac.edit.bootstrapadmin.authsrc` *(UI-only, not persisted)* | checkbox | `false` |  | LDAP user — Specify or select an LDAP user to act as the bootstrap administrator. |
| `com.netiq.iac.bootstrapadmin` | string |  | com.netiq.iac.edit.bootstrapadmin.authsrc = true; app: !osp + ig | Identity Governance bootstrap admin — Enter or select the LDAP DN of the user to serve as the Identity Governance bootstrap administrator. |
| `com.netiq.idm.osp.fileauthsrc.path` | file | `/opt/netiq/idm/apps/osp/adminusers.txt` | com.netiq.iac.edit.bootstrapadmin.authsrc = false | File name and path — Enter or select the name and path of the CSV file containing the bootstrap admin. |
| `com.netiq.iac.bootstrapadmin` | string | `igadmin` | com.netiq.iac.edit.bootstrapadmin.authsrc = false | Identity Governance bootstrap admin — Enter or select the LDAP DN of the user to serve as the Identity Governance bootstrap administrator. |
| `com.netiq.iac.edit.bootstrapadmin.password` *(UI-only, not persisted)* | password |  | com.netiq.iac.edit.bootstrapadmin.authsrc = false | Password — Specify the password of the bootstrap administrator. |

#### auth-defs — Authentication › Password Management

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.idm.pwdmgt.provider` | combo | `sspr` |  | Password Management Provider — The password management provider. Values: `sspr` (Self Service Password Reset (SSPR)), `userapp` (User Application (Legacy)) |
| `com.netiq.idm.sspr.edit.link-type` *(UI-only, not persisted)* | combo |  | com.netiq.idm.pwdmgt.provider = sspr | User interface — Choose the link or links to appear on the sign in page. Values: `cant_signin` ("Can't sign in?"), `other` (Other links), `none` (None) |
| `com.netiq.idm.sspr.edit.show-forgotten-password` *(UI-only, not persisted)* | checkbox |  | com.netiq.idm.sspr.edit.link-type = other | "Forgot password?" — Select this to cause the "Forgot password?" link to appear. |
| `com.netiq.idm.sspr.edit.show-forgotten-username` *(UI-only, not persisted)* | checkbox |  | com.netiq.idm.sspr.edit.link-type = other | "Forgot username?" — Select this to cause the "Forgot username?" link to appear. |
| `com.netiq.idm.sspr.edit.show-activate-account` *(UI-only, not persisted)* | checkbox |  | com.netiq.idm.sspr.edit.link-type = other | "Activate account" — Select this to cause the "Activate account" link to appear. |
| `com.netiq.idm.pwd-mgt` | combo | `internal` | com.netiq.idm.pwdmgt.provider = userapp | Forgotten Password — Configure forgotten password functionality. Values: `none` (None), `internal` (Internal), `external` (External) |
| `com.netiq.idm.forgot-pwd-url` | uri |  | com.netiq.idm.pwd-mgt = external | Forgot Password Link — Forgot Password Link, e.g. http://pwdmgthost:port/pwdmgtwar/jsps/pwdmgt/ForgotPassword.jsp or ./jsps/pwdmgt/ForgotPassword.jsp |
| `com.netiq.idm.forgot-pwd-return-url` | uri |  | com.netiq.idm.pwd-mgt = external | Forgot Password Return Link |
| `com.netiq.idm.forgot-pwd-service-url` | uri |  | com.netiq.idm.pwd-mgt = external | Forgot Password Web Service URL — Forgot Password Web Service URL, e.g. https://idmhost:sslport/idm/pwdmgt/service |

#### sso-defs — IDM SSO Clients › Dashboard

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.idmdash.clientID` | string | `idmdash` |  |  |
| `com.netiq.idmdash.clientPass` | password |  |  |  |
| `com.netiq.idmdash.landing.url` | uri | `/idmdash/#/landing` |  |  |
| `com.netiq.idmdash.redirect.url` | uri | `/idmdash/oauth.html` | app: osp |  |

#### sso-defs — IDM SSO Clients › Administrator

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.idmadmin.clientID` | string | `idmadmin` |  |  |
| `com.netiq.idmadmin.clientPass` | password |  |  |  |
| `com.netiq.idmadmin.landing.url` | uri | `/idmdash/#/landing` |  |  |
| `com.netiq.idmadmin.redirect.url` | uri | `/idmadmin/oauth.html` | app: osp |  |

#### sso-defs — IDM SSO Clients › Forms

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.forms.clientID` | string | `forms` |  |  |
| `com.netiq.forms.clientPass` | password |  |  |  |
| `com.netiq.forms.redirect.url` | uri | `/forms/oauth.html` |  |  |

#### sso-defs — IDM SSO Clients › Workflow Engine

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.microfocus.workflow.clientID` | string | `workflow` |  |  |
| `com.microfocus.workflow.clientPass` | password |  |  |  |

#### sso-defs — IDM SSO Clients › Engine

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.idmengine.clientID` | string | `idmengine` |  |  |
| `com.netiq.idmengine.clientPass` | password |  |  |  |

#### sso-defs — IDM SSO Clients › RBPM

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.rbpm.clientID` | string | `rbpm` |  |  |
| `com.netiq.rbpm.clientPass` | password |  |  |  |
| `com.netiq.rbpm.landing.url` | uri | `/idmdash/#/landing` |  | URL link to landing page |
| `com.netiq.rbpm.redirect.url` | uri |  | app: osp |  |
| `com.netiq.idm.ua.sso-configuration` | combo | `nochange` | app: !ua:sso | Single Sign On (SSO) — Select to configure SSO settings. Values: `nochange` (No Change), `auto` (Auto), `manualpkcs8` (Manual PKCS8), `manualpkcs12` (Manual PKCS12) |
| `com.novell.sso.signingcert.oauth` | text_area |  | com.netiq.idm.ua.sso-configuration = manualpkcs8 | SSO Digital Signature Certificate — SSO digital signature, base64 X509 Certificate (with headers) |
| `com.novell.sso.signingkeyfile.oauth` | file |  | com.netiq.idm.ua.sso-configuration = manualpkcs8 | SSO Digital Signature Private Key — SSO digital signature private key |
| `com.novell.sso.signingkeyPassword.oauth` | password |  | com.netiq.idm.ua.sso-configuration = manualpkcs8 | Signing Key Password — SSO Signing key password. |
| `com.novell.sso.signingkey.alias` | string |  | com.netiq.idm.ua.sso-configuration = manualpkcs12 | Key Alias — The alias of the key store entry containing the certificate and key |

#### sso-defs — IDM SSO Clients › Reporting Client

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.rpt.rpt-web.clientID` | string | `rptw` |  |  |
| `com.netiq.rpt.rpt-web.redirect.url` | uri | `/IDMRPT/oauth.html` | app: osp |  |

#### sso-defs — IDM SSO Clients › Reporting Utility Client

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.rpt.clientID` | string | `rpt` |  |  |
| `com.netiq.rpt.clientPass` | password |  |  |  |
| `com.netiq.rpt.response-types` | string | `password` | app: osp |  |

#### sso-defs — IDM SSO Clients › Reporting Client

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.rpt.redirect.url` | uri | `/IDMRPT/oauth.html` | app: osp |  |

#### sso-defs — IDM SSO Clients › Data Collection Service

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.idmdcs.clientID` | string | `idmdcs` |  |  |
| `com.netiq.idmdcs.clientPass` | password |  |  |  |
| `com.netiq.idmdcs.landing.url` | uri | `/idmdash/#/landing` |  |  |
| `com.netiq.idmdcs.redirect.url` | uri | `/idmdcs/oauth.html` | app: osp |  |

#### sso-defs — IDM SSO Clients › DCS Driver

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.dcsdrv.clientID` | string | `dcsdrv` |  |  |
| `com.netiq.dcsdrv.clientPass` | password |  |  |  |

#### sso-defs — IDM SSO Clients › REST API

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.rbpmrest.clientID` | string | `rbpmrest` |  |  |
| `com.netiq.rbpmrest.clientPass` | password |  |  |  |

#### sso-defs — IDM SSO Clients › SSPR

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.sspr.clientID` | string | `sspr` |  |  |
| `com.netiq.sspr.clientPass` | password |  |  |  |
| `com.netiq.sspr.redirect.url` | uri |  | app: osp |  |

#### audit-defs — CEF Auditing › CEF Auditing

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.ism.audit.cef.enabled` | checkbox |  |  | Send audit events — Select to enable sending CEF audit events to an audit server. |
| `com.netiq.ism.audit.cef.host` | string |  | com.netiq.ism.audit.cef.enabled = true | Destination host — Enter the DNS name or IP address of the audit server. |
| `com.netiq.ism.audit.cef.port` | port | `6514` | com.netiq.ism.audit.cef.enabled = true | Destination port — Enter the TCP or UDP port number at which the audit server receives CEF events. |
| `com.netiq.ism.audit.cef.protocol` | combo | `TCP` | com.netiq.ism.audit.cef.enabled = true | Network protocol — Select the network protocol used by the audit server to receive CEF events. Values: `tcp` (TCP), `udp` (UDP) |
| `com.netiq.ism.audit.cef.cache-file-dir` | file |  | com.netiq.ism.audit.cef.enabled = true | Event intermediate cache directory — Enter or select a file system directory where CEF audit events will be briefly stored before the events are sent to the audit server. |
| `com.netiq.ism.audit.cef.use-tls` | checkbox | `true` | com.netiq.ism.audit.cef.protocol = tcp | Use TLS — Select if the audit server is configured to use TLS with TCP. |
| `com.netiq.idm.audit.cef.tls-keystore` | file |  | com.netiq.ism.audit.cef.use-tls = true; app: !fips + (ua:se \| ua:ae) | Identity Manager keystore file — Enter or select a Java keystore file that contains the keypair for TLS client authentication. |
| `com.netiq.idm.audit.cef.tls-keystore-password` | password |  | com.netiq.ism.audit.cef.use-tls = true; app: ua:se \| ua:ae | Identity Manager keystore password — Enter the password for the Java keystore file. |

#### nam-autosaml2-def — Access Manager Auto-Configure SAML2 › Access Manager Administration Console

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.auth.nam.autosaml2.nam.adminconsole.url` | string |  |  | Access Manager Administration Console Url |

#### nam-autosaml2-def — Access Manager Auto-Configure SAML2 › Access Manager Administrator Credentials

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.auth.nam.autosaml2.nam.credentials.username` | string |  |  | User name |
| `com.netiq.auth.nam.autosaml2.nam.credentials.password` | password |  |  | Password |

#### nam-autosaml2-def — Access Manager Auto-Configure SAML2 › Unique Display Name for Access Manager/Authentication Server Trust Relationship

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.auth.nam.autosaml2.nam.trust.name` | string |  |  | Display name |

#### nam-autosaml2-def — Access Manager Auto-Configure SAML2 › Authentication Server Administrator Credentials

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.auth.nam.autosaml2.osp.credentials.username` | string |  |  | User name (Usually: uaadmin or iacadmin) |
| `com.netiq.auth.nam.autosaml2.osp.credentials.password` | password |  |  | Password |

#### am-auto-oauth2-def — Access Manager Auto-Configure OAuth 2.0 › Access Manager Administration Console

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.auth.am.auto-oauth2.console-host` | string |  |  | Access Manager administration console host — Enter the host identifer (name or address) of the Access Manager console. |
| `com.netiq.auth.am.auto-oauth2.console-port` | port |  |  | Access Manager administration console port — Enter the TCP port number of the Access Manager console. |
| `com.netiq.auth.am.auto-oauth2.console-admin` | string |  |  | Admin DN — Enter the LDAP DN of an Access Manager console administrator. |
| `com.netiq.auth.am.auto-oauth2.console-password` | password |  |  | Admin password — Enter the password of an Access Manager console administrator. |
| `com.netiq.auth.am.auto-oauth2.console-update-idp` | checkbox | `true` |  | Admin password — Select to update (reload) the Access Manager IDP configuration after completion. |

#### am-auto-oauth2-def — Access Manager Auto-Configure OAuth 2.0 › Access Manager IDP OAuth 2.0 Administrator Credentials

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.auth.am.auto-oauth2.oauth2-admin` | user |  |  | Administrator DN — Enter the LDAP DN of an Access Manager IDP user that is granted the NAM_OAUTH2_ADMIN role. |
| `com.netiq.auth.am.auto-oauth2.oauth2-password` | password |  |  | Administrator password — Enter the password of the administrator specified above. |

#### am-auto-oauth2-def — Access Manager Auto-Configure OAuth 2.0 › Advanced Options

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.ism.am.oauth.instance-id` | string |  |  | ISM Application instance ID — Enter a short identifier to uniquely qualify the OAuth resources created in Access Manager.  This is only useful if more than one instance of an ISM application (e.g., Identity Governance) is going to use the same instance of Access Manager, and if the two ISM product instances must not share attribute sets, scopes, or client definitions. |

#### ewf-sso-defs — Web client

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.wfconsole.clientID` | string | `wfconsole` |  |  |
| `com.netiq.wfconsole.redirect.url` | uri | `/wfconsole/oauth.html` |  |  |
| `com.netiq.wfconsole.response-types` | string | `code` |  |  |

#### ewf-sso-defs — Workflow Consumer

| Key | Type | Default | Shown when | Meaning |
|---|---|---|---|---|
| `com.netiq.workflow.clientID` | string | `workflow` |  |  |
| `com.netiq.workflow.clientPass` | password |  |  |  |
| `com.netiq.workflow.response-types` | string | `client_credentials` | app: osp |  |
| `com.netiq.workflow.authorization.type` | combo | `internal` | app: !ewf:sso | Authorization type Values: `internal` (Internal), `external` (External) |
| `com.netiq.wfe.consumer.url` | uri | `/api/wfi` | app: !ewf:sso | Consumer URL — The URL of the workflow consumer. |
