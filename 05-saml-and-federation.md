# 5. SAML 2.0 and Federation

OSP can delegate user login to an external **SAML 2.0 Identity Provider**, such as NetIQ Access Manager (NAM), Entra ID, Okta or ADFS. In this mode OSP acts as a **SAML Service Provider (SP)**. It keeps issuing OAuth2/OIDC tokens to the IDM applications, but the user authenticates at the IdP.

```
 Browser ──► IDM app (idmdash, IDMProv, IDMRPT…) ──OAuth2 code flow──► OSP  (OAuth AS + SAML SP)
                                                                          │  SAML AuthnRequest (Redirect or POST)
                                                                          ▼
                                                                   External SAML IdP (e.g. NAM)
                                                                          │  SAML Response → POST /osp/a/idm/auth/saml2/spassertion_consumer
                                                                          ▼
                                     OSP reads SAML attribute "mail" → finds the eDirectory user whose
                                     mail equals it (objectClass inetOrgPerson) → OSP session → OAuth code
                                     (acr = idm:login:user:saml2)
```

Sources:
- **[C]** (confirmed in the shipped product and configuration): `osp-conf-edir.jar!/…/idm/services/authcfg.xml`, the translator `uaconfig-osp-7.1.0-4.jar` (`Saml2PropertiesGenerator`), the OSP SAML components, and ConfigUpdate (`OspAdapter`, `NAMAdministrationConsoleValidator`).
- **[I]**: inferred.

