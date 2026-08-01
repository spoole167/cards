---
id: hibernate-native-datetime
tier: 3
tier_label: Wrong Results
title: Hibernate Native Query Date Return Types Changed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: hibernate
---

Native SQL queries now return <code>java.time.LocalDate</code> instead of <code>java.sql.Date</code>. Code casting to the old types throws ClassCastException or silently loses time components.

## What You'll See {.error-output}

```error-output
// Native query — same code, different result type
Object result = em.createNativeQuery("SELECT created_date FROM orders WHERE id = 1")
                  .getSingleResult();

// Before (Spring Boot 3.5)
result.getClass() → java.sql.Date

// After (Spring Boot 4.0)
result.getClass() → java.time.LocalDate

// ClassCastException at runtime
java.lang.ClassCastException:
  java.time.LocalDate cannot be cast to java.sql.Date

// Or subtler: time component silently lost
java.sql.Timestamp ts = (java.sql.Timestamp) result;  // was Timestamp
// Now: java.time.LocalDateTime — no getTime() method
```

## What Changed {.what-changed}

Hibernate 7 (used by Spring Boot 4.0) changed the default Java types returned for SQL <code>DATE</code>, <code>TIME</code>, and <code>TIMESTAMP</code> columns in native queries. <code>java.sql.Date</code> became <code>java.time.LocalDate</code>, <code>java.sql.Time</code> became <code>java.time.LocalTime</code>, and <code>java.sql.Timestamp</code> became <code>java.time.LocalDateTime</code>.

## Why {.why-changed}

The <code>java.sql</code> date/time classes are legacy wrappers around <code>java.util.Date</code> with known timezone and truncation bugs. Hibernate aligned its defaults with <code>java.time</code>: immutable, thread-safe, the modern standard.

## The Fix {.diffs}

```diff-card
# // Fix native query result handling
@@removed
java.sql.Date date = (java.sql.Date) row[2];
long millis = date.getTime();
@@added
LocalDate date = (LocalDate) row[2];
long millis = date.atStartOfDay(ZoneOffset.UTC)
                  .toInstant().toEpochMilli();
```

```diff-card
# // Typed native query
@@removed
@Query(value = "SELECT created_date FROM orders", nativeQuery = true)
List<java.sql.Date> findAllDates();
@@added
@Query(value = "SELECT created_date FROM orders", nativeQuery = true)
List<java.time.LocalDate> findAllDates();
```

```diff-card
# // Timestamp results
@@removed
java.sql.Timestamp ts = (java.sql.Timestamp) row[3];
@@added
LocalDateTime ts = (LocalDateTime) row[3];
```

## How To Fix {.fixes}

#### Update result type casts to java.time (recommended).

Replace all <code>java.sql.Date</code> casts with <code>java.time.LocalDate</code>, <code>java.sql.Time</code> with <code>java.time.LocalTime</code>, and <code>java.sql.Timestamp</code> with <code>java.time.LocalDateTime</code>.

#### Force old types with explicit addScalar().

For legacy code that can't be updated, use Hibernate's <code>addScalar("col", StandardBasicTypes.DATE)</code> to force the old <code>java.sql</code> return types on specific queries.

## Scope Check {.scope-check}

Search for <code>nativeQuery = true</code>, <code>createNativeQuery</code>, and any cast to <code>java.sql.Date</code>, <code>java.sql.Time</code>, or <code>java.sql.Timestamp</code> in query result handling code.

## Watch Out {.watch-out}

- This only affects native queries that return raw <code>Object[]</code> rows. JPQL queries with entity mappings or <code>@ColumnResult</code> type mappings are unaffected because the type is explicit.
- <code>java.time.LocalDate</code> has no time component. If your code relied on <code>java.sql.Date.getTime()</code> returning midnight epoch millis, the replacement requires an explicit timezone conversion.

## Verify {.verify}

SELECT queries return correct date/time types (check ResultSet)

## Further Info {.further-info}

Driven by Hibernate 7.0, upstream of Spring Boot 4.0, and documented in its migration guide. See also: hibernate-dialect-removal.

## Links {.footer-links}

- [spring-break module: hibernate-native-datetime](https://github.com/spoole167/spring-break/tree/main/hibernate-native-datetime)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

