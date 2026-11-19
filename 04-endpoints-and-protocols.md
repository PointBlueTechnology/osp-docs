# 4. Endpoints, OAuth 2.0 / OIDC, and Client Configuration

All paths in this chapter are relative to `https://<host>/osp`. In IDM the **tenant** is `idm` and the **service** is `auth`. Every path follows the pattern `/{servlet}/[v1/]{tenant}/{service}/{protocol}/{endpoint}`.

The servlets are:

| Servlet | Prefix | Purpose | Access |
|---|---|---|---|
| Main | `/a/` | End-user and protocol endpoints | Per endpoint |
| System | `/s/` | Admin API | `IDPAdministrator` role |
| Heartbeat | `/h/` | Heartbeat data | Controlled |
| Monitor | `/m/` | Monitoring | Controlled |
| Test | `/_test_/` | Test helpers | **Closed in IDM** |

Errors come back as an XDAS fault in JSON, for example `{"Fault":{"Code":{"Value":"Sender","Subcode":{"Value":"XDAS_OUT_ENTITY_NON_EXISTANT"}},"Reason":{"Text":"Endpoint not found + 0"}}}`:

| Subcode | Meaning |
|---|---|
| `XDAS_OUT_ENTITY_NON_EXISTANT` | Unknown endpoint |
| `XDAS_OUT_POLICY_VIOLATION` | Wrong HTTP method |
| `XDAS_OUT_INVALID_INPUT` / `No handler defined for protocol` | Protocol not enabled (for example SAML) |

## 4.1 OAuth 2.0 / OpenID Connect: `/a/idm/auth/oauth2/`

| Endpoint | Methods | Auth | Purpose |
|---|---|---|---|
| `.well-known/openid-configuration` | GET | none | OIDC discovery |
| `jwks` | GET | none | Public signing key(s) as JWK with `x5c` certificate chain |
| `auth`, `grant` | GET/POST | user (browser) | Authorization endpoint. Discovery advertises `auth`; the IDM apps use `grant`. They are the same handler. |
| `token` | POST | client | Token endpoint (all grants) |
| `rocredentials`, `ccredentials`, `refresh`, `authcode*`, `implicit*`, `ext-grant` | POST | client | Legacy per-grant token endpoints |
| `getattributes` | GET/POST | bearer | User attributes for a token. The IDM apps' `com.netiq.client.authserver.url.token` points here. |
| `userinfo` | GET/POST | bearer | OIDC UserInfo |
| `introspect` | POST | client or bearer | RFC 7662 introspection. The token being inspected must differ from the caller's own token. |
| `revoke` | POST | client | RFC 7009 revocation |
| `logout` | GET | none | RP-initiated logout; redirects to `/a/idm/auth/app/logout` |
| `landingpad` | GET/POST | none | Post-login landing |
| `sign` | POST | client | Signs/encrypts data with the OSP key |
| `resetcachedclient` | DELETE | client | Drop a cached client definition |
| `getmy…`/`setmy…`/`deletemy…` `revocationtargets`, `useranswers` | GET/PUT/DELETE | user | Self-service data |
| `gettheir…`/`settheir…`/`deletetheir…` | GET/PUT/DELETE | admin | Same, for other users |
| `client-registration` / `register` | | | **Not configured** (403/404). Clients come only from properties. |

### What discovery advertises

From the live idm254 server:

