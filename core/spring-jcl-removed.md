---
id: spring-jcl-removed
tier: 1
tier_label: Won't Build
title: spring-jcl Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: true
subsystem: core
---

Spring Framework replaced its spring-jcl logging bridge with a direct dependency on Commons Logging 1.3.0. Manual spring-jcl exclusions break.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] Failed to execute goal on project my-service:
  Could not resolve dependencies:
  The following artifacts could not be resolved:
    org.springframework:spring-jcl:jar:7.0.0 (not found in central)
```

## What Changed {.what-changed}

The <code>org.springframework:spring-jcl</code> module was removed in Spring Framework 7.0. Spring now depends directly on <code>commons-logging:commons-logging:1.3.0</code>, which gained native SLF4J and <code>java.util.logging</code> bridge support. Any POM that references <code>spring-jcl</code>, or excludes <code>commons-logging</code> in its favour, breaks the build.

## Why {.why-changed}

Commons Logging 1.3 added the same SLF4J bridge that spring-jcl provided, making Spring's forked module redundant.

## The Fix {.diffs}

```diff-card
# // pom.xml — remove spring-jcl exclusion/inclusion
@@removed
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
@@added
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <!-- Commons Logging 1.3.0 is pulled in transitively — no exclusion needed -->
</dependency>
```

```diff-card
# // pom.xml — remove explicit spring-jcl dependency
@@removed
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jcl</artifactId>
</dependency>
@@added
<!-- spring-jcl no longer exists. Commons Logging 1.3.0 bridges to SLF4J natively. -->
```

## How To Fix {.fixes}

#### Remove spring-jcl references.

Delete any explicit <code>spring-jcl</code> dependency declarations and any <code>commons-logging</code> exclusions on Spring modules. Commons Logging 1.3.0 bridges to SLF4J out of the box.

#### Remove jcl-over-slf4j.

If you had <code>org.slf4j:jcl-over-slf4j</code> on the classpath, remove it. Commons Logging 1.3.0 provides the same bridge natively. Having both will cause a classpath conflict.

## Scope Check {.scope-check}

Search for <code>spring-jcl</code>, <code>jcl-over-slf4j</code>, and exclusions of <code>commons-logging</code> in all POM and Gradle files. All three are leftovers of the old logging bridge dance.

## Watch Out {.watch-out}

- The classic "commons-logging exclusion + jcl-over-slf4j" pattern every Spring project used for a decade is now harmful: it excludes the Commons Logging 1.3.0 that Spring needs, then substitutes an older SLF4J bridge that may conflict. Remove both.
- If you use a BOM or parent POM that applies global exclusions on commons-logging (common in large enterprises), that exclusion now breaks Spring. Update the corporate parent POM.

## Further Info {.further-info}

A rare migration win: a decade of logging-bridge boilerplate can come out of your POMs.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/spring-jcl-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

