---
id: tomcat-war-deployment
tier: 1
tier_label: Won't Build
title: Tomcat WAR Deployment Requires New Runtime Starter
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: web
no_module: true
no_module_reason: |
  A packaging break, not a compilation failure. Demonstrating it means building a WAR and deploying it to an external Tomcat server, which the spring-break format (compile plus unit tests) cannot exercise.
---

Boot 4.0 adds <code>spring-boot-starter-tomcat-runtime</code> for WAR deployment. The old provided-scope <code>spring-boot-starter-tomcat</code> pattern may conflict with the external container's classpath.

## What You'll See {.error-output}

```error-output
// Deploying WAR to external Tomcat with Boot 3.5 dependency pattern:
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>

// Boot 4.0: may result in:
java.lang.ClassNotFoundException: jakarta.servlet.FilterRegistration
// Or: version conflicts between embedded and container Tomcat classes
```

## What Changed {.what-changed}

A new <code>spring-boot-starter-tomcat-runtime</code> artifact covers WAR deployment. It provides the runtime bridge classes needed to run a Boot application inside an external Tomcat, without the full embedded Tomcat server dependencies. <code>spring-boot-starter-tomcat</code> is now exclusively for embedded use.

## Why {.why-changed}

Embedded Tomcat and external container are distinct deployment models with different runtime dependencies. The old trick of marking <code>spring-boot-starter-tomcat</code> as <code>provided</code> blurred that line and was error-prone.

## The Fix {.diffs}

```diff-card
# // pom.xml — WAR deployment dependency update
@@removed
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
@@added
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat-runtime</artifactId>
    <scope>provided</scope>
</dependency>
```

## How To Fix {.fixes}

#### Replace spring-boot-starter-tomcat (provided) with spring-boot-starter-tomcat-runtime.

In your WAR project's <code>pom.xml</code>, replace the provided-scope <code>spring-boot-starter-tomcat</code> dependency with <code>spring-boot-starter-tomcat-runtime</code>. Keep the <code>provided</code> scope: the external container supplies Tomcat at runtime.

## Scope Check {.scope-check}

Search <code>pom.xml</code> for <code>spring-boot-starter-tomcat</code> with <code>scope=provided</code>. Also check <code>SpringBootServletInitializer</code> subclasses: any WAR project using that class is affected.

## Watch Out {.watch-out}

- If your project also runs as an executable JAR (dual deployment mode), keep <code>spring-boot-starter-tomcat</code> for the embedded server but add <code>spring-boot-starter-tomcat-runtime</code> as provided.

## Verify {.verify}

WAR file deploys to external Tomcat without ClassNotFoundException or dependency conflicts after swapping to spring-boot-starter-tomcat-runtime

## Further Info {.further-info}

The Boot 3.5 pattern, <code>spring-boot-starter-tomcat</code> at <code>provided</code> scope, can still work on 4.0. Mixing it with the new split risks classpath conflicts or <code>ClassNotFoundException</code> at deployment time.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

