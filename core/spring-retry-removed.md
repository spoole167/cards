---
id: spring-retry-removed
tier: 1
tier_label: Won't Build
title: spring-retry Removed from BOM
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
---

Spring Boot 4.0 removed spring-retry from its managed dependency BOM. Declare an explicit version or the build fails.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] 'dependencies.dependency.version' for org.springframework.retry:spring-retry:jar
  is missing. @ line 42, column 17
[ERROR] Failed to execute goal on project my-service:
  Could not resolve dependencies: Failed to collect dependencies at
  org.springframework.retry:spring-retry:jar:RELEASE (no version)
```

## What Changed {.what-changed}

<code>org.springframework.retry:spring-retry</code> is no longer version-managed by the Spring Boot BOM. Any declaration without an explicit <code>&lt;version&gt;</code> tag fails because Maven/Gradle can't resolve it.

## Why {.why-changed}

Spring Retry's release cadence diverged from Spring Boot's. Managing it in the BOM implied a level of compatibility testing that wasn't happening. Removing it from the BOM lets teams pin the version they test against.

## The Fix {.diffs}

```diff-card
# // pom.xml — add explicit version
@@removed
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
@@added
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
    <version>2.0.11</version>
</dependency>
```

```diff-card
# // build.gradle — add explicit version
@@removed
implementation 'org.springframework.retry:spring-retry'
@@added
implementation 'org.springframework.retry:spring-retry:2.0.11'
```

## How To Fix {.fixes}

#### Add an explicit version.

Add <code>&lt;version&gt;2.0.11&lt;/version&gt;</code> (or the latest compatible release) to your spring-retry dependency declaration. Check <a href="https://mvnrepository.com/artifact/org.springframework.retry/spring-retry">Maven Central</a> for the latest version.

#### Import Spring Retry's own BOM.

If multiple modules use spring-retry, add its BOM to your <code>&lt;dependencyManagement&gt;</code> section so versions are managed in one place.

## Scope Check {.scope-check}

Search for <code>spring-retry</code> in POM and Gradle files. Any declaration without an explicit version will fail. Also check for <code>@Retryable</code> and <code>@EnableRetry</code> annotations: these confirm the library is in use.

## Watch Out {.watch-out}

- If you use <code>spring-cloud-starter-circuitbreaker-resilience4j</code>, it may have pulled in spring-retry transitively. With the BOM removal, the transitive version is no longer aligned. Pin it explicitly to avoid classpath surprises.
- Spring Retry 2.x requires Spring Framework 6.0+. Since Boot 4.0 ships Framework 7.0, use Retry 2.0.x: older 1.x versions are incompatible.

## Verify {.verify}

mvn dependency:tree shows aligned Spring Retry version and retry works

## Further Info {.further-info}

The library itself is unchanged and still maintained; only Boot's version management went away. See also: retry-semantics, retryable-transaction-order.

## Links {.footer-links}

- [spring-break module: spring-retry-removed](https://github.com/spoole167/spring-break/tree/main/spring-retry-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