This chapter covers:
- [5.1 What is configurable](#51-what-is-configurable)
- [5.2 Make OSP a SAML SP (manual)](#52-make-osp-a-saml-sp-manual-procedure)
- [5.3 Automatic NAM setup](#53-automatic-setup-against-netiq-access-manager)
- [5.4 OSP as an OAuth/OIDC client of another IdP?](#54-can-osp-be-an-oauthoidc-client-of-an-external-idp)
- [5.5 Alternative: NAM replaces OSP](#55-alternative-access-manager-replaces-osp-as-the-oauth-server)
- [5.6 What OSP cannot do (SAML IdP, WS-Fed, NAAF)](#56-other-federation-capabilities-in-the-code)
- [5.7 Troubleshooting](#57-troubleshooting)

## 5.1 What is configurable

OSP has exactly **one interactive login method** at a time, `com.netiq.idm.osp.login.method` [C]:

| Value | Contract enabled (acr) | Effect |
|---|---|---|
| `np` (default) | `np-contract` (`idm:login:user:np`) | Name/password against LDAP |
| `krb` | `krb-contract` (`idm:login:user:kerberos`) and the Kerberos extension grant | SPNEGO with name/password fallback |
| `saml2` | `saml2-contract` (`idm:login:user:saml2`) and the SAML2-bearer grant (`urn:ietf:params:oauth:grant-type:saml2-bearer`) | Turns on the SAML2 protocol, the SP and the trusted IdP `saml2-idp`. **Turns off the browser name/password form** (`np-enabled=false`). |

> **The OAuth `password` grant keeps working with SAML** [C]. The `ro-np` contract (`idm:login:user:ro-np`) is controlled separately by `com.netiq.idm.osp.np-bg-enabled` (default `true`). IDM back-end components that use the password grant (Reporting `rpt`, `dcsdrv`, ConfigUpdate's NAM auto-setup) keep authenticating directly against LDAP. Setting `np-bg-enabled=false` closes that path but may break those components.

### Core SAML properties

| Property | Default | Meaning |
|---|---|---|
| `com.netiq.idm.osp.login.method` | `np` | Set to `saml2` |
| `com.netiq.idm.osp.login.saml2.metadata-url` | none | IdP metadata URL, fetched by OSP at start-up over HTTPS with OSP's truststore (`idm.jks`). A fetch failure is **fatal** for the SAML configuration (`failOnError=true`). |
| `com.netiq.idm.osp.login.saml2.metadata` | none | The IdP metadata embedded as **Base64 of the XML**. **Wins over `metadata-url`** when both are set. Use Base64; raw XML is injected as XML elements and is likely to break [I]. |
| `com.netiq.idm.osp.login.saml2.mapping-attr` | `mail` | **eDirectory** attribute to match |
| `com.netiq.idm.osp.login.saml2.saml-mapping-attr` | = `mapping-attr` | **SAML attribute name** in the assertion whose value is matched |
| `com.netiq.idm.osp.login.saml2.saml-mapping-attr.format` | `urn:oasis:names:tc:SAML:2.0:attrname-format:basic` | NameFormat of that SAML attribute |
| `com.netiq.idm.osp.login.saml2.mapping-class-attr` | `inetOrgPerson` | Required objectClass of the matched user |
| `com.netiq.idm.osp.logout.saml2.landing-page` | none | `internal` (OSP's logged-out page) or `external` |
| `com.netiq.idm.osp.logout.saml2.landing-page.url` | none | Absolute URL used when `external`. Deprecated alternative: `com.netiq.idm.osp.logout.nam.url`. If neither is set and the metadata URL path starts with `/nidp`, OSP uses `https://<nam>/nidp/jsp/logoutSuccess.jsp`. |

**User matching is attribute-based, not NameID-based** [C]. OSP:
1. reads the SAML attribute named `saml-mapping-attr` (default `mail`, basic format);
2. searches the users and admins containers for an entry where `mapping-attr` (default `mail`) equals that value **and** `objectClass` equals `mapping-class-attr` (`inetOrgPerson`).

The NameID is ignored for matching. The user must be found under `as.users-container-dn` or `as.admins-container-dn`.

### Optional SAML properties (hardening and protocol)

| Property | Default | Meaning |
|---|---|---|
| `com.netiq.idm.osp.requireSignedAssertions` | **`false`** | Require signed assertions (`WantAssertionsSigned`). **Recommended `true`.** |
| `com.netiq.idm.osp.signAuthnRequests` | **`false`** | Sign AuthnRequests (`AuthnRequestsSigned`) |
| `com.netiq.osp.login.saml2.validateMetadataCert` | **`false`** | Validate the IdP metadata certificate. **Note the prefix `com.netiq.osp`.** |
| `com.netiq.idm.osp.saml2.signMetadata` | `false` | Sign OSP's SP metadata |
| `com.netiq.idm.osp.login.saml2.identifierFormat` | empty | NameIDPolicy format requested in the AuthnRequest (does not affect matching) |
| `com.netiq.idm.osp.login.saml2.responseBinding` | `HTTP-POST` | Binding requested for the Response |
| `com.netiq.idm.osp.login.saml2.binding-post.{sso,slo}`, `binding-redirect.{sso,slo}` | `true` | Bindings OSP uses toward the IdP |
| `com.netiq.idm.osp.login.saml2.{ssoPost,ssoRedirect,sloPost,sloRedirect,sloSoap}` | `true`, `true`, `true`, `true`, `false` | SP endpoints published in metadata |
| `com.netiq.idm.osp.login.saml2.rolledKeysInMetadata` | `false` | Include rolled-over keys in metadata |
| `com.netiq.idm.osp.saml2.{signatureDigestAlgorithm,signatureC14nAlgorithm,contentEncryptionAlgorithm,omitMetadataEncryptionMethods,validateMetadataSchema,validateMessagesSchema,useHardenedSchema,verifyPrecheck}` | built-in | Crypto and schema settings |
| `com.netiq.idm.osp.login.saml2.{signatureDigestAlgorithm,signatureC14nAlgorithm}` | built-in | Same, per trusted IdP |
| `com.netiq.idm.osp.saml2.keyManagementAlgorithms` | built-in | *Inject*: XML fragment |

### What OSP publishes as an SP [C]

| Item | Value |
|---|---|
| **Entity ID** | `https://<url.host>/osp/a/idm/auth/saml2/metadata` (built from `com.netiq.idm.osp.url.host`, **so `url.host` must be the external URL**) |
| Metadata | GET on the entity ID URL. It contains only an `SPSSODescriptor`. ConfigUpdate also tries `…/saml2/spmetadata` as a fallback, probably for other OSP versions [I]. |
| Assertion consumer | `https://<url.host>/osp/a/idm/auth/saml2/spassertion_consumer` (POST; Redirect, Artifact and PAOS also accepted) |
| Single logout | `…/saml2/spslo` (Redirect, POST); SOAP `…/saml2/spsoap` |
| Signing and encryption certificate | The tenant signing/encrypting key. By default this is the single `osp` key in `osp.jks`, the same key that signs OAuth tokens (expires every 730 days). |

While SAML is off, these URLs return 404 `XDAS_OUT_INVALID_INPUT` / `No handler defined for protocol`. That is the state of idm254 today.

## 5.2 Make OSP a SAML SP (manual procedure)

**Prerequisites:**
- `com.netiq.idm.osp.url.host` is the public URL (on idm254: `https://idm254.pointbluetech.com`).
- If you use `metadata-url`, the IdP's TLS chain is trusted by `idm.jks`. Public CAs such as Let's Encrypt are usually already covered through `cacerts` [I]. Otherwise, import the certificate:

  ```bash
  keytool -importcert -alias samlidp -file idp.crt -keystore /opt/netiq/idm/apps/tomcat/conf/idm.jks
  ```

- Users who will log in via SAML have `mail` populated, unique, and objectClass `inetOrgPerson`.

### Step 1: IdP metadata

- **NAM:** `https://<nam-idp>/nidp/saml2/metadata`. On your cluster: `https://nam513.pointbluetech.com/nidp/saml2/metadata`.
- **Others:** the federation metadata URL, or the downloaded XML (`base64 -w0 idp.xml`).

### Step 2: Set the properties

Run these inside the OSP container as `novlua`:

```bash
kubectl -n idm exec -it deploy/osp -- su - novlua
```

**Option A: ConfigUpdate silent mode** (handles encoding and encryption):

```bash
cd /opt/netiq/idm/apps/configupdate
cat > /tmp/saml.properties <<'EOF'
com.netiq.idm.osp.login.method = saml2
com.netiq.idm.osp.login.saml2.mapping-attr = mail
com.netiq.idm.osp.login.saml2.metadata-url = https://nam513.pointbluetech.com/nidp/saml2/metadata
com.netiq.idm.osp.login.saml2.edit.metadata-source = url
com.netiq.idm.osp.login.saml2.edit.metadata.load-on-exit = true
com.netiq.idm.osp.logout.saml2.landing-page = external
com.netiq.idm.osp.logout.saml2.landing-page.url = https://nam513.pointbluetech.com/nidp/jsp/logoutSuccess.jsp
EOF
./configupdate.sh -master_keystore_password='<MASTER_KEYSTORE_PWD>' --silent /tmp/saml.properties
rm /tmp/saml.properties
```

With `load-on-exit = true`, ConfigUpdate downloads the metadata and stores it Base64-encoded in `login.saml2.metadata`. OSP then no longer depends on the IdP being reachable at start-up.

**Option B: edit `/opt/netiq/idm/apps/tomcat/conf/ism-configuration.properties` directly.** Back it up first, then append:

```properties
com.netiq.idm.osp.login.method = saml2
com.netiq.idm.osp.login.saml2.metadata-url = https://nam513.pointbluetech.com/nidp/saml2/metadata
# or: com.netiq.idm.osp.login.saml2.metadata = <base64 of IdP metadata XML, one line>
com.netiq.idm.osp.login.saml2.mapping-attr = mail
com.netiq.idm.osp.requireSignedAssertions = true
com.netiq.osp.login.saml2.validateMetadataCert = true
com.netiq.idm.osp.logout.saml2.landing-page = external
com.netiq.idm.osp.logout.saml2.landing-page.url = https://nam513.pointbluetech.com/nidp/jsp/logoutSuccess.jsp
```

### Step 3: Restart OSP

```bash
kubectl -n idm rollout restart deploy/osp
```

The file is on the PVC, and the first-run logic does not overwrite it. Check that `curl -s https://idm254.pointbluetech.com/osp/a/idm/auth/saml2/metadata` now returns XML.

### Step 4: Register OSP at the IdP

Import `https://idm254.pointbluetech.com/osp/a/idm/auth/saml2/metadata`, or configure by hand:

| IdP setting | Value |
|---|---|
| SP entity ID | `https://idm254.pointbluetech.com/osp/a/idm/auth/saml2/metadata` |
| ACS (HTTP-POST) | `https://idm254.pointbluetech.com/osp/a/idm/auth/saml2/spassertion_consumer` |
| SLO | `https://idm254.pointbluetech.com/osp/a/idm/auth/saml2/spslo` |
| SP certificate | from the metadata (the `osp` key) |
| NameID | any; NAM's auto-setup uses emailAddress and transient |
| **Attribute release** | **An attribute named `mail` (NameFormat `basic` or `unspecified`) carrying the user's email.** This is what OSP matches on. |
| Sign assertions | yes, if you set `requireSignedAssertions=true` (recommended) |

In NAM: create a SAML 2.0 Service Provider from the metadata. Attach an Attribute Set that maps LDAP `mail` → remote name `mail`, and select it under "Attributes sent with authentication".

### Step 5: Test

1. Open `https://idm254.pointbluetech.com/idmdash/`. You are redirected to OSP and then to NAM.
2. Log in at NAM. You land back on the dashboard.
3. Decode the access token: `acr` should be `idm:login:user:saml2`.
4. Log out and check the landing page.

### Step 6 (optional): Tighten

- `com.netiq.idm.osp.signAuthnRequests=true`
- `com.netiq.idm.osp.np-bg-enabled=false` (only after checking that no IDM component needs the password grant)

## 5.3 Automatic setup against NetIQ Access Manager

ConfigUpdate's "Configure Access Manager on exit" checkbox (`com.netiq.idm.osp.login.saml2.edit.configure-nam`) runs `NAMAutoSaml2PostExecutor` when you save. It prompts for:

| Prompt | Key (ephemeral) | Default |
|---|---|---|
| Access Manager Administration Console URL | `com.netiq.auth.nam.autosaml2.nam.adminconsole.url` | scheme://host:port of the metadata URL |
| NAM admin user / password | `…nam.credentials.username` / `.password` | `cn=admin,o=novell` |
| Trust relationship display name | `…nam.trust.name` | `Auto-Created IDM-NAM Trust` |
| OSP admin user / password | `…osp.credentials.username` / `.password` | usually `uaadmin` |

It then:

1. Calls the NAM Admin Console REST API `GET amsvc/v1/idpclusters` and picks the IdP cluster whose base URL host equals the metadata URL host.
2. Fails if an SP with the same display name already exists.
3. Pings OSP (`/osp/a/idm/auth/app/ping` → `pong`) and tries to download OSP's SP metadata (`/saml2/metadata`, then `/saml2/spmetadata`).
4. If OSP is not SAML-enabled yet, it gets a token via the **password grant**, using the first client that allows `password` and the OSP admin credentials. It then calls **`POST /osp/s/restart`** with that token and retries the metadata download.
5. `POST amsvc/v1/samlattributemaps`: LDAP `mail` → remote attribute named `mapping-attr`, format `unspecified`.
6. `POST amsvc/v1/idpclusters/{id}/serviceproviders`: enabled SAML2 SP; metadata imported as text; emailAddress (from `mail`) and transient NameID; POST response binding; the attribute map sent with authentication.
7. Optionally `PUT amsvc/v1/idpclusters/{id}` `{"update":"all"}` to push the IdP configuration.

On your cluster, the NAM 5.1.3 IdP is `https://nam513.pointbluetech.com/nidp`. Neither NAM ingress exposes the NAM Administration Console, so ConfigUpdate (running in the OSP pod) needs a route to it. Otherwise, use the manual procedure.

## 5.4 Can OSP be an OAuth/OIDC client of an external IdP?

**Not in a supported way. The capability ships with OSP but nothing enables it** [C]:

- OSP's code base, shared with NAM's IdP, contains OAuth2/OIDC *client* data sources and authentication sources:
  - `OAuth2DataSource`, `OAuth2ClientDataSource`, `OpenIdDataSource` (`issuer`, `clientId`, `clientSecret`, `usernameClaim`, `useDPoP`, `autoUpdateJwks`…), plus Facebook and Microsoft Live;
  - `OAuth2AuthenticationSource`, `OpenIdAuthenticationSource`, and the `OAuth2Authentication` / `OpenIdAuthentication` login steps.
- None of the shipped `osp-conf-*.jar` configurations reference them.
- The property translator has no generator for them.
- `authcfg.xml` has no injection point for extra authentication sources.
- ConfigUpdate offers only `np`, `krb` and `saml2`.

Enabling it would mean building a **custom `osp-conf` jar** with a hand-written `authcfg.xml`: an OpenID data source, an authentication source, a contract, and a PrincipalMapping to the LDAP user. You would point `-Dinternal.osp.framework.ext-context-file` at that jar. This is unsupported, would have to be redone on every upgrade, and has not been tested here.

Supported alternatives:

1. **SAML instead of OIDC.** Most OIDC providers (Entra ID, Okta, Ping, Keycloak, NAM) also act as SAML IdPs. Use §5.2.
2. **Broker through NAM.** Configure the upstream OIDC/social provider as an identity source in NAM, and make NAM the SAML IdP for OSP. NAM does the OIDC client work; OSP sees SAML.
3. **Replace OSP with NAM** (§5.5).

## 5.5 Alternative: Access Manager replaces OSP as the OAuth server

ConfigUpdate → Authentication Server → **"Access Manager is the OAuth provider"** (`com.netiq.idm.osp.edit.is-access-manager`). The IDM apps then use NAM directly. OSP does not read this flag [C]; it only changes the apps' client configuration:

| Client property | With OSP | With NAM |
|---|---|---|
| `com.netiq.idm.osp.oauth.issuer` | `…/osp/a/idm/auth/oauth2` | `https://<nam>/nidp/oauth/nam` |
| authorize | `…/osp/a/idm/auth/oauth2/auth` | `…/nidp/oauth/nam/authz` |
| `com.netiq.client.authserver.url.logout` | `…/osp/a/idm/auth/app/logout` | `…/nidp/app/logout` |
| `com.netiq.client.authserver.url.extend.session` | `…/osp/a/idm/auth/app/activity` | `…/nidp/app` |

The "Configure Access Manager OAuth" action (`ConfigureAccessManagerOAuth`, `am-auto-oauth2-def.xml`) logs in to the NAM Console with `console-host`, `console-port`, `console-admin` and `console-password`. It uses an IdP user holding `NAM_OAUTH2_ADMIN` (`oauth2-admin`, `oauth2-password`) and creates the resource server, scopes, attribute sets and client definitions the IDM apps need. `com.netiq.ism.am.oauth.instance-id` keeps several IDM instances apart in one NAM.

Choose this when NAM is already your SSO and you want its full federation feature set (OIDC upstream, MFA, risk-based authentication) in front of IDM.

## 5.6 Other federation capabilities in the code

| Capability | Status [C] |
|---|---|
| SAML 2.0 **SP** | **Supported** (this chapter) |
| SAML 2.0 **IdP** role (OSP issuing assertions to other SPs: `/saml2/sso`, `/slo`, …) | Shipped, **not configured** in any `osp-conf` jar |
| SAML 1.1 (`/saml/*`), WS-Federation (`/wsfed/*`) | Shipped, not configured |
| OAuth2/OIDC upstream (relying party) | Shipped, not configured (§5.4) |
| NetIQ Advanced Authentication (NAAF) login (`naaf_*.jsp`, `NaafAuthenticationSource`) | Shipped, not configured. `com.netiq.idm.osp.naaf.*` is written by the installer but not read. |
| X.509 certificate login, HTTP Basic login, contract selection, SCIM 2 | Shipped, not configured |
| Kerberos/SPNEGO | **Supported** (`login.method=krb`, `login.krb.mapping-attr` (default `cn`), `login.krb.mapping-class-attr`) |

## 5.7 Troubleshooting

| Symptom | Cause / fix |
|---|---|
| `/saml2/metadata` returns 404 `No handler defined for protocol` | SAML is not enabled, or OSP was not restarted. Another cause: the metadata failed to load (`failOnError`). Check `catalina.out` / OSP log at `-Dcom.netiq.idm.osp.logging.level=DEBUG`. |
| OSP starts but SAML stays disabled after setting `metadata-url` | The OSP pod cannot reach or trust the IdP URL. Embed the metadata (`login.saml2.metadata`, Base64) or fix trust in `idm.jks`. |
| User authenticates at the IdP, then gets an OSP error | The assertion lacks a `mail` attribute, or no unique `inetOrgPerson` user under the containers has that `mail`. Check the IdP's attribute release (not the NameID). |
| IdP rejects the AuthnRequest | SP metadata not imported, or outdated after the `osp` key changed. Re-import `/saml2/metadata`. |
| Works, but assertions are accepted unsigned | Set `com.netiq.idm.osp.requireSignedAssertions=true`. |
| Logout loops straight back into login | Set `logout.saml2.landing-page=external` with the IdP's logout URL. |
| ConfigUpdate NAM auto-setup: "Unable to obtain SAML2 metadata from the Authentication Server" | OSP is not SAML-enabled and the remote restart failed. Check that a `password`-grant client exists and the OSP admin credentials are valid. |
