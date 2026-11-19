# Appendix B. Runtime Property Placeholders (generated)

This appendix lists every `${property:default}` placeholder and `<Inject propertyName=…>` point in OSP's built-in eDirectory configuration:
- `osp-conf-edir.jar!/WEB-INF/conf/current/ospcfg.xml`
- `…/idm/tenantcfg.xml`
- `…/idm/services/authcfg.xml`

Any of these keys can be set in `ism-configuration.properties`. A key can also be set as a Java system property in `setenv.sh` or as an environment variable; the resolution order is properties, then `-D`, then environment, then the default. A key is effective only if the corresponding element is enabled; for example, SAML keys need `login.method=saml2`.

- "Inject" rows take an **XML fragment** as their value. A value that does not parse as XML is inserted as text.
- Keys such as `com.netiq.idm.osp.saml2.enabled`, `np-enabled`, `krb.enabled`, `cef.*` and `sspr.*` are normally **computed** by the property translator (`uaconfig-osp-7.1.0-4.jar`, `com.netiq.osp.uaconfig.*PropertiesGenerator`) from the user-facing keys in [03](03-configuration-reference.md). Set the user-facing key, not the computed one.

This table was generated from the 7.1.0 configuration files shipped in `osp-conf-edir.jar` and was not hand-edited.

