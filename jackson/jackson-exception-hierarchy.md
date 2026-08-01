---
id: jackson-exception-hierarchy
tier: 2
tier_label: Won't Run
title: JacksonException No Longer Extends IOException
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: true
subsystem: jackson
---

Jackson 3.0 changed the exception hierarchy so JacksonException extends RuntimeException instead of IOException. Existing catch blocks silently miss Jackson errors, and throws clauses become lies.

## What You'll See {.error-output}

```error-output
com.example.OrderController: Unhandled exception in /api/orders
tools.jackson.core.exc.StreamReadException: Unexpected character ('x' (code 120)):
  expected a valid value (JSON String, Number, Array, Object or token)
 at [Source: (String)"x]"; line: 1, column: 2]
 at tools.jackson.core.json.JsonParser._reportUnexpectedChar(JsonParser.java:...)
 ...
Caused by: tools.jackson.core.JacksonException (not an IOException!)
---
HTTP 500 Internal Server Error
{"timestamp":"2026-04-15T10:22:03","status":500,
 "error":"Internal Server Error","path":"/api/orders"}
```

## What Changed {.what-changed}

In Jackson 2.x, <code>JacksonException</code> extended <code>java.io.IOException</code>, so any <code>catch (IOException e)</code> block would also catch malformed-JSON errors. In Jackson 3.0, <code>JacksonException</code> extends <code>RuntimeException</code>. Catch blocks for <code>IOException</code> no longer intercept Jackson parsing failures, and methods that declared <code>throws IOException</code> for Jackson work now throw unchecked exceptions instead.

## Why {.why-changed}

The original hierarchy was a design mistake: it forced checked-exception handling even for in-memory parsing, which involves no I/O. Making <code>JacksonException</code> unchecked aligns with modern Java conventions and removes the boilerplate try/catch.

## The Fix {.diffs}

```diff-card
# // Controller error handling — before
@@removed
try {
    Order order = objectMapper.readValue(body, Order.class);
} catch (IOException e) {
    return ResponseEntity.badRequest().body("Invalid JSON");
}
@@added
try {
    Order order = objectMapper.readValue(body, Order.class);
} catch (JacksonException e) {
    return ResponseEntity.badRequest().body("Invalid JSON");
}
```

```diff-card
# // Spring @ExceptionHandler — before
@@removed
@ExceptionHandler(IOException.class)
public ResponseEntity<String> handleJsonError(IOException ex) {
@@added
@ExceptionHandler(JacksonException.class)
public ResponseEntity<String> handleJsonError(JacksonException ex) {
```

```diff-card
# // Method signature cleanup
@@removed
public Order parseOrder(String json) throws IOException {
@@added
public Order parseOrder(String json) {
```

## How To Fix {.fixes}

#### Search for catch blocks.

Grep for <code>catch (IOException</code> and <code>catch (JsonProcessingException</code>. Replace with <code>catch (JacksonException</code> where the intent is to handle Jackson parse failures. Keep <code>IOException</code> catches for genuine I/O work (file reads, sockets).

#### Update @ExceptionHandler advice.

If your <code>@RestControllerAdvice</code> catches <code>IOException</code> to return 400 on bad JSON, add a separate handler for <code>JacksonException</code>. Otherwise those errors now surface as unhandled 500s.

#### Clean up throws clauses.

Methods that declared <code>throws IOException</code> only for Jackson can drop the clause entirely. The compiler won't complain, but the stale declaration misleads readers.

## Scope Check {.scope-check}

Search for <code>catch (IOException</code>, <code>catch (JsonProcessingException</code>, and <code>throws IOException</code> near any <code>ObjectMapper</code> usage. Every hit where the catch was meant to intercept Jackson errors now misses at runtime. REST controllers, message listeners, and integration test assertions are the most common locations.

## Watch Out {.watch-out}

- The code compiles cleanly because <code>JacksonException</code> is now unchecked. You only discover the problem when bad input arrives, the catch block no longer fires, and clients get 500 errors instead of 400s.
- A broad <code>catch (Exception</code> still catches Jackson errors, but loses the ability to return a domain-appropriate error message. Narrow your catch clauses.
- Spring's built-in <code>HttpMessageNotReadableException</code> still wraps Jackson errors for <code>@RequestBody</code> parsing. The risk is in manual <code>ObjectMapper</code> calls inside your own code.

## Further Info {.further-info}

Driven by Jackson 3.0, upstream of Spring Boot 4.0. The change was announced in 2022 and finalised in Jackson 3.0-rc1. See also: jackson-group-id, jackson-class-renames.

## Links {.footer-links}

- [spring-break module: jackson-exception-hierarchy](https://github.com/spoole167/spring-break/tree/main/jackson-exception-hierarchy)

- [Jackson 3 migration guide](https://github.com/FasterXML/jackson/blob/main/jackson3/MIGRATING_TO_JACKSON_3.md)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

