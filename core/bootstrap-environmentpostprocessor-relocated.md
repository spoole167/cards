---
id: bootstrap-environmentpostprocessor-relocated
tier: 1
tier_label: Won't Build
title: EnvironmentPostProcessor Relocated to org.springframework.boot
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
---

EnvironmentPostProcessor moved from org.springframework.boot.env to org.springframework.boot. The spring.factories registration key must also change.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/EnvironmentPostProcessorUsage.java:[3,43]
  error: package org.springframework.boot.env does not contain EnvironmentPostProcessor
```

## What Changed {.what-changed}

<code>EnvironmentPostProcessor</code> moved from <code>org.springframework.boot.env</code> to <code>org.springframework.boot</code>.

## Why {.why-changed}

EnvironmentPostProcessor is used across many contexts, so it moved to the root package for discoverability.

## The Fix {.diffs}

```diff-card
# // EnvironmentPostProcessor import
@@removed
import org.springframework.boot.env.EnvironmentPostProcessor;
@@added
import org.springframework.boot.EnvironmentPostProcessor;
```

## How To Fix {.fixes}

#### Update imports.

Find all usages of <code>org.springframework.boot.env.EnvironmentPostProcessor</code> and replace with <code>org.springframework.boot.EnvironmentPostProcessor</code>. A pure package move: no method signatures or behaviour changed.

#### Search spring.factories / AutoConfiguration.imports.

If you register an <code>EnvironmentPostProcessor</code> in <code>META-INF/spring.factories</code> or <code>META-INF/spring/org.springframework.boot.env.EnvironmentPostProcessor.imports</code>, update the key to <code>org.springframework.boot.EnvironmentPostProcessor</code>.

## Scope Check {.scope-check}

Search for <code>org.springframework.boot.env.EnvironmentPostProcessor</code> across all Java/Kotlin sources and <code>META-INF</code> registration files.

## Watch Out {.watch-out}

- The <code>spring.factories</code> key for <code>EnvironmentPostProcessor</code> must also be updated. With the old key your processor is skipped at startup: no error, your environment customisation never runs.

## Further Info {.further-info}

Affects code that customises early environment configuration. Verified in the same spring-break test module as the companion BootstrapRegistry relocation: the two moves go in opposite directions and are otherwise unrelated.

## Links {.footer-links}

- [spring-break module: bootstrap-registry-relocated](https://github.com/spoole167/spring-break/tree/main/bootstrap-registry-relocated)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

