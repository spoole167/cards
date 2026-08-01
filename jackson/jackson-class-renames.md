---
id: jackson-class-renames
tier: 1
tier_label: Won't Build
title: Jackson Class Renames
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: true
subsystem: jackson
---

Jackson 3.0 renamed core classes like JsonSerializer, JsonDeserializer, and SerializerProvider. Every custom serialiser is now broken.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/DateSerializer.java:[4,44]
  error: cannot find symbol
    symbol:   class JsonSerializer
    location: package tools.jackson.databind
[ERROR] /src/main/java/com/example/DateSerializer.java:[12,47]
  error: cannot find symbol
    symbol:   class SerializerProvider
    location: package tools.jackson.databind
```

## What Changed {.what-changed}

Jackson 3.0 renamed several core classes: <code>JsonSerializer</code> became <code>ValueSerializer</code>, <code>JsonDeserializer</code> became <code>ValueDeserializer</code>, <code>SerializerProvider</code> became <code>SerializationContext</code>, <code>DeserializationContext</code> kept its name but moved packages. Method signatures on these classes also changed.

## Why {.why-changed}

The "Json" prefix was misleading: Jackson handles YAML, XML, CBOR, and other formats, so serialisers work with values, not specifically JSON. The rename also aligned the class hierarchy with the new streaming API.

## The Fix {.diffs}

```diff-card
# // Custom serializer class
@@removed
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

public class DateSerializer extends JsonSerializer<LocalDate> {
    @Override
    public void serialize(LocalDate value, JsonGenerator gen,
            SerializerProvider provider) throws IOException {
@@added
import tools.jackson.databind.ser.ValueSerializer;
import tools.jackson.databind.SerializationContext;

public class DateSerializer extends ValueSerializer<LocalDate> {
    @Override
    public void serialize(LocalDate value, JsonGenerator gen,
            SerializationContext ctxt) throws IOException {
```

```diff-card
# // Custom deserializer class
@@removed
import com.fasterxml.jackson.databind.JsonDeserializer;

public class DateDeserializer extends JsonDeserializer<LocalDate> {
@@added
import tools.jackson.databind.deser.ValueDeserializer;

public class DateDeserializer extends ValueDeserializer<LocalDate> {
```

```diff-card
# // Exception handling
@@removed
import com.fasterxml.jackson.databind.JsonMappingException;
@@added
import tools.jackson.databind.DatabindException;
```

## How To Fix {.fixes}

#### OpenRewrite (recommended).

Run the <a href="https://docs.openrewrite.org/recipes/java/jackson/upgradejackson_2_3_typechanges">Jackson 3 type changes recipe</a>. It handles class renames, method signature updates, and import changes in one pass.

#### Manual migration.

Replace class names and imports file by file. The full mapping is in the <a href="https://github.com/FasterXML/jackson/blob/main/jackson3/MIGRATING_TO_JACKSON_3.md">Jackson 3 migration guide</a>. Don't forget to update method parameter types in overridden methods.

## Scope Check {.scope-check}

Search for <code>extends JsonSerializer</code>, <code>extends JsonDeserializer</code>, <code>SerializerProvider</code>, and <code>JsonMappingException</code>. Every custom serialiser, deserialiser, and exception handler needs updating.

## Watch Out {.watch-out}

- The method signatures changed too. <code>serialize()</code> now takes a <code>SerializationContext</code> instead of <code>SerializerProvider</code>. Renaming the class but keeping the old parameter type gives an abstract method error instead of a clean override.
- Annotations like <code>@JsonSerialize(using = ...)</code> still work but must reference the new class names. A custom serialiser registered via annotation will fail at runtime if it extends the wrong base class.

## Further Info {.further-info}

Driven by Jackson 3.0, upstream of Spring Boot 4.0, and finalised in Jackson 3.0-rc1. Affects spring-boot-starter-json consumers with custom serialisers. See also: jackson-group-id, jackson-exception-hierarchy.

## Links {.footer-links}

- [spring-break module: jackson-class-renames](https://github.com/spoole167/spring-break/tree/main/jackson-class-renames)

- [Jackson 3 migration guide](https://github.com/FasterXML/jackson/blob/main/jackson3/MIGRATING_TO_JACKSON_3.md)

