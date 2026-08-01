---
id: jackson-property-inclusion
tier: 3
tier_label: Wrong Results
title: spring.jackson.default-property-inclusion Silently Ignored
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: jackson
---

The <code>spring.jackson.default-property-inclusion</code> property no longer configures Jackson 3's ObjectMapper. Null fields reappear in your API responses without warning.

## What You'll See {.error-output}

```error-output
# application.properties (still present, no startup warning)
spring.jackson.default-property-inclusion=non_null

// API response — before (Spring Boot 3.5)
{"name": "Alice", "age": 30}

// API response — after (Spring Boot 4.0)
{"name": "Alice", "age": 30, "middleName": null, "nickname": null}

// Contract test failure
org.opentest4j.AssertionFailedError:
Expected JSON keys: [name, age]
  Actual JSON keys: [name, age, middleName, nickname]
```

## What Changed {.what-changed}

Spring Boot 4.0's auto-configuration for Jackson 3 no longer reads the <code>spring.jackson.default-property-inclusion</code> property. The property sits in your config file doing nothing: no deprecation warning, no startup error. The ObjectMapper reverts to Jackson's default inclusion policy: <code>ALWAYS</code> (include everything, including nulls).

## Why {.why-changed}

Jackson 3 moved inclusion configuration from a single global enum to a granular per-type system. The old Spring Boot property didn't map cleanly to the new API, so the binding was removed pending a redesign.

## The Fix {.diffs}

```diff-card
# // application.properties — old (silently ignored)
@@removed
spring.jackson.default-property-inclusion=non_null
@@added
# Removed — configure via ObjectMapper bean instead
```

```diff-card
# // ObjectMapper customiser bean
@@removed
# (relied on spring.jackson.default-property-inclusion)
@@added
@Bean
public Jackson2ObjectMapperBuilderCustomizer nonNullInclusion() {
    return builder -> builder
        .serializationInclusion(JsonInclude.Include.NON_NULL);
}
```

```diff-card
# // Or per-class annotation
@@removed
public class UserDto {
@@added
@JsonInclude(JsonInclude.Include.NON_NULL)
public class UserDto {
```

## How To Fix {.fixes}

#### Register an ObjectMapper customiser bean (recommended).

Create a <code>Jackson2ObjectMapperBuilderCustomizer</code> bean that calls <code>serializationInclusion(NON_NULL)</code>. This replaces the property-based configuration and works with Jackson 3.

#### Annotate DTOs individually.

Add <code>@JsonInclude(JsonInclude.Include.NON_NULL)</code> to each DTO class. More verbose, but gives you per-class control and makes the contract explicit in the code.

## Scope Check {.scope-check}

Check whether your <code>application.properties</code> or <code>application.yml</code> sets <code>spring.jackson.default-property-inclusion</code>. If it does, that setting is now dead.

## Watch Out {.watch-out}

- The only signal is larger JSON payloads with unexpected null fields. Without contract tests, that is easy to miss.
- If your front-end checks for key presence (e.g. <code>if ("middleName" in user)</code>) instead of checking for null values, the extra null fields change its branching logic.

## Verify {.verify}

JSON output excludes null fields where expected (compare before/after)

## Further Info {.further-info}

Driven by the combination of Jackson 3.0 and Spring Boot 4.0's auto-configuration changes. See also: jackson-dates-timestamps, jackson-date-format.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/jackson-property-inclusion)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [Jackson 3 in Spring (blog)](https://spring.io/blog/2025/10/07/introducing-jackson-3-support-in-spring/)

