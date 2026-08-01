---
id: actuator-nullable-removed
tier: 2
tier_label: Won't Run
title: 'Actuator Endpoint @Nullable: org.springframework.lang Replaced by JSpecify'
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: observability
no_module: true
no_module_reason: |
  The Boot 4.0 Migration Guide states actuator endpoint parameters "no longer can use org.springframework.lang.Nullable to declare that a parameter is optional", but this does not reproduce: tested 2026-07-15 on 4.0.0 GA and 4.0.7 via MVC, a @ReadOperation parameter annotated with org.springframework.lang.Nullable is still treated as optional (200 without the parameter). Most likely because Framework 7 meta-annotates the deprecated annotation with JSpecify @Nullable, which actuator recognises transitively. Migrating to org.jspecify.annotations.Nullable remains the right advice, but no module can green-line on 3.5 and red-line on 4.0 for this change.
---

The migration guide says actuator endpoint parameters need org.jspecify.annotations.Nullable, but the deprecated org.springframework.lang.Nullable still works. Migrate as hygiene, not as a fix.

## What You'll See {.error-output}

```error-output
// No error observed. On 4.0.0 and 4.0.7, an endpoint parameter
// annotated with @org.springframework.lang.Nullable is still treated
// as optional: GET without the parameter returns 200 (verified via MVC).
// The migration guide describes a break that does not reproduce.
```

## What Changed {.what-changed}

The Spring Boot 4.0 Migration Guide states that <code>org.springframework.lang.Nullable</code> no longer marks Actuator <code>@ReadOperation</code> / <code>@WriteOperation</code> parameters as optional, and that the supported annotation is now <code>org.jspecify.annotations.Nullable</code> (on the classpath transitively via Spring Framework 7.0). In practice the old annotation is still honoured on 4.0.0 and 4.0.7.

## Why {.why-changed}

Spring Framework 7.0 migrated its codebase to JSpecify null-safety annotations and Spring Boot 4.0 followed. The <code>org.springframework.lang</code> annotations remain for source compatibility but are no longer the active standard.

## The Fix {.diffs}

```diff-card
# // Actuator endpoint parameter
@@removed
import org.springframework.lang.Nullable;

@Endpoint(id = "my-endpoint")
public class MyEndpoint {
    @ReadOperation
    public String get(@Nullable String name) { ... }
}
@@added
import org.jspecify.annotations.Nullable;

@Endpoint(id = "my-endpoint")
public class MyEndpoint {
    @ReadOperation
    public String get(@Nullable String name) { ... }
}
```

## How To Fix {.fixes}

#### Replace org.springframework.lang.Nullable with org.jspecify.annotations.Nullable.

Update the import on all Actuator endpoint parameters. Only the package changes; the annotation name and logic stay the same. JSpecify is already on the classpath via Spring Framework 7.0.

## Scope Check {.scope-check}

Search for <code>@Nullable</code> in classes annotated with <code>@Endpoint</code>, <code>@WebEndpoint</code>, or <code>@JmxEndpoint</code>. The issue is specific to Actuator endpoint parameter binding.

## Watch Out {.watch-out}

- The old annotation works today only because it is meta-annotated with JSpecify's <code>@Nullable</code>. Do not rely on that: it is deprecated, and a later release could stop honouring it. Migrate now while it is a mechanical import swap.

## Verify {.verify}

No observable break: an endpoint parameter annotated with the old @Nullable still returns 200 without the parameter on 4.0.0 and 4.0.7. Migrating to JSpecify is hygiene, not a fix

## Further Info {.further-info}

This is one piece of a wider null-safety overhaul, described in the Spring Boot 4.0 blog post on null-safe applications.

## Links {.footer-links}

- [Migration guide: removed support for org.springframework.lang.Nullable](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide#removed-support-for-javaxannotationsnonnull-and-orgspringframeworklangnullable)

- [Null-safe applications with Spring Boot 4](https://spring.io/blog/2025/11/12/null-safe-applications-with-spring-boot-4)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

