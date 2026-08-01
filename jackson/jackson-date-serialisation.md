---
id: jackson-date-serialisation
tier: 3
tier_label: Wrong Results
title: Jackson Date Serialisation Flip
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: none
subsystem: jackson
---

Jackson 3.0 defaults to ISO-8601 date strings instead of numeric timestamps. Every API response containing a date silently changes shape.

## What You'll See {.error-output}

```error-output
// API response — before (Spring Boot 3.5)
{"created": 1714089600000}

// API response — after (Spring Boot 4.0)
{"created": "2025-04-26T00:00:00Z"}

// Front-end JavaScript blows up
TypeError: date.getTime is not a function
// or: contract test fails
Expected: <1714089600000> (Long)
  Actual: <"2025-04-26T00:00:00Z"> (String)
```

## What Changed {.what-changed}

Jackson 3.0 changed the default value of <code>SerializationFeature.WRITE_DATES_AS_TIMESTAMPS</code> from <code>true</code> to <code>false</code>. Dates, times, and durations now serialise as ISO-8601 strings by default instead of numeric epoch milliseconds.

## Why {.why-changed}

ISO-8601 strings are human-readable, timezone-aware, and the de facto standard in JSON APIs. The old numeric default caused constant confusion about whether the value was seconds or milliseconds, and lost timezone information entirely.

## The Fix {.diffs}

#### Removed
```diff-card
# // application.properties — restore old behaviour
# (no explicit setting needed in Boot 3.5)
```

#### Added
```diff-card
spring.jackson.serialization.write-dates-as-timestamps=true
```

#### Removed
```diff-card
# // or adopt ISO-8601 and fix the client
const ts = new Date(response.created);  // was a number
```

#### Added
```diff-card
const ts = new Date(response.created);  // now parses ISO string
```

#### Removed
```diff-card
# // ObjectMapper explicit configuration
// Jackson 2 default: timestamps enabled
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());
```

#### Added
```diff-card
// Jackson 3 default: ISO strings
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());
// only if you need the old numeric format:
mapper.enable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
```

## How To Fix {.fixes}

#### Opt in to the new ISO-8601 default (recommended).

Update your API clients and contract tests to expect ISO-8601 strings. This is the better long-term format. Coordinate the change with front-end teams so they parse the string, not a number.

#### Restore numeric timestamps.

Add <code>spring.jackson.serialization.write-dates-as-timestamps=true</code> to <code>application.properties</code>. This gives you the old behaviour while you plan the migration.

## Scope Check {.scope-check}

Search for any DTO field of type <code>java.util.Date</code>, <code>Instant</code>, <code>LocalDate</code>, <code>LocalDateTime</code>, <code>ZonedDateTime</code>, or <code>Duration</code>. Every one of those fields will serialise differently after the upgrade. Check API consumers, contract tests, and front-end parsing code.

## Watch Out {.watch-out}

- This change is invisible at compile time and at startup. The app runs perfectly until a client tries to parse the date field as a number. Without contract tests, you won't catch this before production.
- The <code>@JsonFormat</code> annotation on individual fields still overrides the global default. Annotated dates are unaffected; the unannotated ones flip.

## Verify {.verify}

JSON responses show dates in expected format (check API output)

## Further Info {.further-info}

Driven by Jackson 3.0, upstream of Spring Boot 4.0, and announced in its release notes. See also: jackson-dates-timestamps, jackson-property-inclusion.

#### From the field.

The recurring report: integration tests that compare JSON character-by-character fail after the upgrade with zero logic changes. Boot ships <code>spring.jackson.use-jackson2-defaults=true</code> as an escape hatch that pins the old behaviour globally while you migrate consumers.

## Links {.footer-links}

- [spring-break module: jackson-date-serialisation](https://github.com/spoole167/spring-break/tree/main/jackson-date-serialisation)

- [Jackson 3 migration guide](https://github.com/FasterXML/jackson/blob/main/jackson3/MIGRATING_TO_JACKSON_3.md)

- [Jackson 3 in Spring (blog)](https://spring.io/blog/2025/10/07/introducing-jackson-3-support-in-spring/)

- [Spring Boot 4 migration: what actually broke (Java Code Geeks)](https://www.javacodegeeks.com/2026/05/spring-boot-4-migration-breaking-changes-new-defaultsand-what-actually-broke.html)
