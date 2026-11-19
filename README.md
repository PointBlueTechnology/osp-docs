# OSP (One SSO Provider): Documentation

**OSP** (OpenText/NetIQ *One SSO Provider*, internally "OSP / OIDP") is the authorization server bundled with **Identity Manager 4.10.2**.

| | |
|---|---|
| Version analysed | OSP **7.1.0** (2025-06-12), container image `osp:idm-4.10.2-591` (and `0200-81` on idm254), Tomcat 10.1 |
| Source | `Identity_Manager_4.10.2_Containers.tar.gz`, plus read-only checks of the idm254 deployment on `nam-k3s` |
| Date | 2026-11-19 |

The vendor documents almost none of this. Statements marked **[C]** were confirmed in the shipped product or configuration; **[I]** marks inferences.

## What OSP is

- An **OAuth 2.0 / OpenID Connect authorization server**, issuer `https://<host>/osp/a/idm/auth/oauth2`. Every IDM web app logs in through it:
  - Dashboard (`idmdash`), admin UI (`idmadmin`), UA (`rbpm`), Forms, Workflow, Reporting, Data Collection Service and SSPR.
  - Identity Console, optionally.
- It authenticates users against **eDirectory** over LDAP (or AD with a different config jar). It issues JWT access tokens carrying IDM claims (`roles` from `nrfMemberOf`, `name` = user DN, …).
- It can delegate login to an external **SAML 2.0 IdP**, in which case it is a **SAML SP**, or use **Kerberos**. One login method is active at a time.
- It is configured **only by properties** (`tomcat/conf/ism-configuration.properties`), normally through IDM's **ConfigUpdate** tool. It has no admin UI.
- It is built on the same code base as NetIQ Access Manager's IdP. Much more ships with it than IDM turns on: SAML IdP, WS-Fed, OIDC upstream, Advanced Authentication.

## Chapters

**Configuration and federation**

| Doc | Contents |
|---|---|
| [03 Configuration reference](03-configuration-reference.md) | How configuration is loaded; every runtime property by area; value encryption; JVM flags; tools; branding |
| [04 Endpoints, OAuth/OIDC and clients](04-endpoints-and-protocols.md) | URL map, what discovery and tokens contain, **how to add an OAuth client**, the admin API |
| [05 SAML and federation](05-saml-and-federation.md) | **Making OSP a SAML SP** (manual and NAM auto-setup), SAML properties, whether OSP can be an OIDC client (no: alternatives), replacing OSP with NAM |
| [Appendix A: ConfigUpdate option catalog](appendix-a-configupdate-options.md) | All ~230 settings ConfigUpdate exposes, with labels and help text (generated) |
| [Appendix B: Runtime placeholders](appendix-b-runtime-placeholders.md) | All 268 `${key:default}` placeholders in OSP's built-in configuration (generated) |

**Operating it**

| Doc | Contents |
|---|---|
| [06 Operations](06-operations.md) | Applying changes, health checks, logging, key and certificate rotation, backup, common errors |
| [07 Security observations](07-security-observations.md) | Hardening findings and recommended settings |

**Deployment background**

| Doc | Contents |
|---|---|
| [02 Container and Kubernetes](02-container-and-kubernetes.md) | Image anatomy, first-start configuration, persistent volume layout, Helm chart |

## Quick answers

| Question | Answer |
|---|---|
| Where is the configuration? | `/opt/netiq/idm/apps/tomcat/conf/ism-configuration.properties`. In Kubernetes this is `/config/osp/tomcat/conf/…` on the shared PVC. Restart OSP after editing it. |
| How do I make OSP a SAML SP? | Set `com.netiq.idm.osp.login.method=saml2` and `…login.saml2.metadata-url` (or Base64 `…metadata`). Restart. Import `https://<host>/osp/a/idm/auth/saml2/metadata` into the IdP. Have the IdP release a `mail` attribute. Details: [05 §5.2](05-saml-and-federation.md#52-make-osp-a-saml-sp-manual-procedure). |
| Can OSP log users in through an external OAuth/OIDC IdP? | Not supported. The capability ships but is not enabled. Use SAML, broker through NAM, or let NAM replace OSP ([05 §5.4](05-saml-and-federation.md#54-can-osp-be-an-oauthoidc-client-of-an-external-idp)). |
| How do I add my own OIDC client? | Add `<prefix>.clientID`, `.clientPass`, `.redirect.url` and `.response-types=code`, then restart ([04 §4.5](04-endpoints-and-protocols.md#45-configuring-oauth-clients)). |
| Token lifetimes? | Access token 120 s; refresh token 30 days (confidential) or 45 min (public); OSP session inactivity `com.netiq.idm.session-timeout`, which wins over `sessionTTL` (idm254: 1200 s = 20 min) ([03 §3.2.4](03-configuration-reference.md#324-sessions-and-tokens)). |
| When does the signing key expire? | 730 days after install by default ([06](06-operations.md#key-and-certificate-rotation)). |

## Method and limitations

- Review of the shipped OSP configuration, ConfigUpdate and the container scripts. Runtime behaviour was confirmed only through unauthenticated `GET` requests to idm254 and a read-only cluster inventory.
- **Nothing was changed on the live system**, and no authenticated call was made.
- Untested on a live system: the SAML enablement procedure, the custom-client example, and key rollover. Validate them in a lab first.
