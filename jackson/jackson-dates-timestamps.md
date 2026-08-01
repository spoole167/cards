---
id: jackson-dates-timestamps
tier: 2
tier_label: Won't Run
title: write-dates-as-timestamps Property Breaks Startup
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: jackson
---

The <code>spring.jackson.serialization.write-dates-as-timestamps</code> property no longer binds in Boot 4.0: Jackson 3 moved the feature out of <code>SerializationFeature</code>. One leftover config line stops your application from starting.

## What You'll See {.error-output}

```error-output
# application.properties (worked on Boot 3.5)
spring.jackson.serialization.write-dates-as-timestamps=true

// Spring Boot 4.0 — application fails to start:
org.springframework.boot.context.properties.bind.BindException:
  Failed to bind properties under 'spring.jackson.serialization' to
  java.util.Map<tools.jackson.databind.SerializationFeature, Boolean>
Caused by: java.lang.IllegalArgumentException: No enum constant
  tools.jackson.databind.SerializationFeature.write-dates-as-timestamps
```

## What Changed {.what-changed}

Jackson 3 reorganised its feature flags: date/time toggles such as WRITE_DATES_AS_TIMESTAMPS are no longer constants on <code>SerializationFeature</code>. Boot 4.0 still binds <code>spring.jackson.serialization.*</code> keys against that enum, so the old property value fails enum conversion and property binding aborts context startup.

## Why {.why-changed}

The old <code>SerializationFeature</code> enum values don't map one-to-one to Jackson 3's new configuration API; the date/time features moved to their own feature set. Spring Boot's property binding follows the new enum, so values naming removed constants fail fast instead of being remapped.

## The Fix {.diffs}

```diff-card
# // application.properties — old (silently ignored)
@@removed
spring.jackson.serialization.write-dates-as-timestamps=true
@@added
# Removed — configure via ObjectMapper customiser instead
```

```diff-card
# // ObjectMapper customiser replacement
@@removed
# (relied on spring.jackson.serialization.write-dates-as-timestamps)
@@added
@Bean
public Jackson2ObjectMapperBuilderCustomizer timestampDates() {
    return builder -> builder.featuresToEnable(
        SerializationFeature.WRITE_DATES_AS_TIMESTAMPS
    );
}
```

```diff-card
# // Alternative: per-field annotation
@@removed
private Instant timestamp;
@@added
@JsonFormat(shape = JsonFormat.Shape.NUMBER)
private Instant timestamp;
```

## How To Fix {.fixes}

#### Register an ObjectMapper customiser bean.

Create a <code>Jackson2ObjectMapperBuilderCustomizer</code> that explicitly enables <code>WRITE_DATES_AS_TIMESTAMPS</code>. This replaces the dead properties-based config.

#### Migrate clients to ISO-8601 (recommended).

If you can coordinate with API consumers, drop the timestamp format entirely and adopt ISO-8601 strings. Remove the old property and let Jackson 3's new default take effect.

## Scope Check {.scope-check}

Search your <code>application.properties</code> and <code>application.yml</code> files for any <code>spring.jackson.serialization</code> or <code>spring.jackson.deserialization</code> entries. Every one of those feature toggles is potentially dead in Boot 4.0.

## Watch Out {.watch-out}

- This compounds with the jackson-date-format change. If you set <code>write-dates-as-timestamps=true</code> specifically to keep numeric timestamps, you now have two problems: the default flipped, and the property that pinned the old behaviour stops your app from starting.
- Any <code>spring.jackson.serialization.*</code> or <code>spring.jackson.deserialization.*</code> value naming an enum constant that Jackson 3 moved or removed fails startup the same way.

## Verify {.verify}

Application fails to start: BindException on spring.jackson.serialization, "No enum constant tools.jackson.databind.SerializationFeature.write-dates-as-timestamps"

## Further Info {.further-info}

Driven by the combination of Jackson 3.0 and Spring Boot 4.0's auto-configuration changes. Verified 15 July 2026 on 4.0.7. Compounds with jackson-date-format. See also: jackson-date-format, jackson-property-inclusion.

## Links {.footer-links}

- [spring-break module: jackson-dates-timestamps](https://github.com/spoole167/spring-break/tree/main/jackson-dates-timestamps)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [Jackson 3 in Spring (blog)](https://spring.io/blog/2025/10/07/introducing-jackson-3-support-in-spring/)

