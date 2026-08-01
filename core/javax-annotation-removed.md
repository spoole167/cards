---
id: javax-annotation-removed
tier: 2
tier_label: Won't Run
title: javax.annotation Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: true
subsystem: core
---

Spring Framework 7.0 stopped processing javax.annotation lifecycle annotations. @PostConstruct and @PreDestroy methods silently never fire. The code compiles and nothing warns you.

## What You'll See {.error-output}

```error-output
// No compile error — javax.annotation-api is still downloadable.
// Failure is silent at runtime:

@PostConstruct
public void init() {
    this.ready = true;  // never called on Boot 4.0
}

// Later in a test or endpoint:
java.lang.NullPointerException: Cannot invoke method because 'this.cache' is null
// or simply: assertions on initialisation state fail unexpectedly
```

## What Changed {.what-changed}

The <code>javax.annotation:javax.annotation-api</code> dependency is no longer on the Spring Boot classpath. Annotations <code>@javax.annotation.PostConstruct</code>, <code>@javax.annotation.PreDestroy</code>, and <code>@javax.annotation.Resource</code> are unavailable. The Jakarta equivalent is <code>jakarta.annotation:jakarta.annotation-api</code>, managed by the Boot BOM. See also: javax-inject-removed.

## Why {.why-changed}

The Jakarta EE transition moved all <code>javax.*</code> APIs to <code>jakarta.*</code>. Spring Framework 7.0 completed this transition by dropping the legacy <code>javax.annotation</code> artifact.

## The Fix {.diffs}

```diff-card
# // Java imports
@@removed
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;
@@added
import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
```

```diff-card
# // pom.xml — if you had an explicit dependency
@@removed
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
@@added
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
</dependency>
```

## How To Fix {.fixes}

#### Switch to jakarta.annotation.

Replace <code>javax.annotation</code> imports with <code>jakarta.annotation</code>. The annotations are identical: only the package name changed. The <code>jakarta.annotation-api</code> artifact is managed by the Spring Boot BOM so no version is needed.

#### Remove explicit dependency version.

If you declared <code>javax.annotation-api</code> explicitly in your POM, replace it with <code>jakarta.annotation-api</code> and omit the version.

## Scope Check {.scope-check}

Search for <code>javax.annotation</code> in all Java/Kotlin sources and build files. Pay particular attention to <code>@PostConstruct</code>: it appears in almost every service class with initialisation logic. Also check for <code>javax.annotation:javax.annotation-api</code> in POM files and Gradle build scripts.

## Watch Out {.watch-out}

- <code>@PostConstruct</code> is skipped if the import is wrong: no error at startup, the method never runs. If your cache, connection pool, or scheduled task isn't initialising, check the import first.
- <code>@Resource</code> (JSR-250 named injection) also lives in <code>javax.annotation</code>. If you use it for JNDI lookups or named bean injection, that import needs updating too.

## Verify {.verify}

App starts and @PostConstruct methods fire: confirm via log output or assert that initialisation state is set after context loads

## Further Info {.further-info}

Driven by the Jakarta EE transition completed in Spring Framework 7.0. The javax.annotation:javax.annotation-api artifact still exists in Maven Central and still compiles. The break is at runtime, and @Resource injection stops working the same way.

## Links {.footer-links}

- [spring-break module: javax-annotation-removed](https://github.com/spoole167/spring-break/tree/main/javax-annotation-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

