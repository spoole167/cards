---
id: mongodb-property-renames
tier: 3
tier_label: Wrong Results
title: MongoDB Configuration Properties Renamed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: true
subsystem: data-messaging
---

<code>spring.data.mongodb.*</code> properties are silently ignored in Boot 4.0. The new prefix is <code>spring.mongodb.*</code>. The app starts but connects with defaults: wrong host, no auth, wrong database.

## What You'll See {.error-output}

```error-output
# application.properties (Boot 3.5 style, ignored on 4.0):
spring.data.mongodb.uri=mongodb://user:pass@prod-host:27017/mydb
spring.data.mongodb.database=mydb

# Boot 4.0: app starts, connects to localhost:27017 with no auth.
# No error: wrong database, wrong host. Data operations succeed
# against the wrong instance.
```

## What Changed {.what-changed}

All MongoDB connection and configuration properties moved from the <code>spring.data.mongodb</code> namespace to <code>spring.mongodb</code>. Examples: <code>spring.data.mongodb.uri</code> → <code>spring.mongodb.uri</code>, <code>spring.data.mongodb.database</code> → <code>spring.mongodb.database</code>. Actuator management properties changed from <code>management.health.mongo.*</code> to <code>management.health.mongodb.*</code>.

## Why {.why-changed}

MongoDB auto-configuration moved into a dedicated persistence module, and the old prefix was a legacy of living inside Spring Data's namespace. The rename matches Boot's own namespace conventions.

## The Fix {.diffs}

```diff-card
# // application.properties
@@removed
spring.data.mongodb.uri=mongodb://user:pass@host:27017/mydb
spring.data.mongodb.database=mydb
spring.data.mongodb.auto-index-creation=true
@@added
spring.mongodb.uri=mongodb://user:pass@host:27017/mydb
spring.mongodb.database=mydb
spring.mongodb.auto-index-creation=true
```

```diff-card
# // Actuator health properties
@@removed
management.health.mongo.enabled=false
@@added
management.health.mongodb.enabled=false
```

## How To Fix {.fixes}

#### Rename all spring.data.mongodb.* properties to spring.mongodb.*

Do a project-wide search-and-replace: <code>spring.data.mongodb.</code> → <code>spring.mongodb.</code>. Check all <code>application.properties</code>, <code>application.yml</code>, and any profile-specific variants. OpenRewrite has a migration recipe that handles this automatically.

## Scope Check {.scope-check}

Grep for <code>spring.data.mongodb</code> across all property files, YAML files, and <code>@Value</code> annotations. Also check <code>management.health.mongo</code>.

## Watch Out {.watch-out}

- The app starts, connects with driver defaults, and runs normally against the wrong instance. Confirm the property is being read by testing with a URI that cannot resolve, such as a non-existent host.
- Spring Session MongoDB property keys changed separately: <code>spring.session.mongodb.*</code> → <code>spring.session.data.mongodb.*</code>. Check the spring-session-property-renames card.

## Verify {.verify}

Application connects to MongoDB and all configuration (URI, auth, pool size, etc.) applies correctly after renaming properties

## Further Info {.further-info}

Every property under the old prefix moved, pool size and auth settings included.

## Links {.footer-links}

- [spring-break module: mongodb-property-renames](https://github.com/spoole167/spring-break/tree/main/mongodb-property-renames)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

