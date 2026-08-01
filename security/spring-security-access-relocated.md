---
id: spring-security-access-relocated
tier: 1
tier_label: Won't Build
title: Spring Security Access API Moved to spring-security-access
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: security
---

AccessDecisionManager and related legacy authorisation classes moved to a separate spring-security-access module. Add it or migrate to AuthorizationManager.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/AccessApiUsage.java:[3,48]
  error: package org.springframework.security.access does not exist
```

## What Changed {.what-changed}

<code>AccessDecisionManager</code>, <code>AccessDecisionVoter</code>, <code>@EnableGlobalMethodSecurity</code>, and related legacy authorisation classes have been moved from <code>spring-security-core</code> to a new standalone <code>spring-security-access</code> artifact.

## Why {.why-changed}

Spring Security 7.0 makes <code>AuthorizationManager</code> the sole supported authorisation API. The legacy Access API survives for migration, isolated in its own module so finished migrations don't carry the dead weight.

## The Fix {.diffs}

```diff-card
# // pom.xml — add legacy module if needed
@@added
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-access</artifactId>
</dependency>
```

```diff-card
# // Preferred: migrate to AuthorizationManager
@@removed
import org.springframework.security.access.AccessDecisionManager;
// implements AccessDecisionManager
@@added
import org.springframework.security.authorization.AuthorizationManager;
// implements AuthorizationManager<RequestAuthorizationContext>
```

## How To Fix {.fixes}

#### Add spring-security-access (short-term fix).

Add <code>spring-security-access</code> as an explicit dependency. This restores compilation without any logic changes and buys time to do the full migration.

#### Migrate to AuthorizationManager (recommended).

Replace <code>AccessDecisionManager</code> / <code>AccessDecisionVoter</code> implementations with <code>AuthorizationManager</code>. Replace <code>@EnableGlobalMethodSecurity</code> with <code>@EnableMethodSecurity</code>. The Spring Security migration guide has step-by-step instructions.

## Scope Check {.scope-check}

Search for <code>AccessDecisionManager</code>, <code>AccessDecisionVoter</code>, <code>@EnableGlobalMethodSecurity</code>, and <code>org.springframework.security.access</code> imports across all sources.

## Watch Out {.watch-out}

- <code>@EnableGlobalMethodSecurity</code> is also in the legacy module. If you use method security, you need either the legacy module or to migrate to <code>@EnableMethodSecurity</code> before removing it.

## Further Info {.further-info}

Driven by Spring Security 7.0; the spring.io announcement in the footer covers the full background.

## Links {.footer-links}

- [spring-break module: spring-security-access-relocated](https://github.com/spoole167/spring-break/tree/main/spring-security-access-relocated)

- [Spring Security access API moves](https://spring.io/blog/2025/09/09/access-api-moves-to-spring-security-access)

