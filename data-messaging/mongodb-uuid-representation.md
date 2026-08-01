---
id: mongodb-uuid-representation
tier: 3
tier_label: Wrong Results
title: MongoDB UUID and BigDecimal Representations No Longer Defaulted
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: data-messaging
no_module: true
no_module_reason: |
  Confirmed against the Spring Boot 4.0 migration guide and spring-projects/ spring-boot#33532: Spring Data MongoDB no longer defaults UUID/BigDecimal representations, and spring.mongodb.representation.uuid / spring.data.mongodb.representation.big-decimal must be set explicitly. No spring-break module tests it: demonstrating the failure needs a running MongoDB instance holding documents written under the old defaults, then reading them back under the new ones. Not achievable in an isolated Maven test.
---

Boot 4.0 removes the default UUID and BigDecimal BSON representations. Documents written under Boot 3.5 defaults may deserialise wrongly or fail. Configure the representations explicitly.

## What You'll See {.error-output}

```error-output
// Boot 3.5: UUID stored as STANDARD binary subtype 4
// Boot 4.0 default: UUID stored as JAVA_LEGACY binary subtype 3

// Reading a Boot 3.5 document in a Boot 4.0 app:
org.bson.codecs.configuration.CodecConfigurationException:
  No codec found for UUID with representation JAVA_LEGACY

// Or: UUID field deserialises to null or a wrong value
```

## What Changed {.what-changed}

The <code>spring.mongodb.representation.uuid</code> and <code>spring.data.mongodb.representation.big-decimal</code> properties must now be configured explicitly. Spring Boot 4.0 no longer injects defaults for these representations. For compatibility with existing data, declare the same values Boot 3.5 set automatically.

## Why {.why-changed}

Boot's defaults overrode the MongoDB driver defaults, surprising teams using MongoDB outside Spring Boot or migrating data between tools. Explicit configuration puts the application in control of the data format.

## The Fix {.diffs}

```diff-card
# // application.properties — restore Boot 3.5 representations
@@removed
# Boot 3.5: these were set automatically (no configuration needed)
@@added
# Boot 4.0: must be explicit to match what Boot 3.5 used
spring.mongodb.representation.uuid=standard
# For BigDecimal — check what representation your Boot 3.5 data was written with:
# spring.data.mongodb.representation.big-decimal=decimal128
```

## How To Fix {.fixes}

#### Declare the representations that match your existing data.

Set <code>spring.mongodb.representation.uuid</code> to <code>standard</code> if your documents were written with Boot 3.5 defaults. For BigDecimal, check which BSON type your existing documents use (inspect with mongosh) and configure <code>spring.data.mongodb.representation.big-decimal</code> accordingly.

#### Migrate existing data if changing representations.

To adopt different representations going forward, write a migration script to convert existing documents first. Changing representation without migrating data makes old documents unreadable.

## Scope Check {.scope-check}

Grep for <code>UUID</code> fields in MongoDB document classes (<code>@Document</code>). Also find any <code>BigDecimal</code> or <code>BigInteger</code> fields. These are at risk. Run a read test against your production data snapshot before deploying.

## Watch Out {.watch-out}

- Wrong-representation reads return valid-looking, wrong UUIDs. No exception, no log line.
- Test against a real data snapshot. Unit tests create fresh documents and never catch representation mismatches.

## Verify {.verify}

UUID and BigDecimal fields round-trip correctly and existing documents with the old representation are still readable after explicit configuration

## Further Info {.further-info}

Boot 3.5 set UUID representation to <code>STANDARD</code> and BigDecimal to a specific codec. With no Boot default, the representation falls back to whatever the MongoDB driver or Spring Data MongoDB defaults to. New documents may use a different binary subtype, breaking code that inspects raw BSON.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [MongoDB UUID Representations](https://www.mongodb.com/docs/drivers/java/sync/current/fundamentals/data-formats/document-data-format-bson/#uuid-representation)

