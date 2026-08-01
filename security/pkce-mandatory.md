---
id: pkce-mandatory
tier: 2
tier_label: Won't Run
title: PKCE Mandatory for Confidential OAuth Clients
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: security
---

Spring Security 7 enforces PKCE for all OAuth 2.0 clients, including confidential clients. Authorization requests without a code_challenge are rejected by the authorization server.

## What You'll See {.error-output}

```error-output
[oauth2-client] o.s.s.oauth2.client.web.OAuth2LoginAuthenticationFilter:
  Authentication request failed:
org.springframework.security.oauth2.core.OAuth2AuthenticationException:
  [invalid_request] PKCE code_challenge is required for this client.
 at org.springframework.security.oauth2.client.oidc.authentication
  .OidcAuthorizationCodeAuthenticationProvider.authenticate(...)
---
HTTP 302 → /login?error=invalid_request
---
Browser shows: "OAuth2 Error: [invalid_request]
PKCE code_challenge is required for this client."
```

## What Changed {.what-changed}

Spring Security 7 now sends PKCE parameters (<code>code_challenge</code> and <code>code_challenge_method</code>) on every Authorization Code request, for both public and confidential clients. If the authorization server requires PKCE but the client was not sending it (pre-upgrade behaviour for confidential clients), or if the authorization server rejects unexpected PKCE parameters, the flow breaks.

## Why {.why-changed}

OAuth 2.1 (RFC 9700) mandates PKCE for all clients, public and confidential alike, because PKCE prevents authorization code interception attacks regardless of client type. Spring Security aligned with that requirement.

## The Fix {.diffs}

```diff-card
# // application.yml — no changes needed on client side
@@removed
spring:
  security:
    oauth2:
      client:
        registration:
          my-app:
            authorization-grant-type: authorization_code
            client-id: my-confidential-client
            client-secret: ${CLIENT_SECRET}
            scope: openid,profile
@@added
# Client config stays the same — Spring Security 7 adds PKCE automatically.
# But your authorization server MUST accept PKCE parameters.
spring:
  security:
    oauth2:
      client:
        registration:
          my-app:
            authorization-grant-type: authorization_code
            client-id: my-confidential-client
            client-secret: ${CLIENT_SECRET}
            scope: openid,profile
```

```diff-card
# // Keycloak realm config — enable PKCE for confidential client
@@removed
"clientId": "my-confidential-client",
"publicClient": false,
"pkceCodeChallengeMethod": ""
@@added
"clientId": "my-confidential-client",
"publicClient": false,
"pkceCodeChallengeMethod": "S256"
```

```diff-card
# // Disable PKCE enforcement if auth server can't support it (temporary)
@@removed
// default PKCE behaviour
@@added
@Bean
public OAuth2AuthorizationRequestResolver pkceResolver(
        ClientRegistrationRepository repo) {
    DefaultOAuth2AuthorizationRequestResolver resolver =
        new DefaultOAuth2AuthorizationRequestResolver(
            repo, "/oauth2/authorization");
    // Temporary: remove when auth server supports PKCE
    resolver.setAuthorizationRequestCustomizer(
        OAuth2AuthorizationRequestCustomizers.withoutPkce());
    return resolver;
}
```

## How To Fix {.fixes}

#### Enable PKCE on your authorization server.

Configure your authorization server (Keycloak, Auth0, Okta, Azure AD) to accept PKCE <code>code_challenge</code> parameters for confidential clients. Most modern providers support this; it may need enabling per client. Use <code>S256</code> as the challenge method.

#### Verify no auth server rejects extra parameters.

Some older or custom authorization servers reject unknown parameters. If yours rejects <code>code_challenge</code>, you need to update the auth server first, or temporarily disable PKCE enforcement on the Spring Security side using a custom resolver.

#### Disable PKCE temporarily (not recommended).

If you cannot update the authorization server immediately, register a custom <code>OAuth2AuthorizationRequestResolver</code> that strips PKCE parameters. Treat this as technical debt and plan the auth server update.

## Scope Check {.scope-check}

Check every OAuth 2.0 client registration in your application. For each one, verify that the corresponding authorization server accepts PKCE parameters. If you use Spring Authorization Server, it already supports PKCE. If you use a third-party provider, check their documentation.

## Watch Out {.watch-out}

- The failure often appears only in the browser redirect flow. Automated tests that mock the OAuth exchange may not trigger the error. Test with a real authorization server.
- If you use Spring Authorization Server as your auth server and it is also being upgraded, both sides get PKCE support automatically. The risk is when the client (Spring Boot 4) and the auth server (third-party or older version) are upgraded independently.
- Azure AD and some ADFS versions may need tenant-level configuration changes to allow PKCE for confidential clients. Check your identity provider's documentation.

## Verify {.verify}

OAuth2 login completes with PKCE challenge in the auth request

## Further Info {.further-info}

PKCE enforcement was tracked in spring-security#16391. Affects spring-boot-starter-oauth2-client consumers using the authorization_code grant. See also: oauth-password-grant.

## Links {.footer-links}

- [spring-break module: pkce-mandatory](https://github.com/spoole167/spring-break/tree/main/pkce-mandatory)

- [PKCE enforcement issue](https://github.com/spring-projects/spring-security/issues/16391)

- [Spring Security 7 migration](https://docs.spring.io/spring-security/reference/6.5/migration-7/configuration.html)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

