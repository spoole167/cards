---
id: jackson-component-rename
tier: 1
tier_label: Won't Build
title: '@JsonComponent and @JsonMixin Renamed to @JacksonComponent and @JacksonMixin'
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: |-
  false
  The root cause, the Jackson 3 package migration, is covered by the jackson-group-id module. Developers migrating to Boot 4.0 hit the package change first; the annotation rename is a secondary step once that migration is complete.
subsystem: jackson
---

<code>@JsonComponent</code> and <code>@JsonMixin</code> are removed in Boot 4.0 and fail to compile. Replace with <code>@JacksonComponent</code> and <code>@JacksonMixin</code>.

## What You'll See {.error-output}

```error-output
@JsonComponent
public class MoneySerializer extends JsonSerializer<Money> { ... }

// Boot 4.0 compile error:
error: cannot find symbol
  symbol: @JsonComponent
  location: class MoneySerializer
```

## What Changed {.what-changed}

Spring Boot's custom serialiser registration annotations were renamed to align with Jackson 3's branding. <code>@JsonComponent</code> becomes <code>@JacksonComponent</code> and <code>@JsonMixin</code> becomes <code>@JacksonMixin</code>. The functionality is identical: only the annotation names changed.

## Why {.why-changed}

Jackson 3 rebranded from the <code>com.fasterxml.jackson</code> group ID to <code>tools.jackson</code>. Spring Boot's annotation names were updated to match, making it clear they belong to the Jackson 3 ecosystem rather than the legacy Jackson 2 stack.

## The Fix {.diffs}

```diff-card
# // Custom serializer
@@removed
import org.springframework.boot.jackson.JsonComponent;

@JsonComponent
public class MoneySerializer extends JsonSerializer<Money> { ... }
@@added
import org.springframework.boot.jackson.JacksonComponent;

@JacksonComponent
public class MoneySerializer extends JsonSerializer<Money> { ... }
```

```diff-card
# // Mixin
@@removed
import org.springframework.boot.jackson.JsonMixin;

@JsonMixin(Money.class)
public abstract class MoneyMixin { ... }
@@added
import org.springframework.boot.jackson.JacksonMixin;

@JacksonMixin(Money.class)
public abstract class MoneyMixin { ... }
```

## How To Fix {.fixes}

#### Rename the annotations.

Replace all <code>@JsonComponent</code> with <code>@JacksonComponent</code> and all <code>@JsonMixin</code> with <code>@JacksonMixin</code>. Update the imports accordingly.

## Scope Check {.scope-check}

Grep for <code>@JsonComponent</code> and <code>@JsonMixin</code> across your source tree. Also search for <code>import org.springframework.boot.jackson.JsonComponent</code> and <code>import org.springframework.boot.jackson.JsonMixin</code>.

## Watch Out {.watch-out}

- Do not confuse Spring Boot's <code>@JacksonComponent</code> with Jackson's own <code>@JsonSerialize</code> / <code>@JsonDeserialize</code> annotations. Spring Boot's annotation auto-registers the serialiser with the <code>ObjectMapper</code> bean; Jackson's own annotations require explicit wiring.

## Verify {.verify}

Custom serialisers and mixins annotated with the new names register correctly with the auto-configured ObjectMapper

## Further Info {.further-info}

@JsonComponent and @JsonMixin were Spring Boot-specific annotations that registered custom serialisers, deserialisers, and mixins with the auto-configured ObjectMapper.

## Links {.footer-links}

- [spring-break module: jackson-component-rename](https://github.com/spoole167/spring-break/tree/main/jackson-component-rename)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

