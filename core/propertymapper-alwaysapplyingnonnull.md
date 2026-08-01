---
id: propertymapper-alwaysapplyingnonnull
tier: 1
tier_label: Won't Build
title: PropertyMapper.alwaysApplyingWhenNonNull() Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
---

PropertyMapper.alwaysApplyingWhenNonNull() is removed because skipping null values is now the default. Just delete the call.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/PropertyMapperUsage.java:[8,47]
  error: cannot find symbol
    symbol:   method alwaysApplyingWhenNonNull()
    location: class PropertyMapper
```

## What Changed {.what-changed}

<code>PropertyMapper.alwaysApplyingWhenNonNull()</code> is gone. Its behaviour, skipping mappings when the source value is <code>null</code>, is now the default for <code>PropertyMapper</code>, so calling it was a no-op from 4.0 onward.

## Why {.why-changed}

The non-null default was the right behaviour for virtually all use cases. Requiring an explicit opt-in call was boilerplate with no practical reason to exist. Making it the default and removing the method simplifies the API.

## The Fix {.diffs}

```diff-card
# // Remove the method call — the behaviour is now default
@@removed
PropertyMapper map = PropertyMapper.get().alwaysApplyingWhenNonNull();
@@added
PropertyMapper map = PropertyMapper.get();
```

## How To Fix {.fixes}

#### Delete the alwaysApplyingWhenNonNull() call.

Remove <code>.alwaysApplyingWhenNonNull()</code> from the chain. The mapper behaves identically on Boot 4.0 without it. If you explicitly want to map null values, use <code>.always()</code> on individual mappings.

## Scope Check {.scope-check}

Search for <code>alwaysApplyingWhenNonNull</code> across all Java/Kotlin sources. This typically appears in autoconfiguration classes or custom configuration adapters that mirror Spring Boot's internal style.

## Watch Out {.watch-out}

- If you were relying on the absence of <code>alwaysApplyingWhenNonNull()</code> to intentionally map null values through, that behaviour has also changed: nulls are now skipped by default. Add <code>.always()</code> on the specific mappings that need to pass nulls.

## Further Info {.further-info}

PropertyMapper is an internal Spring Boot utility, used mostly in auto-configuration classes to map configuration properties onto builder objects.

## Links {.footer-links}

- [spring-break module: propertymapper-alwaysapplyingnonnull](https://github.com/spoole167/spring-break/tree/main/propertymapper-alwaysapplyingnonnull)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

