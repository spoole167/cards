---
id: spring-session-mongodb-removed
tier: 2
tier_label: Won't Run
title: Spring Session MongoDB Auto-Configuration Removed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: data-messaging
---

Spring Boot 4.0 dropped built-in MongoDB session auto-configuration. The app compiles but fails at startup with <code>NoSuchBeanDefinitionException</code>.

## What You'll See {.error-output}

```error-output
// Boot 4.0 startup failure:
org.springframework.beans.factory.NoSuchBeanDefinitionException:
  No qualifying bean of type
  'org.springframework.session.SessionRepository' available

// Or context load failure mentioning missing MongoIndexedSessionRepository
```

## What Changed {.what-changed}

Spring Boot's auto-configuration for MongoDB-backed Spring Session was removed from <code>spring-boot-autoconfigure</code>. The <code>MongoSessionConfiguration</code> and related auto-configuration classes no longer exist in the Spring Boot artifact. Spring Data MongoDB took ownership of the integration.

## Why {.why-changed}

Boot 4.0 handed several Spring Session store integrations to the teams closest to the technology. MongoDB and Hazelcast now maintain their own, keeping Boot lean and letting each integration track its store's release cycle.

## The Fix {.diffs}

```diff-card
# // pom.xml — switch to Spring Data MongoDB's own session support
@@removed
<!-- Boot 3.5: built-in auto-config via spring-session-data-mongodb -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-mongodb</artifactId>
</dependency>
@@added
<!-- Boot 4.0: session support is part of spring-boot-starter-data-mongodb -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
<!-- Refer to Spring Data MongoDB docs for explicit session config -->
```

## How To Fix {.fixes}

#### Migrate to the Spring Data MongoDB session integration.

The MongoDB team's Spring Session support is bundled with Spring Data MongoDB. Add <code>spring-boot-starter-data-mongodb</code> and configure the session repository according to Spring Data MongoDB's documentation. The <code>spring.session.store-type=mongodb</code> property alone is no longer sufficient.

## Scope Check {.scope-check}

Check <code>spring.session.store-type=mongodb</code> in your properties. Grep for <code>MongoIndexedSessionRepository</code> and <code>MongoSessionConfiguration</code> in your source.

## Watch Out {.watch-out}

- MongoDB session property keys changed: <code>spring.session.mongodb.*</code> became <code>spring.session.data.mongodb.*</code>. This is a separate but related break covered in the session property renames card.

## Verify {.verify}

MongoDB-backed session store initialises correctly after migrating to the MongoDB-team-maintained integration

## Further Info {.further-info}

In Boot 3.5, <code>spring-session-data-mongodb</code> on the classpath plus <code>spring.session.store-type=mongodb</code> was enough: Boot wired the session repository for you. That wiring is what disappeared; the integration now ships as part of Spring Data MongoDB.

## Links {.footer-links}

- [spring-break module: spring-session-mongodb-removed](https://github.com/spoole167/spring-break/tree/main/spring-session-mongodb-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

