---
id: oauth-password-grant-removed
tier: 2
tier_label: Won't Run
title: OAuth 2.0 Password Grant Completely Removed
series: spring-boot 3.5 → 4.0
effort: L
openrewrite: false
subsystem: security
max_pages: 3
---

Spring Security 7 removed the OAuth 2.0 Resource Owner Password Credentials grant. Apps that authenticate with it fail at startup or return 401s at runtime.

## What You'll See {.error-output}

```error-output
org.springframework.beans.factory.BeanCreationException:
  Error creating bean with name 'securityFilterChain'
...
Caused by: java.lang.IllegalArgumentException:
  The grant type 'password' is not supported.
  Supported grant types are: [authorization_code, client_credentials,
  refresh_token, urn:ietf:params:oauth:grant-type:device_code,
  urn:ietf:params:oauth:grant-type:token-exchange]
 at org.springframework.security.oauth2.client.registration.ClientRegistration$Builder.build(...)
---
APPLICATION FAILED TO START
```

## What Changed {.what-changed}

The password grant, deprecated in Spring Security 5.x, is gone in Spring Security 7 (shipped with Spring Boot 4.0). The <code>ClientRegistration</code> builder rejects <code>authorization-grant-type: password</code>, and the <code>ResourceOwnerPasswordResourceDetails</code> class no longer exists.

## Why {.why-changed}

The password grant hands user credentials directly to the client application, defeating the point of OAuth delegation. The OAuth 2.1 specification (RFC 9700) formally removed it; Spring Security followed.

## The Fix {.diffs}

```diff-card
# // application.yml — client registration
@@removed
spring:
  security:
    oauth2:
      client:
        registration:
          my-api:
            authorization-grant-type: password
            client-id: my-client
            client-secret: secret
        provider:
          my-api:
            token-uri: https://auth.example.com/oauth/token
@@added
spring:
  security:
    oauth2:
      client:
        registration:
          my-api:
            authorization-grant-type: authorization_code
            client-id: my-client
            client-secret: secret
            scope: openid,profile
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
        provider:
          my-api:
            issuer-uri: https://auth.example.com
```

```diff-card
# // Java configuration — programmatic registration
@@removed
ClientRegistration.withRegistrationId("legacy")
    .authorizationGrantType(AuthorizationGrantType.PASSWORD)
    .tokenUri("https://auth.example.com/oauth/token")
    .clientId("my-client")
    .clientSecret("secret")
    .build();
@@added
ClientRegistration.withRegistrationId("legacy")
    .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
    .authorizationUri("https://auth.example.com/authorize")
    .tokenUri("https://auth.example.com/oauth/token")
    .redirectUri("{baseUrl}/login/oauth2/code/{registrationId}")
    .clientId("my-client")
    .clientSecret("secret")
    .build();
```

## How To Fix {.fixes}

#### Migrate to Authorization Code + PKCE.

Replace the password grant with Authorization Code flow. For browser-based apps, use Authorization Code with PKCE. This is the recommended approach and requires changes to both the client application and possibly the authorization server configuration.

#### Use Client Credentials for service-to-service.

If the password grant was used for machine-to-machine communication (no actual user), switch to the Client Credentials grant. This does not require user interaction.

#### Custom token exchange as a last resort.

If you must support legacy systems that send username/password, implement a custom token exchange endpoint on your authorization server that accepts credentials and issues tokens. This keeps the anti-pattern in one controlled place while you plan a proper migration.

## Scope Check {.scope-check}

Search for <code>authorization-grant-type: password</code> in YAML/properties files and <code>AuthorizationGrantType.PASSWORD</code> in Java code. Also search for <code>ResourceOwnerPasswordResourceDetails</code> which was the OAuth 2.0 password grant helper class. Any hit means that authentication flow needs redesigning.

## Watch Out {.watch-out}

- The grant is gone from the OAuth 2.1 specification itself, so your authorization server (Keycloak, Auth0, Okta) may drop support too. Check both sides.
- Mobile apps that used password grant to avoid browser redirects should migrate to Authorization Code with PKCE, which modern mobile OAuth SDKs support natively.
- Integration tests that authenticate by posting username/password to the token endpoint will break. Update test helpers to use a test-specific grant flow or mock the security context directly.


## Verify {.verify .pw-page-start}

Token endpoint returns 200 for auth code flow (not password grant)

## Further Info {.further-info}

Removal tracked in spring-security#6003. Affects spring-boot-starter-oauth2-client consumers. See also: pkce-mandatory.

## Links {.footer-links}

- [spring-break module: oauth-password-grant-removed](https://github.com/spoole167/spring-break/tree/main/oauth-password-grant-removed)

- [OAuth Password Grant removal issue](https://github.com/spring-projects/spring-security/issues/6003)

- [Spring Security 7 migration](https://docs.spring.io/spring-security/reference/6.5/migration-7/configuration.html)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