| Item | Values |
|---|---|
| Issuer | `https://<host>/osp/a/idm/auth/oauth2` |
| Grants | `authorization_code`, `implicit`, `password`, `client_credentials`, `refresh_token`. Also accepted at `token` but not advertised: `urn:ietf:params:oauth:grant-type:jwt-bearer`, token-exchange, `urn:ietf:params:oauth:grant-type:saml2-bearer` (when SAML is on), and `vnd-netiq:osp:oauth2:kerberos` (when Kerberos is on). |
| Response types | `code`, `token`, `id_token`, and their combinations |
| Response modes | `query`, `fragment`, `form_post` |
| PKCE | `S256` only. **Required for public clients** (no secret); optional for confidential ones. |
| Client authentication | `client_secret_basic`, `client_secret_post`, `client_secret_jwt`, `private_key_jwt` (JTI replay-protected). Also TLS client auth. |
| DPoP | Supported (RS/ES/PS 256–512) |
| Scopes | `openid`, `profile`, `email`, `phone`, `ism` |
| `acr_values` | `idm:login:user:np` (interactive name/password), `idm:login:user:ro-np` (password grant). When enabled, also `idm:login:user:kerberos` and `idm:login:user:saml2`. |
| Claims | `sub`, `iss`, `aud`, `exp`, `iat`, `auth_time`, `acr`, `name` (user DN), `first_name`, `last_name`, `initials`, `email`, `roles`, `language`, `cacheable`, `expiration`, `auth_src_id`, `client`, `txn`, plus the standard OIDC profile/phone claims |
| Request objects | `request`, `request_uri` and `claims` parameters supported; RFC 9207 `iss` in the authorization response |

### Tokens

- **Access tokens** are JWTs in RFC 9068 format, signed with the `osp` key. They are **not encrypted** unless a key-management algorithm is configured (`com.netiq.idm.osp.oauth.access-token-format.keyManagementAlgorithm` or per client).
  - The default lifetime is 120 s (`com.netiq.idm.osp.oauth.accessTokenTTL`), which is very short. The IDM apps refresh constantly.
- **Authorization codes** live 120 s (`com.netiq.idm.osp.oauth.authCodeTTL`).
- **Refresh tokens:**
  - Confidential clients: `refreshTokenTTL` 30 days (installer), no rotation by default.
  - Public clients: 2700 s with rotation.
  - Each refresh token is tied to the OSP session for `sessionTokenRevocationTTL` (48 h), so a logout revokes it.
- **The `ism` scope:**
  - Granted automatically to every client with no consent prompt (`allowNoUserInteraction=true`).
  - It adds the IDM claims from eDirectory:

    | Claim | LDAP attribute | Notes |
    |---|---|---|
    | `first_name` | `givenName` | |
    | `last_name` | `sn` | |
    | `initials` | `initials` | |
    | `email` | `mail` | |
    | `roles` | `nrfMemberOf` | capped by `oauth.attr.roles.maxValues`; the installer sets 1 |
    | `language` | `srvprvPreferredLocale` | |
    | `name` | user DN | |

  - To add more claims, set `com.netiq.idm.osp.extra.attrs = attr1,attr2`. Each becomes a claim of the same name.
    - `com.netiq.idm.osp.oauth.attr.<name>.accessToken` controls whether it appears in the access token: `jwt` (default), `true` or `false`.
    - `com.netiq.idm.osp.oauth.attr.<name>.maxValues` caps multi-valued attributes.

## 4.2 Application endpoints: `/a/idm/auth/app/`

| Endpoint | Purpose |
|---|---|
| (root), `login` | Login page (`IDPLogin` form: `Ecom_User_ID`, `Ecom_Password`, CSRF `x-oidp-csrf`) |
| `logout` | Ends the OSP session; returns 303 to the login page or the configured landing page |
| `activity` | Session keep-alive used by the IDM apps (`com.netiq.client.authserver.url.extend.session`) |
| `sessionage` | Session age |
| `ping` | Returns `pong`. Use it as a liveness check. |
| `contractcontinue`, `authreq`, `its`, `jsp`, `setContractCookie`, `clearContractCookie`, `authCards`, `branding`, `uiIcons`, `interTenantAuth` | Login-flow internals |
| `/a/idm/auth/resource/*` | Static resources |

## 4.3 SAML: `/a/idm/auth/saml2/` (SP role, when `login.method=saml2`)

| Endpoint | Purpose |
|---|---|
| `metadata` | **SP metadata; this URL is also the SP entityID** |
| `spmetadata` | Fallback path that ConfigUpdate also tries (older OSP versions) [I] |
| `spassertion_consumer` | ACS (POST, Redirect, Artifact, PAOS) |
| `spslo` | Single logout (Redirect/POST) |
| `spsoap` | SOAP back-channel |

