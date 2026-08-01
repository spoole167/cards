---
id: bootstrap-registry-relocated
tier: 1
tier_label: Won't Build
title: BootstrapRegistry Relocated to org.springframework.boot.bootstrap
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
---

BootstrapRegistry and ConfigurableBootstrapContext moved to org.springframework.boot.bootstrap. The old import path is gone.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/BootstrapRegistryUsage.java:[3,47]
  error: package org.springframework.boot does not contain BootstrapRegistry
```

## What Changed {.what-changed}

<code>BootstrapRegistry</code> and <code>ConfigurableBootstrapContext</code> moved from <code>org.springframework.boot</code> to <code>org.springframework.boot.bootstrap</code>.

## Why {.why-changed}

Spring Boot 4.0 groups its infrastructure by concern: bootstrap lifecycle classes got their own subpackage.

## The Fix {.diffs}

```diff-card
# // BootstrapRegistry import
@@removed
import org.springframework.boot.BootstrapRegistry;
@@added
import org.springframework.boot.bootstrap.BootstrapRegistry;
```

```diff-card
# // ConfigurableBootstrapContext import
@@removed
import org.springframework.boot.ConfigurableBootstrapContext;
@@added
import org.springframework.boot.bootstrap.ConfigurableBootstrapContext;
```

## How To Fix {.fixes}

#### Update imports.

Find all usages of <code>org.springframework.boot.BootstrapRegistry</code> and <code>org.springframework.boot.ConfigurableBootstrapContext</code> and replace with the <code>.bootstrap</code> subpackage. Both are pure package moves: no method signatures or behaviour changed.

## Scope Check {.scope-check}

Search for <code>org.springframework.boot.BootstrapRegistry</code> and <code>org.springframework.boot.ConfigurableBootstrapContext</code> across all Java/Kotlin sources.

## Further Info {.further-info}

Affects code that customises early application startup. Verified in the same spring-break test module as the companion EnvironmentPostProcessor relocation: the two moves go in opposite directions and are otherwise unrelated.

## Links {.footer-links}

- [spring-break module: bootstrap-registry-relocated](https://github.com/spoole167/spring-break/tree/main/bootstrap-registry-relocated)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

