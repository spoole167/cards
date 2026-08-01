---
id: jackson-group-id
tier: 1
tier_label: Won't Build
title: Jackson 3.0 Group ID Change
series: spring-boot 3.5 → 4.0
effort: L
openrewrite: org.openrewrite.java.jackson.UpgradeJackson_2_3
subsystem: jackson
max_pages: 2
---

Jackson 3.0 moved from com.fasterxml.jackson to tools.jackson. Every Maven coordinate and every import path is now wrong.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/JacksonConfig.java:[3,32]
  package com.fasterxml.jackson.databind does not exist
[ERROR] /src/main/java/com/example/JacksonConfig.java:[8,5]
  cannot find symbol
    symbol: class ObjectMapper
```

## What Changed {.what-changed}

Jackson 3.0 moved from the <code>com.fasterxml.jackson</code> Maven group to <code>tools.jackson</code>. Spring Boot 4.0's managed dependencies point to Jackson 3.0, so the old group ID no longer resolves. Every import statement using <code>com.fasterxml.jackson</code> breaks.

## Why {.why-changed}

Jackson's maintainers separated the project identity from the original FasterXML organisation, giving a clean break to drop deprecated APIs and ship Jackson 3.0 as a proper major release.

## The Fix {.diffs}

#### Removed
```diff-card
# // pom.xml — explicit Jackson dependency
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

#### Added
```diff-card
<dependency>
    <groupId>tools.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

#### Removed
```diff-card
# // Java imports — ObjectMapper and core types
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.core.JsonProcessingException;
```

#### Added
```diff-card
import tools.jackson.databind.ObjectMapper;
import tools.jackson.core.JacksonException;
```

## How To Fix {.fixes}

#### OpenRewrite (recommended).

Run the <a href="https://docs.openrewrite.org/recipes/java/jackson/upgradejackson_2_3_typechanges">Jackson 3 migration recipe</a>. Handles group ID changes and import updates across the entire codebase in one pass.

#### Manual: update Maven coordinates first.

Replace <code>com.fasterxml.jackson</code> with <code>tools.jackson</code> in all POM or Gradle files, then fix imports file by file.

## Scope Check {.scope-check}

Search your codebase for <code>com.fasterxml.jackson</code>. Every hit is a coordinate or import that needs updating. If you use <code>spring-boot-starter-json</code> or <code>spring-boot-starter-web</code>, Jackson arrives transitively. You still need to update explicit dependency declarations and all source-level imports.

## Watch Out {.watch-out}

- The class renames (JsonSerializer → ValueSerializer, SerializerProvider → SerializationContext) arrive in the same Jackson 3.0 upgrade. Fixing only the group ID leaves broken class references. See jackson-class-renames for the full list.
- Jackson exceptions (<code>JsonMappingException</code>, <code>JsonProcessingException</code>) are <code>Serializable</code>. If you've persisted these anywhere (dead letter queues, error logs), those serialised streams contain the old class names and won't deserialise correctly after migration.

## Further Info {.further-info}

Driven by Jackson 3.0, upstream of Spring Boot 4.0. The FasterXML to tools.jackson rename was announced in 2023 and finalised in Jackson 3.0-rc1. See also: jackson-class-renames, jackson-exception-hierarchy.

#### From the field.

Every production migration writeup so far names Jackson 3 as the single biggest risk in the whole 4.0 upgrade. The import change is the easy half; the behaviour shifts covered in the sibling cards are where the time goes. Teams that budgeted a day for "update the imports" report spending weeks on serialisation diffs.

## Links {.footer-links}

- [spring-break module: jackson-group-id](https://github.com/spoole167/spring-break/tree/main/jackson-group-id)

- [Jackson 3 migration guide](https://github.com/FasterXML/jackson/blob/main/jackson3/MIGRATING_TO_JACKSON_3.md)

- [Spring Boot 4.0 migration guide for production teams (DEV)](https://dev.to/aytronn/spring-boot-40-migration-guide-for-production-teams-what-actually-breaks-and-how-to-upgrade-safely-22me)

- [Spring Boot 4 migration: what actually broke (Java Code Geeks)](https://www.javacodegeeks.com/2026/05/spring-boot-4-migration-breaking-changes-new-defaultsand-what-actually-broke.html)
