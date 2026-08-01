---
id: apacheds-ldap-removed
tier: 1
tier_label: Won't Build
title: ApacheDS Embedded LDAP Support Removed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: security
---

ApacheDSContainer is removed in Spring Security 7.0. Use UnboundIdContainer instead.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/ApacheDsUsage.java:[3,57]
  error: package org.springframework.security.ldap.server does not contain
  ApacheDSContainer
```

## What Changed {.what-changed}

<code>org.springframework.security.ldap.server.ApacheDSContainer</code> has been removed from Spring Security. The embedded LDAP server capability is now provided exclusively through <code>org.springframework.security.ldap.server.UnboundIdContainer</code>.

## Why {.why-changed}

ApacheDS development had stalled and its Jakarta EE compatibility lagged. UnboundID is actively maintained and provides equivalent functionality for embedded LDAP testing and development use cases.

## The Fix {.diffs}

```diff-card
# // pom.xml — swap the embedded LDAP dependency
@@removed
<dependency>
    <groupId>org.apache.directory.server</groupId>
    <artifactId>apacheds-server-jndi</artifactId>
    <scope>test</scope>
</dependency>
@@added
<dependency>
    <groupId>com.unboundid</groupId>
    <artifactId>unboundid-ldapsdk</artifactId>
    <scope>test</scope>
</dependency>
```

```diff-card
# // Java config — replace the container class
@@removed
import org.springframework.security.ldap.server.ApacheDSContainer;
// ...
new ApacheDSContainer("dc=springframework,dc=org", "classpath:users.ldif");
@@added
import org.springframework.security.ldap.server.UnboundIdContainer;
// ...
new UnboundIdContainer("dc=springframework,dc=org", "classpath:users.ldif");
```

## How To Fix {.fixes}

#### Replace ApacheDSContainer with UnboundIdContainer.

Swap the dependency in your build file and update the import and instantiation. The constructor signature is the same: base DN and LDIF classpath location. The LDIF format is also compatible.

#### Use Spring Boot's embedded LDAP auto-configuration.

If you're only using embedded LDAP for tests, add <code>spring-boot-starter-ldap</code> and set <code>spring.ldap.embedded.base-dn</code> in test properties. Spring Boot will auto-configure UnboundID for you.

## Scope Check {.scope-check}

Search for <code>ApacheDSContainer</code> and <code>apacheds-server-jndi</code> (or other ApacheDS artifacts) across Java sources and build files.

## Watch Out {.watch-out}

- LDIF files written for ApacheDS should work with UnboundID, but verify any vendor-specific schema extensions you may have used.

## Further Info {.further-info}

Both containers were available in 3.5; only UnboundID survives in 4.0.

## Links {.footer-links}

- [spring-break module: apacheds-ldap-removed](https://github.com/spoole167/spring-break/tree/main/apacheds-ldap-removed)

- [Spring Security 7 LDAP migration](https://docs.spring.io/spring-security/reference/6.5/migration-7/ldap.html)

