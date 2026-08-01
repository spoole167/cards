---
id: simpdest-message-matcher-removed
tier: 1
tier_label: Won't Build
title: SimpDestinationMessageMatcher Removed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: security
---

SimpDestinationMessageMatcher is gone in Spring Security 7.0. WebSocket security now uses the AuthorizationManager pattern.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/SimpDestUsage.java:[3,66]
  error: package org.springframework.security.messaging.util.matcher
  does not contain SimpDestinationMessageMatcher
```

## What Changed {.what-changed}

<code>org.springframework.security.messaging.util.matcher.SimpDestinationMessageMatcher</code> has been removed. Spring Security 7.0 replaces the old <code>AbstractSecurityWebSocketMessageBrokerConfigurer</code> / matcher-based approach with <code>MessageMatcherDelegatingAuthorizationManager</code>.

## Why {.why-changed}

The old configurer and matcher classes were deprecated in Security 6.x. 7.0 removed them, completing the move to AuthorizationManager.

## The Fix {.diffs}

```diff-card
# // Old — matcher-based WebSocket security config
@@removed
import org.springframework.security.messaging.util.matcher.SimpDestinationMessageMatcher;
// ...
new SimpDestinationMessageMatcher("/topic/**");
```

```diff-card
# // New — AuthorizationManager-based config
@@added
import org.springframework.security.messaging.access.intercept.MessageMatcherDelegatingAuthorizationManager;
// ...
MessageMatcherDelegatingAuthorizationManager.builder()
    .simpDestMatchers("/topic/**").authenticated()
    .anyMessage().denyAll()
    .build();
```

## How To Fix {.fixes .pw-page-start}

#### Migrate to MessageMatcherDelegatingAuthorizationManager.

Replace any use of <code>SimpDestinationMessageMatcher</code> and <code>AbstractSecurityWebSocketMessageBrokerConfigurer</code> with a <code>MessageMatcherDelegatingAuthorizationManager</code> bean registered on your message broker. See the Spring Security 7 messaging migration guide for the full configuration pattern.

## Scope Check {.scope-check}

Search for <code>SimpDestinationMessageMatcher</code> and <code>AbstractSecurityWebSocketMessageBrokerConfigurer</code>. Both are removed. Only applications using STOMP over WebSocket with Spring Security are affected.

## Watch Out {.watch-out}

- The new AuthorizationManager must be wired into the message broker configuration explicitly. There is no auto-migration path: the configuration structure changes substantially.

## Further Info {.further-info}

Part of Spring Security 7.0's WebSocket security overhaul: the same AuthorizationManager pattern now applies across HTTP, method, and message security.

## Links {.footer-links}

- [spring-break module: simpdest-message-matcher-removed](https://github.com/spoole167/spring-break/tree/main/simpdest-message-matcher-removed)

- [Spring Security 7 messaging migration](https://docs.spring.io/spring-security/reference/6.5/migration-7/messaging.html)

