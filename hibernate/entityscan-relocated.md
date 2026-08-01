---
id: entityscan-relocated
tier: 1
tier_label: Won't Build
title: '@EntityScan Relocated to persistence.autoconfigure'
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: hibernate
---

@EntityScan moved from org.springframework.boot.autoconfigure.domain to org.springframework.boot.persistence.autoconfigure. Old import doesn't compile.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/EntityScanApp.java:[3,52]
  error: package org.springframework.boot.autoconfigure.domain does not exist
```

## What Changed {.what-changed}

<code>@EntityScan</code> moved from <code>org.springframework.boot.autoconfigure.domain</code> to <code>org.springframework.boot.persistence.autoconfigure</code>. The annotation's behaviour is unchanged; only the package differs.

## Why {.why-changed}

Spring Boot 4.0 extracted persistence-related auto-configuration into a dedicated <code>spring-boot-persistence</code> module. This gives persistence concerns a cleaner home and makes the module boundaries explicit.

## The Fix {.diffs}

```diff-card
# // Import
@@removed
import org.springframework.boot.autoconfigure.domain.EntityScan;
@@added
import org.springframework.boot.persistence.autoconfigure.EntityScan;
```

## How To Fix {.fixes}

#### Update the import.

Replace <code>org.springframework.boot.autoconfigure.domain.EntityScan</code> with <code>org.springframework.boot.persistence.autoconfigure.EntityScan</code>. One import change per file; no other modifications needed.

## Scope Check {.scope-check}

Search for <code>org.springframework.boot.autoconfigure.domain.EntityScan</code> across all Java/Kotlin sources. Typically appears on main application classes or dedicated JPA configuration classes.

## Watch Out {.watch-out}

- Other classes from <code>org.springframework.boot.autoconfigure.domain</code> may have moved too. If you import anything else from that package, check the Spring Boot 4.0 Migration Guide for the new location.

## Further Info {.further-info}

Several JPA autoconfiguration classes moved together; EntityScan is the most commonly used of the set.

## Links {.footer-links}

- [spring-break module: entityscan-relocated](https://github.com/spoole167/spring-break/tree/main/entityscan-relocated)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