The IdP role (`sso`, `slo`, `soap`, `assertion_consumer`…), SAML 1.1 (`/saml/*`) and WS-Federation (`/wsfed/*`) exist in the code but are **not configured** in IDM. See [05](05-saml-and-federation.md).

## 4.4 Admin API: `/s/`

These endpoints require a principal holding `IDPAdministrator`, sent as HTTP Basic or a Bearer token.

**Who has `IDPAdministrator`:**
- Users whose `nrfMemberOf` matches the RBPM `rbpmAdmin` or `secAdmin` role. This is "admin" in the UA sense; being in `admins-container-dn` alone is not enough.

| Path | Purpose |
|---|---|
| `/s/ping`, `/s/livenessCheck`, `/s/idm/jwks` | **No auth.** Health and JWKS. |
| `/s/` (GET/PUT/POST), `/s/get`, `/s/list`, `/s/validate`, `/s/resourceMap`, `/s/reload` | System configuration |
| `/s/health`, `/s/memStatus` | Health and memory |
| `/s/loglevel` (GET/PUT), `/s/verboseLogging`, `/s/revertLogging` | Change logging at runtime |
| `/s/restart` | Restart OSP. ConfigUpdate's NAM auto-setup uses it. |
| `/s/tlsClientCertificate` | TLS client certificate |
| `/s/cluster/{nodes,messages,reset,sync}` | Cluster operations |
| `/s/idm` (GET/PUT/DELETE), `/s/idm/{get,health,loglevel,restart,start,stop}` | Tenant |
| `/s/idm/rollkeys`, `/s/idm/deleterolledkeys`, signing/encryption certificates | **Key rollover** |
| `/s/idm/auth/{get,health,restart,start,stop}` | Service |

> Changes made through `/s/` configuration PUTs live in OSP's running configuration, not in `ism-configuration.properties`. They are regenerated from the properties on the next restart. Use the properties as the source of truth.

## 4.5 Configuring OAuth clients

OSP does not support dynamic registration. Every client is a **property group** in `ism-configuration.properties`. OSP treats **any** key ending in `.clientID` as a client definition. The text before `.clientID` is the client's *prefix*.

| Sub-key under `<prefix>` | Required | Effect |
|---|---|---|
| `.clientID` | yes | The `client_id` |
| `.clientPass` | for confidential clients | `client_secret`; store with `._attr_obscurity = ENCRYPT` (or `NONE`). **Leaving it empty makes a public client:** PKCE is required, refresh tokens rotate, refresh TTL is 2700 s. A public client cannot use `password` or `client_credentials`. |
| `.redirect.url` | for browser flows | Allowed redirect URI. OSP **automatically adds** variants with the host's IP, short name, canonical name, `localhost` (if `localhost-auto-add`) and the default port. |
| `.additional.redirect.urls` | no | More redirect URIs, space-separated, taken literally |
| `.response-types` | **strongly recommended** | Allowed grants, comma-separated: `code`, `token`, `password`, `client_credentials`. **If omitted, the client may use every grant**, including password and implicit. |
| `.rotateRefreshTokens`, `.refreshTokenTTL`, `.refreshTokenTTLPercentage` | no | Refresh-token policy for this client |
| `.oic.enabled` | no | Turn OIDC (id_token) on or off for this client |
| `.oic.signatureAlgorithm` / `.oic.keyManagementAlgorithm` / `.oic.contentEncryptionAlgorithm` | no | id_token signing/encryption |
| `.oic.userInfoSignatureAlgorithm` / `…KeyManagement…` / `…ContentEncryption…` | no | Signed or encrypted UserInfo |
| `.access-token-format.format` / `.signatureAlgorithm` / `.keyManagementAlgorithm` / `.contentEncryptionAlgorithm` / `.encryptPrivateData` | no | Per-client access-token format (`jwt`) and JWE encryption |
| `.logout.url`, `.logout.return-param-name` | no | Where `oauth2/logout` returns. Automatic for clients named `rbpm` and `sspr`. |
| `.keyset.jwks` or `.keyset.{sig,enc}.keystore{,-type,-pwd,.key-alias}` | no | Client public keys (for `private_key_jwt` or encrypted tokens) |
| `.landing.url` | — | Used by the IDM apps; **ignored by OSP** |

