---
id: javax-inject-removed
tier: 2
tier_label: Won't Run
title: javax.inject Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: true
subsystem: core
---

Spring Framework 7.0 stopped processing javax.inject annotations. @Inject fields silently stay null. @Named beans may not be registered. The code compiles, the app starts, and nothing warns you.

## What You'll See {.error-output}

```error-output
// No compile error — javax.inject:javax.inject still downloadable.
// Failure is silent at runtime:

@Named("orderService")
public class OrderService {
    @Inject
    private OrderRepository repo;  // null on Boot 4.0 — never injected
}

// At call time:
java.lang.NullPointerException: Cannot invoke
  "com.example.OrderRepository.findById(Long)"
  because "this.repo" is null
```

## What Changed {.what-changed}

The <code>javax.inject:javax.inject</code> dependency is no longer on the Spring Boot classpath. Annotations like <code>@javax.inject.Inject</code>, <code>@javax.inject.Named</code>, and <code>@javax.inject.Singleton</code> are unavailable. The Jakarta equivalent is <code>jakarta.inject:jakarta.inject-api</code>.

## Why {.why-changed}

The Jakarta EE transition moved all <code>javax.*</code> APIs to <code>jakarta.*</code>. Spring Framework 7.0 completed this transition by dropping support for the legacy <code>javax.inject</code> artifact.

## The Fix {.diffs}

```diff-card
# // Java imports
@@removed
import javax.inject.Inject;
import javax.inject.Named;
@@added
import jakarta.inject.Inject;
import jakarta.inject.Named;
```

```diff-card
# // pom.xml — if you had an explicit dependency
@@removed
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
@@added
<dependency>
    <groupId>jakarta.inject</groupId>
    <artifactId>jakarta.inject-api</artifactId>
</dependency>
```

```diff-card
# // Or just use Spring annotations
@@removed
@Named("orderService")
public class OrderService {
    @Inject
    private OrderRepository repo;
@@added
@Service("orderService")
public class OrderService {
    @Autowired
    private OrderRepository repo;
```

## How To Fix {.fixes}

#### Switch to jakarta.inject.

Replace <code>javax.inject</code> imports with <code>jakarta.inject</code>. The annotations are identical: only the package name changed. The <code>jakarta.inject-api</code> artifact is managed by the Spring Boot BOM.

#### Switch to Spring annotations (recommended).

Replace <code>@Inject</code> with <code>@Autowired</code> and <code>@Named</code> with <code>@Component</code> / <code>@Service</code> / <code>@Qualifier</code>. This removes the dependency on the inject API entirely.

## Scope Check {.scope-check}

Search for <code>javax.inject</code> in all Java/Kotlin sources and build files. Every <code>import javax.inject</code> statement needs updating. Also check for <code>javax.inject:javax.inject</code> in POM files.

## Watch Out {.watch-out}

- If you're using <code>@Inject</code> for CDI compatibility (e.g., shared code between Spring and a Jakarta EE server), switch to <code>jakarta.inject</code>: it works in both environments. Don't switch to Spring-specific annotations if portability matters.
- Constructor injection with <code>@Inject</code> on a single-constructor class can drop the annotation entirely. Spring auto-detects single-constructor injection since 4.3.

## Verify {.verify}

App starts and @Inject fields are non-null: confirm via a test that autowires a bean registered with @Named

## Further Info {.further-info}

Driven by the Jakarta EE transition completed in Spring Framework 7.0. The javax.inject:javax.inject:1 artifact still exists in Maven Central and still compiles. The break is at runtime, and the first NullPointerException at call time is your only clue.

## Links {.footer-links}

- [spring-break module: javax-inject-removed](https://github.com/spoole167/spring-break/tree/main/javax-inject-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

