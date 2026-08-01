---
id: testcontainers-class-relocation
tier: 1
tier_label: Won't Build
title: Testcontainers 2.0 Package Relocation
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: true
subsystem: testing
---

Testcontainers 2.0 moved database and middleware containers into module-specific sub-packages (e.g. org.testcontainers.postgresql). The Maven group ID stays org.testcontainers.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/test/java/com/example/IntegrationTest.java:[4,52]
  error: cannot find symbol
    symbol:   class PostgreSQLContainer
    location: package org.testcontainers.containers
```

## What Changed {.what-changed}

Testcontainers 2.0 resolved JPMS split-package problems by moving specialised containers into module-specific packages. <code>PostgreSQLContainer</code> moved from <code>org.testcontainers.containers</code> to <code>org.testcontainers.postgresql</code>. Similar moves apply to all database and middleware modules (Kafka, Redis, RabbitMQ, etc.). <code>GenericContainer</code> remains in <code>org.testcontainers.containers</code>. The Maven group ID (<code>org.testcontainers</code>) is unchanged.

## Why {.why-changed}

JPMS forbids multiple JARs contributing to the same package (split packages). In Testcontainers 1.x every module dumped its containers into <code>org.testcontainers.containers</code>, making modular JARs impossible. The 2.0 sub-package split is a prerequisite for Java 17+ module support.

## The Fix {.diffs}

```diff-card
# // pom.xml — BOM: group ID stays org.testcontainers; remove explicit version if using Boot parent
@@removed
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers-bom</artifactId>
    <version>1.19.8</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
@@added
<!-- Managed by Spring Boot 4.0 parent POM — remove explicit BOM if using starter-parent -->
```

```diff-card
# // Java imports — specialised containers move to module sub-packages
@@removed
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.containers.KafkaContainer;
@@added
import org.testcontainers.postgresql.PostgreSQLContainer;
import org.testcontainers.kafka.KafkaContainer;
```

```diff-card
# // GenericContainer stays put — no change needed
@@removed
import org.testcontainers.containers.GenericContainer;
@@added
import org.testcontainers.containers.GenericContainer;  // unchanged
```

## How To Fix {.fixes}

#### Update Java imports for specialised containers.

For each database or middleware container, change the import from <code>org.testcontainers.containers.XxxContainer</code> to the module-specific sub-package, e.g. <code>org.testcontainers.postgresql.PostgreSQLContainer</code>, <code>org.testcontainers.kafka.KafkaContainer</code>, <code>org.testcontainers.redis.RedisContainer</code>. Check the <a href="https://testcontainers.com/guides/testcontainers-2-migration/">Testcontainers 2.0 migration guide</a> for the full mapping.

#### Remove explicit BOM version if using Spring Boot parent.

Spring Boot 4.0's parent POM manages the Testcontainers 2.0 version. If you previously imported the Testcontainers BOM explicitly with a 1.x version, remove the version override to avoid conflicts.

## Scope Check {.scope-check}

Search for <code>org.testcontainers.containers</code> in Java/Kotlin test sources. Any import of a specialised container (PostgreSQL, MySQL, Kafka, Redis, etc.) from that package needs updating to its new module sub-package. <code>GenericContainer</code> does not need changing.

## Watch Out {.watch-out}

- The Maven group ID is still <code>org.testcontainers</code>; do not change it to <code>com.testcontainers</code>. Only the Java package paths inside the JARs changed.
- Spring Boot's <code>@ServiceConnection</code> annotation resolves container types by class. If you reference the old <code>org.testcontainers.containers.PostgreSQLContainer</code> import in a <code>@ServiceConnection</code> field, updating the import is sufficient; no annotation change needed.
- If you use Testcontainers in a shared test library, update that module first. A mix of 1.x and 2.x imports on the classpath will cause <code>ClassNotFoundException</code> at runtime.

## Further Info {.further-info}

Driven by Testcontainers 2.0, the version Spring Boot 4.0 now manages.

## Links {.footer-links}

- [spring-break module: testcontainers-class-relocation](https://github.com/spoole167/spring-break/tree/main/testcontainers-class-relocation)

- [Testcontainers 2.0 migration guide](https://testcontainers.com/guides/testcontainers-2-migration-guide/)

