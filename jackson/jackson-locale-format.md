---
id: jackson-locale-format
tier: 3
tier_label: Wrong Results
title: Jackson Locale Format Change (BCP 47)
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: jackson
---

Jackson 3.0 serialises <code>Locale</code> using IETF BCP 47 format (zh-CN) instead of Java's underscore format (zh_CN). Locale-matching logic silently breaks.

## What You'll See {.error-output}

```error-output
// API response — before (Spring Boot 3.5)
{"userLocale": "zh_CN"}

// API response — after (Spring Boot 4.0)
{"userLocale": "zh-CN"}

// Downstream lookup fails
Map<String, String> labels = i18nMap.get(user.getLocale());
// returns null — map keys use "zh_CN", response now sends "zh-CN"

// Test failure
org.opentest4j.AssertionFailedError:
Expected: "zh_CN"
  Actual: "zh-CN"
```

## What Changed {.what-changed}

Jackson 3.0 switched <code>Locale</code> serialisation from Java's <code>toString()</code> format (<code>zh_CN</code>) to the IETF BCP 47 language tag format (<code>zh-CN</code>). Underscores become hyphens and the structure follows RFC 5646.

## Why {.why-changed}

BCP 47 is the web standard used by HTTP <code>Accept-Language</code> headers, HTML <code>lang</code> attributes, and JavaScript's <code>Intl</code> API. Aligning Jackson's output with the rest of the web ecosystem removes a constant source of mismatch bugs.

## The Fix {.diffs}

```diff-card
# // Custom serialiser to restore old format
@@removed
// No custom serialiser needed in Boot 3.5
@@added
@JsonSerialize(using = JavaLocaleSerializer.class)
private Locale userLocale;
```

```diff-card
# // Or update all consumers to use BCP 47
@@removed
String localeKey = user.getLocale(); // "zh_CN"
@@added
String localeKey = Locale.forLanguageTag(user.getLocale()).toString(); // normalise
```

```diff-card
# // Fix i18n lookup maps
@@removed
i18nMap.put("zh_CN", chineseLabels);
@@added
i18nMap.put("zh-CN", chineseLabels);
```

## How To Fix {.fixes}

#### Adopt BCP 47 throughout (recommended).

Update your i18n maps, database locale columns, and downstream consumers to use hyphenated BCP 47 tags. This aligns with HTTP headers and browser APIs. Run a data migration for persisted locale strings.

#### Write a custom Locale serialiser.

Register a custom Jackson serialiser that calls <code>locale.toString()</code> instead of <code>locale.toLanguageTag()</code>. This preserves the old underscore format while you plan the broader migration.

## Scope Check {.scope-check}

Search for any DTO or entity field of type <code>java.util.Locale</code> that gets serialised to JSON. Also check database columns, message queues, and i18n resource bundle keys that use the underscore format. Every one of those is a potential mismatch.

## Watch Out {.watch-out}

- If you store locale strings in a database and compare them with equality checks, the format flip causes mismatches that surface as missing data, not errors. A <code>SELECT WHERE locale = 'zh_CN'</code> query won't match the new <code>zh-CN</code> value sent by the API.
- BCP 47 has subtag rules beyond swapping underscores for hyphens. Some locales have three components (e.g. <code>zh_Hant_TW</code> becomes <code>zh-Hant-TW</code>). Use <code>Locale.forLanguageTag()</code> for correct round-tripping.

## Verify {.verify}

Locale-sensitive fields render correctly in all target locales

## Further Info {.further-info}

Driven by Jackson 3.0, upstream of Spring Boot 4.0. See also: jackson-date-format, jackson-group-id.

## Links {.footer-links}

- [spring-break module: jackson-locale-format](https://github.com/spoole167/spring-break/tree/main/jackson-locale-format)

- [Jackson 3 migration guide](https://github.com/FasterXML/jackson/blob/main/jackson3/MIGRATING_TO_JACKSON_3.md)

- [IETF BCP 47 (RFC 5646)](https://www.rfc-editor.org/rfc/rfc5646)

