---
id: undertow-removed
tier: 1
tier_label: Won't Build
title: Undertow Embedded Server Removed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: web
---

Spring Boot 4.0 dropped Undertow as an embedded server option. Projects depending on spring-boot-starter-undertow won't resolve.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] Failed to execute goal on project my-service:
  Could not resolve dependencies for project com.example:my-service:jar:1.0.0:
  The following artifacts could not be resolved:
    org.springframework.boot:spring-boot-starter-undertow:jar:4.0.0 (not found)
```

## What Changed {.what-changed}

The <code>spring-boot-starter-undertow</code> module was removed from Spring Boot 4.0. Undertow is no longer a supported embedded servlet container. The supported options are Tomcat (default), Jetty, and Netty (for reactive).

## Why {.why-changed}

Undertow's development slowed and its Jakarta EE 11 support lagged behind Tomcat and Jetty. Maintaining a third servlet container integration stopped being worth the support burden.

## The Fix {.diffs}

```diff-card
# // pom.xml — remove Undertow starter
@@removed
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
@@added
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- Uses Tomcat by default. For Jetty, add spring-boot-starter-jetty
     and exclude spring-boot-starter-tomcat -->
```

```diff-card
# // application.yml — Undertow-specific config
@@removed
server:
  undertow:
    io-threads: 8
    worker-threads: 64
    buffer-size: 1024
@@added
server:
  tomcat:
    threads:
      max: 200
      min-spare: 10
```

## How To Fix {.fixes}

#### Switch to Tomcat (default).

Remove the <code>spring-boot-starter-undertow</code> dependency and the Tomcat exclusion. Tomcat is the default: <code>spring-boot-starter-web</code> alone is enough.

#### Switch to Jetty.

If you need a non-Tomcat server, use <code>spring-boot-starter-jetty</code> instead. Exclude the Tomcat starter and add the Jetty starter.

## Scope Check {.scope-check}

Search for <code>spring-boot-starter-undertow</code> in POM and Gradle files. Also search for <code>server.undertow</code> in application properties/YAML files: those properties won't cause a build failure but will be silently ignored.

## Watch Out {.watch-out}

- Undertow-specific configuration properties under <code>server.undertow.*</code> have no Tomcat or Jetty equivalent. Review your <code>application.yml</code> and translate settings like <code>io-threads</code>, <code>worker-threads</code>, and <code>buffer-size</code> to the new server's configuration model.
- If you chose Undertow for its non-blocking I/O on the servlet side, consider switching to Spring WebFlux with Netty instead of forcing the same architecture onto Tomcat.

## Verify {.verify}

App starts on Tomcat/Jetty with no Undertow class errors

## Further Info {.further-info}

One of the few 4.0 breaks with no like-for-like replacement: you must change servers.

## Links {.footer-links}

- [spring-break module: undertow-removed](https://github.com/spoole167/spring-break/tree/main/undertow-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

