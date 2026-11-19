# 7. Security Observations

These are findings from reviewing the shipped OSP 7.1.0 configuration and behavior, plus read-only checks of a test deployment.

- **[C]** = confirmed in the shipped product or configuration.
- **[I]** = inferred; test before acting.

**None of these have been exploited or tested against a production system.**

| # | Finding | Evidence | Suggested action |
|---|---|---|---|
| 1 | **All OAuth grants are enabled for any client without `.response-types`**, including `password` and `implicit`. Of the built-in clients, only `workflow`, `forms`, `rptw`, `dcsdrv` and `identityconsole` are restricted. | [C] default `ism-configuration.properties` | Add `.response-types` to every client (for example `code` for the browser apps), after confirming what each app needs. |
| 2 | **One shared secret for all built-in clients** (`SSO_SERVICE_PWD`) | [C] installer | Give each client its own secret through ConfigUpdate, in both OSP and the app files. |
| 3 | **No consent screen.** Every client gets `allowNoUserInteraction=true` and the `ism` scope automatically. | [C] | Acceptable for first-party IDM apps. Be careful adding third-party clients (see [04 §4.5](04-endpoints-and-protocols.md#45-configuring-oauth-clients)). |
| 4 | **Redirect URIs are widened** to IP, short-name, canonical-name, localhost and default-port variants. | [C] | Prefer exact, external HTTPS redirect URIs. Consider `com.netiq.idm.osp.localhost-auto-add=false` and `target-white-list.*`. |
| 5 | **Weak SAML defaults:** signed assertions not required, AuthnRequests unsigned, IdP metadata certificate not validated | [C] `authcfg.xml` | When enabling SAML, set `com.netiq.idm.osp.requireSignedAssertions=true`, `signAuthnRequests=true` and `com.netiq.osp.login.saml2.validateMetadataCert=true`. |
| 6 | **The password grant survives SAML/Kerberos** (`np-bg-enabled=true`). Direct LDAP-password authentication stays possible at the token endpoint even when interactive login is federated. | [C] | Evaluate `com.netiq.idm.osp.np-bg-enabled=false` together with #1. |
| 7 | **Implicit flow and HS256/HS384/HS512 id_tokens are advertised** in discovery | [C] discovery document | Restrict through `.response-types` and per-client `.oic.signatureAlgorithm`. |
| 8 | **Unauthenticated status endpoints:** `/osp/s/ping`, `/osp/s/livenessCheck`, `/osp/s/idm/jwks` | [C] | Low risk (information only). |
| 9 | **Dormant bootstrap defaults:** client `bootstrap`/`secret`, keystore password `password`. Used only if OSP runs without an `osp-conf` jar. | [C] | Make sure `-Dinternal.osp.framework.ext-context-file` is always set. |
| 10 | **Truststore password falls back to `changeit`** if not configured | [C] | Keep `oauth-truststore.pwd` set. |
| 11 | **Token signing key is valid only 730 days by default**, and one key does everything | [C] installer | Calendar the rotation; consider the `1:n` key layout ([06](06-operations.md#key-and-certificate-rotation)). |
| 12 | **Readiness probe doesn't test OSP** (`cat catalina.out`) | [C] chart | Use an HTTPS `httpGet` on `/osp/a/idm/auth/app/ping`. |

Defaults observed in a test deployment:
- TLS 1.3 only on the OSP connector
- HSTS, CSP with nonces, `X-Frame-Options: DENY`, `nosniff`
- CSRF token on the login form
- `__HOST-` CSRF cookie
- `ErrorReportValve showReport=false`
- Configuration files mode `600`
- Access tokens expire after 120 s