| Key | Default in osp-conf XML | Bound to (XML element@attribute) | Inject (XML fragment) |
|---|---|---|---|
| `com.netiq.iac.osp.bootstrapadmin` | `iacadmin` | PrincipalId@text |  |
| `com.netiq.iac.osp.bootstrapadmin.authsrc` | `asf1` | Reference@refId |  |
| `com.netiq.iac.osp.bootstrapadmin.roles.enabled` | `false` | RoleAssignment@enabled |  |
| `com.netiq.idm.osp.as.admins-container-dn` | (none) | Context@context |  |
| `com.netiq.idm.osp.as.bad-name.max-delay` | (none) | Security@maxDelay |  |
| `com.netiq.idm.osp.as.bad-name.min-delay` | (none) | Security@minDelay |  |
| `com.netiq.idm.osp.as.duplicate-resolution-naming-attr` | `mail` | DuplicateResolution@attrName,NamingAttr@name,OAuth2ApplicationTemplate@secondaryNamingAttr |  |
| `com.netiq.idm.osp.as.naming-attr` | `cn` | AttributeMapEntry@nativeName,NamingAttr@name |  |
| `com.netiq.idm.osp.as.restrict-to-contexts` | `false` | LDAPAuthenticationSource@restrictToContexts |  |
| `com.netiq.idm.osp.as.role-attr` | `nrfMemberOf` | AttributeMapEntry@nativeName |  |
| `com.netiq.idm.osp.as.scope` | `subtree` | Context@scope |  |
| `com.netiq.idm.osp.as.users-container-dn` | (none) | Context@context |  |
| `com.netiq.idm.osp.attrmap.dir` | (none) | - | yes |
| `com.netiq.idm.osp.attrmap.oauth` | (none) | - | yes |
| `com.netiq.idm.osp.auth-service.clusterUpgrade` | `false` | AuthenticationService@clusterUpgrade |  |
| `com.netiq.idm.osp.auth-service.debug` | `undefined` | AuthenticationService@debug |  |
| `com.netiq.idm.osp.auth.pwd.expire.show` | `false` | AuthContract@showExpiredPasswordUi |  |
| `com.netiq.idm.osp.auth.pwd.expire.url` | (none) | AuthContract@expiredPasswordUrl |  |
| `com.netiq.idm.osp.cef.cacheFileUniqueId` | `cef.osp` | CacheFile@id |  |
| `com.netiq.idm.osp.cef.cachePath` | (none) | CacheFile@path |  |
| `com.netiq.idm.osp.cef.cefDeviceProduct` | (none) | CEFAuditor@cefDeviceProduct |  |
| `com.netiq.idm.osp.cef.cefDeviceVendor` | (none) | CEFAuditor@cefDeviceVendor |  |
| `com.netiq.idm.osp.cef.cefDeviceVersion` | (none) | CEFAuditor@cefDeviceVersion |  |
| `com.netiq.idm.osp.cef.cefFQDNHostNames` | `true` | CEFAuditor@cefFQDNHostNames |  |
| `com.netiq.idm.osp.cef.cefNumericSeverity` | `true` | CEFAuditor@cefNumericSeverity |  |
| `com.netiq.idm.osp.cef.cefNumericTimestamps` | `true` | CEFAuditor@cefNumericTimestamps |  |
| `com.netiq.idm.osp.cef.cefSyslogAppName` | `true` | CEFAuditor@cefSyslogAppName |  |
| `com.netiq.idm.osp.cef.delivery.strategy` | `persistent` | Delivery@strategy |  |
| `com.netiq.idm.osp.cef.enabled` | `false` | CEFAuditor@enabled |  |
| `com.netiq.idm.osp.cef.fileName` | (none) | CEFAuditor@fileName |  |
| `com.netiq.idm.osp.cef.filePath` | (none) | CEFAuditor@filePath |  |
| `com.netiq.idm.osp.cef.hostName` | `localhost` | CEFAuditor@hostName |  |
| `com.netiq.idm.osp.cef.level` | `ALL` | CEFAuditor@level |  |
| `com.netiq.idm.osp.cef.messageFraming` | `terminatelf` | CEFAuditor@messageFraming |  |
| `com.netiq.idm.osp.cef.persistence.deleteOnExit` | (none) | Persistence@deleteOnExit |  |
| `com.netiq.idm.osp.cef.persistence.initialCapacity` | (none) | Persistence@initialCapacity |  |
| `com.netiq.idm.osp.cef.persistence.maxStorageSize` | (none) | Persistence@maxStorageSize |  |
| `com.netiq.idm.osp.cef.port` | `514` | CEFAuditor@port |  |
| `com.netiq.idm.osp.cef.protocol` | `tcp` | CEFAuditor@protocol |  |
| `com.netiq.idm.osp.cef.queue.allowApplicationSharedDelivery` | (none) | Queue@allowApplicationSharedDelivery |  |
| `com.netiq.idm.osp.cef.queue.allowContainerSharedDelivery` | (none) | Queue@allowContainerSharedDelivery |  |
| `com.netiq.idm.osp.cef.queue.executionInterval` | (none) | Queue@executionInterval |  |
| `com.netiq.idm.osp.cef.queue.limitAction` | (none) | Queue@limitAction |  |
| `com.netiq.idm.osp.cef.queue.maxCapacity` | (none) | Queue@maxCapacity |  |
| `com.netiq.idm.osp.client-ids` | (none) | OAuth2Clients@ids |  |
| `com.netiq.idm.osp.clients` | (none) | - | yes |
| `com.netiq.idm.osp.cluster-cookie.hmac.refId` | `127` | Reference@refId |  |
| `com.netiq.idm.osp.cluster-cookie.hmacAlgorithm` | `HMACSHA256` | ClusterCookie@hmacAlgorithm |  |
| `com.netiq.idm.osp.csp.enabled` | `true` | ContentSecurityPolicy@enabled |  |
| `com.netiq.idm.osp.debug` | `undefined` | OSP@debug,Tenant@debug |  |
| `com.netiq.idm.osp.enforce-https` | `false` | SpnegoAuthentication@enforceHttps |  |
| `com.netiq.idm.osp.fileauthsrc.enabled` | `false` | FileAuthenticationSource@enabled,FileDataSource@enabled |  |
| `com.netiq.idm.osp.fileauthsrc.filename` | (none) | FileDataSource@filename |  |
| `com.netiq.idm.osp.fileauthsrc.hash-algorithm` | `SHA-512` | Field@hashAlgorithm |  |
| `com.netiq.idm.osp.fileauthsrc.path` | (none) | FileDataSource@path |  |
| `com.netiq.idm.osp.forgotten-pwd-url` | (none) | ForgotPassword@url |  |
| `com.netiq.idm.osp.jdbc-ds.enabled` | `false` | JDBCIAuthenticationSource@enabled,JDBCIDataSource@enabled,MappingAttributeStore@enabled,PrincipalProvisioning@enabled,SingleAttrInstanceStore@enabled |  |
| `com.netiq.idm.osp.jdbc.jndiDataSourceName` | (none) | JDBCIDataSource@jndiDataSourceName |  |
| `com.netiq.idm.osp.jdbc.jndiEnvName` | (none) | JDBCIDataSource@jndiEnvName |  |
| `com.netiq.idm.osp.krb.enabled` | `false` | AuthContract@enabled,KerberosAuthenticationSource@enabled,KerberosDataSource@enabled,KerberosTokenAuthentication@enabled,PrincipalMapping@enabled,SpnegoAuthentication@enabled |  |
| `com.netiq.idm.osp.krb.show-error-on-mapping-failure` | `true` | PrincipalMapping@showErrorPageOnFailure |  |
| `com.netiq.idm.osp.krb.sspr.enabled` | `false` | SSPRChecks@enabled |  |
| `com.netiq.idm.osp.ldap.admin-dn` | (none) | LDAPDataSource@adminName |  |
| `com.netiq.idm.osp.ldap.admin-pwd` | (none) | LDAPDataSource@adminPassword |  |
| `com.netiq.idm.osp.ldap.as.instance-data_ref` | (none) | - | yes |
| `com.netiq.idm.osp.ldap.dir-type` | `eDir` | LDAPDataSource@dirType |  |
| `com.netiq.idm.osp.ldap.host` | (none) | Server@host |  |
| `com.netiq.idm.osp.ldap.max-connections` | `31` | Server@maxConnections |  |
| `com.netiq.idm.osp.ldap.port` | (none) | Server@port |  |
| `com.netiq.idm.osp.ldap.pp.expression` | (none) | - | yes |
| `com.netiq.idm.osp.ldap.use-ssl` | (none) | Server@secureConnection |  |
| `com.netiq.idm.osp.logging.console.enabled` | `false` | ConsoleLogger@enabled |  |
| `com.netiq.idm.osp.logging.fieldnameformat` | `readable` | ConsoleLogger@fieldNameFormat,JavaUtilLogger@fieldNameFormat |  |
| `com.netiq.idm.osp.logging.file.app-name` | `osp-` | JavaUtilLogger@applicationName,JavaUtilLogger@loggerName |  |
| `com.netiq.idm.osp.logging.file.dir` | (none) | JavaUtilLogger@logFilePath |  |
| `com.netiq.idm.osp.logging.file.enabled` | `true` | JavaUtilLogger@enabled |  |
| `com.netiq.idm.osp.logging.file.instance-name` | `idm` | JavaUtilLogger@instanceName,JavaUtilLogger@text |  |
| `com.netiq.idm.osp.logging.file.max-file-count` | `10` | JavaUtilLogger@maxArchiveFiles |  |
| `com.netiq.idm.osp.logging.file.max-file-size-mb` | `10` | JavaUtilLogger@maxFileWrapByteSize |  |
| `com.netiq.idm.osp.logging.file.zip-on-archive` | `true` | JavaUtilLogger@zipOnArchive |  |
| `com.netiq.idm.osp.logging.format` | `readable` | ConsoleLogger@outputFormat,JavaUtilLogger@outputFormat |  |
| `com.netiq.idm.osp.logging.level` | `WARNING` | ConsoleLogger@level,JavaUtilLogger@level |  |
| `com.netiq.idm.osp.login.allow-show-hide` | `undefined` | PasswordAuthentication@allowShowHide |  |
| `com.netiq.idm.osp.login.captcha.attempts-before-required` | (none) | Captcha@attemptsBeforeRequired |  |
| `com.netiq.idm.osp.login.captcha.recaptcha.private-key` | (none) | PrivateKey@text |  |
| `com.netiq.idm.osp.login.captcha.recaptcha.public-key` | (none) | PublicKey@text |  |
| `com.netiq.idm.osp.login.captcha.recaptcha2.data-size` | `default` | ReCaptcha2@dataSize |  |
| `com.netiq.idm.osp.login.captcha.recaptcha2.data-type` | `default` | ReCaptcha2@dataType |  |
| `com.netiq.idm.osp.login.captcha.recaptcha2.theme` | `light` | ReCaptcha2@theme |  |
| `com.netiq.idm.osp.login.captcha.type` | `NONE` | Captcha@type |  |
| `com.netiq.idm.osp.login.forgotten-password-target` | `_blank` | ForgotPassword@target |  |
| `com.netiq.idm.osp.login.forgotten-password-text-res-id` | (none) | ForgotPassword@displayNameResourceId |  |
| `com.netiq.idm.osp.login.forgotten-password-title-res-id` | (none) | ForgotPassword@titleResourceId |  |
| `com.netiq.idm.osp.login.krb.mapping-attr` | `cn` | Equal@targetAttrName |  |
| `com.netiq.idm.osp.login.krb.mapping-class-attr` | `inetOrgPerson` | Equal@text |  |
| `com.netiq.idm.osp.login.method` | `np` | Reference@refId |  |
| `com.netiq.idm.osp.login.saml2.auth-tracking-id` | `authTrackingId` | AssertionAttributeMapEntry@samlName,AttributeMapEntry@nativeName |  |
| `com.netiq.idm.osp.login.saml2.auth-tracking-id.enabled` | `false` | AssertionAttributeMapEntry@enabled |  |
| `com.netiq.idm.osp.login.saml2.binding-post.slo` | `true` | Binding@slo |  |
| `com.netiq.idm.osp.login.saml2.binding-post.sso` | `true` | Binding@sso |  |
| `com.netiq.idm.osp.login.saml2.binding-redirect.slo` | `true` | Binding@slo |  |
| `com.netiq.idm.osp.login.saml2.binding-redirect.sso` | `true` | Binding@sso |  |
| `com.netiq.idm.osp.login.saml2.identifierFormat` | (none) | AccessSettings@identifierFormat |  |
| `com.netiq.idm.osp.login.saml2.mapping-attr` | `mail` | AttributeMapEntry@nativeName,Equal@targetAttrName |  |
| `com.netiq.idm.osp.login.saml2.mapping-class-attr` | `inetOrgPerson` | Equal@text |  |
| `com.netiq.idm.osp.login.saml2.metadata` | (none) | - | yes |
| `com.netiq.idm.osp.login.saml2.metadata-url` | (none) | Metadata@source |  |
| `com.netiq.idm.osp.login.saml2.responseBinding` | `HTTP-POST` | AccessSettings@responseBinding |  |
| `com.netiq.idm.osp.login.saml2.rolledKeysInMetadata` | `false` | SAML2SP@rolledKeysInMetadata |  |
| `com.netiq.idm.osp.login.saml2.saml-mapping-attr` | `mail` | AssertionAttributeMapEntry@samlName,AttributeMapEntry@nativeName |  |
| `com.netiq.idm.osp.login.saml2.saml-mapping-attr.format` | `urn:oasis:names:tc:SAML:2.0:attrname-format:basic` | AssertionAttributeMapEntry@nameFormat |  |
| `com.netiq.idm.osp.login.saml2.signatureC14nAlgorithm` | (none) | AccessSettings@signatureC14nAlgorithm |  |
| `com.netiq.idm.osp.login.saml2.signatureDigestAlgorithm` | (none) | AccessSettings@signatureDigestAlgorithm |  |
| `com.netiq.idm.osp.login.saml2.sloPost` | `true` | SAML2SP@sloPost |  |
| `com.netiq.idm.osp.login.saml2.sloRedirect` | `true` | SAML2SP@sloRedirect |  |
| `com.netiq.idm.osp.login.saml2.sloSoap` | `false` | SAML2SP@sloSoap |  |
| `com.netiq.idm.osp.login.saml2.ssoPost` | `true` | SAML2SP@ssoPost |  |
| `com.netiq.idm.osp.login.saml2.ssoRedirect` | `true` | SAML2SP@ssoRedirect |  |
| `com.netiq.idm.osp.login.show-hide-initial-state` | `undefined` | PasswordAuthentication@showHideInitialState |  |
| `com.netiq.idm.osp.login.sign-in-help-target` | `_blank;_self` | SignInHelp@target |  |
| `com.netiq.idm.osp.login.sign-in-help-text-res-id` | (none) | SignInHelp@displayNameResourceId |  |
| `com.netiq.idm.osp.login.sign-in-help-title-res-id` | (none) | SignInHelp@titleResourceId |  |
| `com.netiq.idm.osp.login.sign-in-help-url` | (none) | SignInHelp@url |  |
| `com.netiq.idm.osp.login.sspr.activate-account-target` | `_self` | DisplayLink@target |  |
| `com.netiq.idm.osp.login.sspr.activate-account-url` | (none) | DisplayLink@url |  |
| `com.netiq.idm.osp.login.sspr.forgotten-username-target` | `_self` | DisplayLink@target |  |
| `com.netiq.idm.osp.login.sspr.forgotten-username-url` | (none) | DisplayLink@url |  |
| `com.netiq.idm.osp.login.use-continue-button` | `true` | PasswordAuthentication@continueButton |  |
| `com.netiq.idm.osp.login.use-hints` | `false` | PasswordAuthentication@useHints |  |
| `com.netiq.idm.osp.logout-urls` | (none) | - | yes |
| `com.netiq.idm.osp.logout.landing-page-name` | (none) | LandingPage@page |  |
| `com.netiq.idm.osp.logout.login-res-id` | `OIDPENDUSER.50056;OIDPENDUSER.idm.activate-account;OIDPENDUSER.idm.forgot-username` | DisplayLink@displayNameResourceId |  |
| `com.netiq.idm.osp.logout.login-target` | `_self` | DisplayLink@target |  |
| `com.netiq.idm.osp.logout.login-title-res-id` | `OIDPENDUSER.50080;OIDPENDUSER.idm.activate-account-title;OIDPENDUSER.idm.forgot-username-title` | DisplayLink@titleResourceId |  |
| `com.netiq.idm.osp.np-bg-enabled` | `true` | PasswordAuthentication@enabled |  |
| `com.netiq.idm.osp.np-enabled` | `true` | AuthContract@enabled,PasswordAuthentication@enabled |  |
| `com.netiq.idm.osp.oauth-encrypting-key-alias` | (none) | KeyPair@alias |  |
| `com.netiq.idm.osp.oauth-encrypting-key.pwd` | (none) | KeyPair@password |  |
| `com.netiq.idm.osp.oauth-encrypting-keystore.file` | (none) | KeyStore@keystore |  |
| `com.netiq.idm.osp.oauth-encrypting-keystore.pwd` | (none) | KeyStore@keyStorePassword |  |
| `com.netiq.idm.osp.oauth-encrypting-keystore.type` | `jks` | KeyStore@type |  |
| `com.netiq.idm.osp.oauth-signing-key-alias` | (none) | KeyPair@alias |  |
| `com.netiq.idm.osp.oauth-signing-key.pwd` | (none) | KeyPair@password |  |
| `com.netiq.idm.osp.oauth-signing-keystore.file` | (none) | KeyStore@keystore |  |
| `com.netiq.idm.osp.oauth-signing-keystore.pwd` | (none) | KeyStore@keyStorePassword |  |
| `com.netiq.idm.osp.oauth-signing-keystore.type` | `jks` | KeyStore@type |  |
| `com.netiq.idm.osp.oauth-tls-key-alias` | (none) | KeyPair@alias |  |
| `com.netiq.idm.osp.oauth-tls-key.pwd` | (none) | KeyPair@password |  |
| `com.netiq.idm.osp.oauth-tls-keystore.enabled` | `false` | KeyStore@enabled |  |
| `com.netiq.idm.osp.oauth-tls-keystore.file` | (none) | KeyStore@keystore |  |
| `com.netiq.idm.osp.oauth-tls-keystore.pwd` | (none) | KeyStore@keyStorePassword |  |
| `com.netiq.idm.osp.oauth-tls-keystore.type` | `jks` | KeyStore@type |  |
| `com.netiq.idm.osp.oauth-truststore.checkCertificateExpiration` | (none) | KeyStore@checkCertificateExpiration |  |
| `com.netiq.idm.osp.oauth-truststore.file` | (none) | KeyStore@keystore |  |
| `com.netiq.idm.osp.oauth-truststore.pwd` | `changeit` | KeyStore@keyStorePassword |  |
| `com.netiq.idm.osp.oauth-truststore.type` | `jks` | KeyStore@type |  |
| `com.netiq.idm.osp.oauth.RFC6750Response` | `false` | OAuth2ApplicationTemplate@RFC6750Response |  |
| `com.netiq.idm.osp.oauth.access-token-format.contentEncryptionAlgorithm` | (none) | AccessTokenFormat@contentEncryptionAlgorithm |  |
| `com.netiq.idm.osp.oauth.access-token-format.encryptPrivateData` | (none) | AccessTokenFormat@encryptPrivateData |  |
| `com.netiq.idm.osp.oauth.access-token-format.format` | (none) | AccessTokenFormat@format |  |
| `com.netiq.idm.osp.oauth.access-token-format.keyManagementAlgorithm` | (none) | AccessTokenFormat@keyManagementAlgorithm |  |
| `com.netiq.idm.osp.oauth.access-token-format.signatureAlgorithm` | (none) | AccessTokenFormat@signatureAlgorithm |  |
| `com.netiq.idm.osp.oauth.accessTokenTTL` | `120` | OAuth2ApplicationTemplate@accessTokenTTL |  |
| `com.netiq.idm.osp.oauth.attr.roles.maxValues` | `5` | AttributeMapEntry@maxValues |  |
| `com.netiq.idm.osp.oauth.authCodeTTL` | `120` | OAuth2ApplicationTemplate@authCodeTTL |  |
| `com.netiq.idm.osp.oauth.hmac.refId` | `127` | Reference@refId |  |
| `com.netiq.idm.osp.oauth.hmacAlgorithm` | `HMACSHA256` | OAuth2ApplicationTemplate@hmacAlgorithm |  |
| `com.netiq.idm.osp.oauth.issuer-override` | (none) | OAuth2Protocol@issuer |  |
| `com.netiq.idm.osp.oauth.keyset.jwks` | (none) | - | yes |
| `com.netiq.idm.osp.oauth.keyset.jwks_uri` | (none) | KeySet@jwks_uri |  |
| `com.netiq.idm.osp.oauth.keyset.keystores` | (none) | - | yes |
| `com.netiq.idm.osp.oauth.open-id-connect.contentEncryptionAlgorithm` | (none) | OpenIdConnect@contentEncryptionAlgorithm |  |
| `com.netiq.idm.osp.oauth.open-id-connect.enabled` | `true` | OpenIdConnect@enabled |  |
| `com.netiq.idm.osp.oauth.open-id-connect.keyManagementAlgorithm` | (none) | OpenIdConnect@keyManagementAlgorithm |  |
| `com.netiq.idm.osp.oauth.open-id-connect.signatureAlgorithm` | (none) | OpenIdConnect@signatureAlgorithm |  |
| `com.netiq.idm.osp.oauth.open-id-connect.userInfoContentEncryptionAlgorithm` | (none) | OpenIdConnect@userInfoContentEncryptionAlgorithm |  |
| `com.netiq.idm.osp.oauth.open-id-connect.userInfoKeyManagementAlgorithm` | (none) | OpenIdConnect@userInfoKeyManagementAlgorithm |  |
| `com.netiq.idm.osp.oauth.open-id-connect.userInfoSignatureAlgorithm` | (none) | OpenIdConnect@userInfoSignatureAlgorithm |  |
| `com.netiq.idm.osp.oauth.refreshTokenTTL` | `2592000` | OAuth2ApplicationTemplate@refreshTokenTTL |  |
| `com.netiq.idm.osp.oauth.refreshTokenTTLPercentage` | `101` | OAuth2ApplicationTemplate@refreshTokenTTLPercentage |  |
| `com.netiq.idm.osp.oauth.rotateRefreshTokens` | `false` | OAuth2ApplicationTemplate@rotateRefreshTokens |  |
| `com.netiq.idm.osp.oauth.sessionTokenRevocationTTL` | `172800` | OAuth2ApplicationTemplate@sessionTokenRevocationTTL |  |
| `com.netiq.idm.osp.oauth.sharedsecret` | (none) | OAuth2Clients@secrets |  |
| `com.netiq.idm.osp.origin-white-list.enabled` | `true` | OriginWhitelist@enabled |  |
| `com.netiq.idm.osp.origin-white-list.mode` | `auto` | OriginWhitelist@mode |  |
| `com.netiq.idm.osp.origin-white-list.origins` | (none) | - | yes |
| `com.netiq.idm.osp.pp.lowercaseUsername` | `false` | PrincipalProvisioning@lowercaseUsername |  |
| `com.netiq.idm.osp.properties-file-storage.filename` | (none) | PropertiesFileStorage@filename |  |
| `com.netiq.idm.osp.properties-file-storage.path` | (none) | PropertiesFileStorage@path |  |
| `com.netiq.idm.osp.protocol.http-interfaces` | (none) | - | yes |
| `com.netiq.idm.osp.requireSignedAssertions` | `false` | Protocols@requireSignedAssertions |  |
| `com.netiq.idm.osp.revertHttpFragmentBehavior` | `true` | OSP@revertHttpFragmentBehavior |  |
| `com.netiq.idm.osp.saml2.contentEncryptionAlgorithm` | (none) | SAML2Protocol@contentEncryptionAlgorithm |  |
| `com.netiq.idm.osp.saml2.enabled` | `false` | AuthContract@enabled,IDPAuthentication@enabled,IDPAuthenticationSource@enabled,OAuth2SAML2DataSource@enabled,PrincipalMapping@enabled,SAML2Protocol@enabled |  |
| `com.netiq.idm.osp.saml2.forwardAssertionWindow` | (none) | OAuth2SAML2DataSource@forwardAssertionWindow |  |
| `com.netiq.idm.osp.saml2.keyManagementAlgorithms` | (none) | - | yes |
| `com.netiq.idm.osp.saml2.omitMetadataEncryptionMethods` | (none) | SAML2Protocol@omitMetadataEncryptionMethods |  |
| `com.netiq.idm.osp.saml2.show-error-on-mapping-failure` | `true` | PrincipalMapping@showErrorPageOnFailure |  |
| `com.netiq.idm.osp.saml2.signMetadata` | `false` | SAML2Protocol@signMetadata |  |
| `com.netiq.idm.osp.saml2.signatureC14nAlgorithm` | (none) | SAML2Protocol@signatureC14nAlgorithm |  |
| `com.netiq.idm.osp.saml2.signatureDigestAlgorithm` | (none) | SAML2Protocol@signatureDigestAlgorithm |  |
| `com.netiq.idm.osp.saml2.trustedIDP.verifyPrecheck` | (none) | AccessSettings@verifyPrecheck |  |
| `com.netiq.idm.osp.saml2.useHardenedSchema` | (none) | SAML2Protocol@useHardenedSchema |  |
| `com.netiq.idm.osp.saml2.validateMessagesSchema` | (none) | SAML2Protocol@validateMessagesSchema |  |
| `com.netiq.idm.osp.saml2.validateMetadataSchema` | (none) | SAML2Protocol@validateMetadataSchema |  |
| `com.netiq.idm.osp.saml2.verifyPrecheck` | (none) | SAML2Protocol@verifyPrecheck |  |
| `com.netiq.idm.osp.security.anticsrf.doublesubmit.enabled` | (none) | DoubleSubmit@enabled |  |
| `com.netiq.idm.osp.security.anticsrf.doublesubmit.useHostPrefix` | (none) | DoubleSubmit@useHostPrefix |  |
| `com.netiq.idm.osp.security.anticsrf.verifyorigin.enabled` | (none) | VerifyOrigin@enabled |  |
| `com.netiq.idm.osp.security.anticsrf.verifyorigin.useRefererAsFallback` | (none) | VerifyOrigin@useRefererAsFallback |  |
| `com.netiq.idm.osp.session.inactivityTimeout` | `20` | Sessions@inactivityTimeout |  |
| `com.netiq.idm.osp.signAuthnRequests` | `false` | Protocols@signAuthnRequests |  |
| `com.netiq.idm.osp.smartbanner.enabled` | `true` | SmartBanner@enabled |  |
| `com.netiq.idm.osp.sspr.check.url` | (none) | SSPRChecks@checkUrl |  |
| `com.netiq.idm.osp.sspr.check.url.param-name` | (none) | SSPRChecks@forwardUrlParamName |  |
| `com.netiq.idm.osp.sspr.enabled` | `false` | SSPRChecks@enabled |  |
| `com.netiq.idm.osp.sspr.ignore.cert.subject` | `false` | SSPRChecks@ignoreCertificateSubject |  |
| `com.netiq.idm.osp.sspr.logout.url` | (none) | SSPRChecks@logoutUrl |  |
| `com.netiq.idm.osp.sspr.logout.url.param-name` | (none) | SSPRChecks@logoutUrlParamName |  |
| `com.netiq.idm.osp.sspr.status.url` | (none) | SSPRChecks@statusUrl |  |
| `com.netiq.idm.osp.syslog.cacheFileUniqueId` | `syslog.osp` | CacheFile@id |  |
| `com.netiq.idm.osp.syslog.cachePath` | (none) | CacheFile@path |  |
| `com.netiq.idm.osp.syslog.delivery.strategy` | `persistent` | Delivery@strategy |  |
| `com.netiq.idm.osp.syslog.enabled` | `false` | SysLogAuditor@enabled |  |
| `com.netiq.idm.osp.syslog.fileName` | (none) | SysLogAuditor@fileName |  |
| `com.netiq.idm.osp.syslog.filePath` | (none) | SysLogAuditor@filePath |  |
| `com.netiq.idm.osp.syslog.hostName` | `localhost` | SysLogAuditor@hostName |  |
| `com.netiq.idm.osp.syslog.level` | `ALL` | SysLogAuditor@level |  |
| `com.netiq.idm.osp.syslog.messageFraming` | `octetcounting` | SysLogAuditor@messageFraming |  |
| `com.netiq.idm.osp.syslog.persistence.deleteOnExit` | (none) | Persistence@deleteOnExit |  |
| `com.netiq.idm.osp.syslog.persistence.initialCapacity` | (none) | Persistence@initialCapacity |  |
| `com.netiq.idm.osp.syslog.persistence.maxStorageSize` | (none) | Persistence@maxStorageSize |  |
| `com.netiq.idm.osp.syslog.port` | `514` | SysLogAuditor@port |  |
| `com.netiq.idm.osp.syslog.protocol` | `tcp` | SysLogAuditor@protocol |  |
| `com.netiq.idm.osp.syslog.queue.allowApplicationSharedDelivery` | (none) | Queue@allowApplicationSharedDelivery |  |
| `com.netiq.idm.osp.syslog.queue.allowContainerSharedDelivery` | (none) | Queue@allowContainerSharedDelivery |  |
| `com.netiq.idm.osp.syslog.queue.executionInterval` | (none) | Queue@executionInterval |  |
| `com.netiq.idm.osp.syslog.queue.limitAction` | (none) | Queue@limitAction |  |
| `com.netiq.idm.osp.syslog.queue.maxCapacity` | (none) | Queue@maxCapacity |  |
| `com.netiq.idm.osp.syslog.useLegacyStructuredDataFormat` | `false` | SysLogAuditor@useLegacyStructuredDataFormat |  |
| `com.netiq.idm.osp.target-white-list.enabled` | `true` | TargetWhitelist@enabled |  |
| `com.netiq.idm.osp.target-white-list.mode` | `auto` | TargetWhitelist@mode |  |
| `com.netiq.idm.osp.target-white-list.redirect-url-list` | (none) | - | yes |
| `com.netiq.idm.osp.tcp.timeout-ldap` | `15` | OSP@ldapBindTimeout |  |
| `com.netiq.idm.osp.tcp.timeout-ldap-bind` | `-1` | LDAPDataSource@bindTimeout |  |
| `com.netiq.idm.osp.tcp.timeout-ldap-read` | `-1;0` | LDAPDataSource@readTimeout,OSP@ldapReadTimeout |  |
| `com.netiq.idm.osp.tcp.timeout-request` | `30` | OSP@TCPTimeoutRequest |  |
| `com.netiq.idm.osp.tenant.http-interfaces` | (none) | - | yes |
| `com.netiq.idm.osp.tenant.tlsprobe.connectTimeout` | (none) | ContainerTlsProbe@connectTimeout |  |
| `com.netiq.idm.osp.tenant.tlsprobe.enabled` | `true` | ContainerTlsProbe@enabled |  |
| `com.netiq.idm.osp.tenant.tlsprobe.level` | (none) | ContainerTlsProbe@level |  |
| `com.netiq.idm.osp.tenant.tlsprobe.minProtocol` | (none) | ContainerTlsProbe@minProtocol |  |
| `com.netiq.idm.osp.tenant.tlsprobe.normalizeHost` | (none) | ContainerTlsProbe@normalizeHost |  |
| `com.netiq.idm.osp.tenant.tlsprobe.taskDelay` | (none) | ContainerTlsProbe@taskDelay |  |
| `com.netiq.idm.osp.thread-pool.continue-existing-periodic-tasks-after-shutdown` | (none) | ThreadPool@continueExistingPeriodicTasksAfterShutdown |  |
| `com.netiq.idm.osp.thread-pool.core-pool-size` | (none) | ThreadPool@corePoolSize |  |
| `com.netiq.idm.osp.thread-pool.execute-existing-delayed-tasks-after-shutdown` | (none) | ThreadPool@executeExistingDelayedTasksAfterShutdown |  |
| `com.netiq.idm.osp.thread-pool.keep-alive-time` | (none) | ThreadPool@keepAliveTime |  |
| `com.netiq.idm.osp.thread-pool.max-pool-size` | (none) | ThreadPool@maxPoolSize |  |
| `com.netiq.idm.osp.thread-pool.remove-on-cancel` | (none) | ThreadPool@removeOnCancel |  |
| `com.netiq.idm.osp.ts.attribute` | (none) | LDAPDataSource@instanceStoreAttributeName |  |
| `com.netiq.idm.osp.uuid-factory.hardwareAddress` | (none) | UUIDFactory@hardwareAddress |  |
| `com.netiq.idm.osp.uuid-factory.httpInterface` | (none) | - | yes |
| `com.netiq.idm.osp.uuid-factory.version` | `V1` | UUIDFactory@version |  |
| `com.netiq.idm.osp.uuid-factory.writeStateInterval` | (none) | UUIDFactory@writeStateInterval |  |
| `com.netiq.idm.osp.xFrameAllowedFrom` | (none) | UserInterface@xFrameAllowedFrom |  |
| `com.netiq.idm.osp.xFrameOptions` | `DENY` | UserInterface@xFrameOptions |  |
| `com.netiq.osp.login.saml2.validateMetadataCert` | `false` | TrustedIDP@validateMetadataCert |  |
