---
id: spring-session-property-renames
tier: 3
tier_label: Wrong Results
title: Spring Session Property Prefixes Renamed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: data-messaging
---

<code>spring.session.redis.*</code> and <code>spring.session.mongodb.*</code> are silently ignored in Boot 4.0. The new prefixes are <code>spring.session.data.redis.*</code> and <code>spring.session.data.mongodb.*</code>.

## What You'll See {.error-output}

```error-output
# application.properties (Boot 3.5 style, ignored on 4.0):
spring.session.redis.namespace=myapp
spring.session.redis.flush-mode=on-save
spring.session.mongodb.collection-name=sessions

# Boot 4.0: app starts, sessions stored under default namespace.
# No error. Session isolation between apps sharing the same Redis
# instance is broken.
```

## What Changed {.what-changed}

Spring Session's Redis backend properties moved from <code>spring.session.redis.*</code> to <code>spring.session.data.redis.*</code>. MongoDB backend properties moved from <code>spring.session.mongodb.*</code> to <code>spring.session.data.mongodb.*</code>. This mirrors the MongoDB connection property rename pattern (<code>spring.data.mongodb</code> → <code>spring.mongodb</code>).

## Why {.why-changed}

The <code>data</code> infix ties each session store's configuration to its Spring Data module namespace, making the relationship explicit.

## The Fix {.diffs}

```diff-card
# // Redis session properties
@@removed
spring.session.redis.namespace=myapp:session
spring.session.redis.flush-mode=on-save
spring.session.redis.save-mode=on-set-attribute
spring.session.redis.cleanup-cron=0 * * * * *
@@added
spring.session.data.redis.namespace=myapp:session
spring.session.data.redis.flush-mode=on-save
spring.session.data.redis.save-mode=on-set-attribute
spring.session.data.redis.cleanup-cron=0 * * * * *
```

```diff-card
# // MongoDB session properties
@@removed
spring.session.mongodb.collection-name=sessions
@@added
spring.session.data.mongodb.collection-name=sessions
```

## How To Fix {.fixes}

#### Rename spring.session.redis.* to spring.session.data.redis.*

Do a project-wide search-and-replace: <code>spring.session.redis.</code> → <code>spring.session.data.redis.</code> and <code>spring.session.mongodb.</code> → <code>spring.session.data.mongodb.</code>. Check all property files, YAML files, and profile variants.

## Scope Check {.scope-check}

Grep for <code>spring.session.redis</code> and <code>spring.session.mongodb</code> across all property and YAML files.

## Watch Out {.watch-out}

- The session store type property (<code>spring.session.store-type</code>) is unchanged. Only the store-specific configuration sub-keys are affected.
- Nothing fails at startup, which makes this hard to catch in testing. A wrong namespace in Redis means sessions are stored but never found: users appear logged out immediately after login. Confirm the fix by checking session keys exist in Redis under the expected namespace.

## Verify {.verify}

Session store connects correctly with the renamed property prefixes: confirm sessions persist to Redis or MongoDB with the new configuration

## Further Info {.further-info}

Old-prefix properties still parse, so the app starts cleanly. Sessions fall back to defaults (wrong namespace, wrong collection, wrong TTL) or the auto-configuration picks the first available store type.

## Links {.footer-links}

- [spring-break module: spring-session-property-renames](https://github.com/spoole167/spring-break/tree/main/spring-session-property-renames)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

