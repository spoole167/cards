---
id: propertymapping-relocated
tier: 1
tier_label: Won't Build
title: '@PropertyMapping Relocated to test.context'
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
---

@PropertyMapping moved from org.springframework.boot.test.autoconfigure.properties to org.springframework.boot.test.context. Old import doesn't compile.

## What You'll See {.error-output}

```error-output
$ mvn test-compile
[ERROR] /src/test/java/com/example/CustomTestAnnotation.java:[3,62]
  error: package org.springframework.boot.test.autoconfigure.properties does not exist
```

## What Changed {.what-changed}

<code>@PropertyMapping</code> moved from <code>org.springframework.boot.test.autoconfigure.properties</code> to <code>org.springframework.boot.test.context</code>. Behaviour is unchanged.

## Why {.why-changed}

Spring Boot 4.0 consolidated test-context infrastructure into a more coherent package hierarchy. The separate <code>test.autoconfigure.properties</code> package was collapsed into <code>test.context</code>.

## The Fix {.diffs}

```diff-card
# // Import on custom test annotation
@@removed
import org.springframework.boot.test.autoconfigure.properties.PropertyMapping;
@@added
import org.springframework.boot.test.context.PropertyMapping;
```

## How To Fix {.fixes}

#### Update the import.

Replace <code>org.springframework.boot.test.autoconfigure.properties.PropertyMapping</code> with <code>org.springframework.boot.test.context.PropertyMapping</code>. This typically appears only in custom meta-annotations for test slices.

## Scope Check {.scope-check}

Search for <code>@PropertyMapping</code> and the old import path in your test source tree. Only affects teams that have written custom test slice annotations; most applications won't have this at all.

## Watch Out {.watch-out}

- If you have a shared test-utilities library that defines custom slices, the fix must be applied there rather than in the consuming projects. A single unfixed library breaks all consumers.

## Further Info {.further-info}

@PropertyMapping is used when authoring custom test slice annotations. It binds annotation attributes to Spring Boot auto-configuration properties. Several other test-autoconfigure classes moved in the same cleanup.

## Links {.footer-links}

- [spring-break module: propertymapping-relocated](https://github.com/spoole167/spring-break/tree/main/propertymapping-relocated)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

