---
id: spring-session-hazelcast-removed
tier: 2
tier_label: Won't Run
title: Spring Session Hazelcast Auto-Configuration Removed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: data-messaging
---

Spring Boot 4.0 dropped built-in Hazelcast session auto-configuration. The app compiles but fails at startup with <code>NoSuchBeanDefinitionException</code>.

## What You'll See {.error-output}

```error-output
// Boot 4.0 startup failure:
org.springframework.beans.factory.NoSuchBeanDefinitionException:
  No qualifying bean of type
  'org.springframework.session.SessionRepository' available

// Or context load failure mentioning missing HazelcastIndexedSessionRepository
```

## What Changed {.what-changed}

Spring Boot's auto-configuration for Hazelcast-backed Spring Session was removed from <code>spring-boot-autoconfigure</code>. The classes <code>HazelcastSessionConfiguration</code> and <code>HazelcastSessionProperties</code> no longer exist in the Spring Boot artifact. The Hazelcast team took ownership of the integration.

## Why {.why-changed}

Boot 4.0 handed several Spring Session store integrations to the teams closest to the technology. Hazelcast and MongoDB now maintain their own, keeping Boot lean and letting each integration track its store's release cycle.

## The Fix {.diffs}

```diff-card
# // pom.xml — add Hazelcast team's Spring Session integration
@@removed
<!-- Boot 3.5: built-in, no extra dependency needed beyond spring-session-hazelcast -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-hazelcast</artifactId>
</dependency>
@@added
<!-- Boot 4.0: use Hazelcast team's own integration artifact -->
<dependency>
    <groupId>com.hazelcast</groupId>
    <artifactId>hazelcast-spring-session</artifactId>
    <version><!-- check Hazelcast release --></version>
</dependency>
```

## How To Fix {.fixes}

#### Migrate to the Hazelcast-maintained Spring Session integration.

Replace <code>spring-session-hazelcast</code> with the artifact published by the Hazelcast team and configure it from Hazelcast's own documentation.

## Scope Check {.scope-check}

Check <code>spring.session.store-type=hazelcast</code> in your properties. Grep for <code>HazelcastIndexedSessionRepository</code> and <code>HazelcastSessionConfiguration</code> in your source.

## Watch Out {.watch-out}

- Property keys and bean names may differ in the Hazelcast-maintained integration. Verify them against Hazelcast's own documentation rather than the Spring Boot migration guide.

## Verify {.verify}

Hazelcast-backed session store initialises correctly after migrating to the Hazelcast-team-maintained integration

## Further Info {.further-info}

In Boot 3.5, <code>spring-session-hazelcast</code> on the classpath plus <code>spring.session.store-type=hazelcast</code> was enough: Boot wired the session store for you. That wiring is what disappeared.

## Links {.footer-links}

- [spring-break module: spring-session-hazelcast-removed](https://github.com/spoole167/spring-break/tree/main/spring-session-hazelcast-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [Hazelcast Spring Session Documentation](https://docs.hazelcast.com/hazelcast/latest/spring/spring-session)

