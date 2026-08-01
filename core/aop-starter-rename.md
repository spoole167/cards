---
id: aop-starter-rename
tier: 1
tier_label: Won't Build
title: AOP Starter Renamed to spring-boot-starter-aspectj
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
---

spring-boot-starter-aop is no longer in the Spring Boot 4.0 BOM. Replace it with spring-boot-starter-aspectj.

## What You'll See {.error-output}

```error-output
$ mvn validate
[ERROR] 'dependencies.dependency.version' for
  org.springframework.boot:spring-boot-starter-aop:jar is missing.
```

## What Changed {.what-changed}

<code>spring-boot-starter-aop</code> has been removed from the Spring Boot BOM. The replacement, <code>spring-boot-starter-aspectj</code>, provides the same AspectJ weaving capabilities.

## Why {.why-changed}

Spring AOP (interface-proxy and CGLIB-proxy based AOP) works without any starter. The old name implied you needed this starter for any AOP, when you only need it for AspectJ load-time or compile-time weaving. The rename makes the distinction clear.

## The Fix {.diffs}

```diff-card
# // pom.xml dependency
@@removed
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
@@added
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aspectj</artifactId>
</dependency>
```

## How To Fix {.fixes}

#### Replace the starter in pom.xml / build.gradle.

Find <code>spring-boot-starter-aop</code> in all build files and replace with <code>spring-boot-starter-aspectj</code>. No code changes needed: the classpath content is equivalent.

## Scope Check {.scope-check}

Search all <code>pom.xml</code> and <code>build.gradle</code> files for <code>spring-boot-starter-aop</code>. Multi-module projects may declare it in several places.

## Watch Out {.watch-out}

- If you only use Spring's proxy-based AOP (<code>@Aspect</code> beans, <code>@Transactional</code>, etc.) you may not need this starter at all. Proxy-based AOP is included in <code>spring-boot-starter</code> transitively. The aspectj starter is only required if you use AspectJ weaving agents.

## Further Info {.further-info}

This is a T1B failure (artifact no longer in the BOM): the build fails at dependency resolution, before any source compiles.

## Links {.footer-links}

- [spring-break module: aop-starter-rename](https://github.com/spoole167/spring-break/tree/main/aop-starter-rename)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