Properties can't express:
- audience restrictions
- per-client access-token TTL
- scope restrictions
- PKCE enforcement for confidential clients

### Example: add a custom application

1. Add the client to OSP's `ism-configuration.properties`:

   ```properties
   # A confidential web app using the authorization-code flow
   com.example.myapp.clientID = myapp
   com.example.myapp.clientPass._attr_obscurity = NONE
   com.example.myapp.clientPass = <long random secret>
   com.example.myapp.redirect.url = https://myapp.example.com/oidc/callback
   com.example.myapp.response-types = code
   ```

2. Restart OSP: `kubectl -n idm rollout restart deploy/osp`.
3. Configure the application:

   | Setting | Value |
   |---|---|
   | Issuer / discovery | `https://idm254.pointbluetech.com/osp/a/idm/auth/oauth2/.well-known/openid-configuration` |
   | Client ID | `myapp` |
   | Client secret | the value from step 1 |
   | Scopes | `openid profile email` (add `ism` for IDM roles) |

   Users authenticate against eDirectory, or against the SAML IdP if [05](05-saml-and-federation.md) is enabled.

### Example: service account (client credentials)

```properties
com.example.batch.clientID = batchjob
com.example.batch.clientPass = <secret>
com.example.batch.clientPass._attr_obscurity = NONE
com.example.batch.response-types = client_credentials
```

```bash
curl -s -u batchjob:'<secret>' -d grant_type=client_credentials \
  https://idm254.pointbluetech.com/osp/a/idm/auth/oauth2/token
```

> Restrict `client_credentials` to the clients that need it, and give each client its own secret.

### Built-in IDM clients

| Prefix | client_id | Used by | Grants (as installed) |
|---|---|---|---|
| `com.netiq.idmdash` | `idmdash` | Dashboard | all (unset) |
| `com.netiq.idmadmin` | `idmadmin` | Admin UI | all |
| `com.netiq.rbpm` | `rbpm` | IDMProv (UA) | all |
| `com.netiq.rbpmrest` | `rbpmrest` | UA REST | all |
| `com.netiq.idmengine` | `idmengine` | IDM engine / drivers | all |
| `com.netiq.forms` | `forms` | Form renderer | `code,token` |
| `com.microfocus.workflow` | `workflow` | Workflow engine | `client_credentials` |
| `com.netiq.sspr` | `sspr` | SSPR | all |
| `com.netiq.rpt` | `rpt` | Reporting utility | all (installer); `password` per ConfigUpdate default |
| `com.netiq.rpt.rpt-web` | `rptw` | Reporting web (public client, no secret) | `code` |
| `com.netiq.idmdcs` | `idmdcs` | Data Collection Service | all |
| `com.netiq.dcsdrv` | `dcsdrv` | DCS driver | `password` |
| `com.netiq.edirapi` | `identityconsole` | Identity Console (only if `ID_CONSOLE_USE_OSP=y`) | `code,token` |

Each IDM app has the same client ID and secret in **its own** `ism-configuration.properties`. If you change a secret, change it in both places. ConfigUpdate does this when it is run against each component.

## 4.6 Quick tests

```bash
H=https://idm254.pointbluetech.com/osp
curl -s $H/a/idm/auth/app/ping                                   # pong
curl -s $H/a/idm/auth/oauth2/.well-known/openid-configuration | jq .issuer
curl -s $H/a/idm/auth/oauth2/jwks | jq '.keys[].kid'
# Password grant (only for a client that allows "password"):
curl -s -u <client>:<secret> -d grant_type=password -d username=<cn> -d password=<pw> -d scope=openid \
  $H/a/idm/auth/oauth2/token | jq .
```
