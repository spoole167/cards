---
id: httpheaders-multivaluemap
tier: 1
tier_label: Won't Build
title: HttpHeaders No Longer Implements MultiValueMap
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: web
---

HttpHeaders no longer implements MultiValueMap. Code that treats headers as a general-purpose map fails to compile.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/HttpHeadersUsage.java:[8,24]
  error: incompatible types: HttpHeaders cannot be converted to MultiValueMap
[ERROR] /src/main/java/com/example/HttpHeadersUsage.java:[12,16]
  error: cannot find symbol
    symbol: method containsKey(String)
```

## What Changed {.what-changed}

<code>HttpHeaders</code> no longer implements <code>MultiValueMap&lt;String, String&gt;</code> or <code>Map&lt;String, List&lt;String&gt;&gt;</code>. Map-style methods like <code>containsKey()</code>, <code>keySet()</code>, and <code>entrySet()</code> are gone from the type. Header-specific methods replace them.

## Why {.why-changed}

Treating HTTP headers as a generic map encouraged misuse and blocked performance work. The dedicated <code>HttpHeaders</code> API is more expressive and frees the backing store from the Map contract.

## The Fix {.diffs}

```diff-card
# // Replace Map method calls with HttpHeaders equivalents
@@removed
headers.containsKey("Content-Type")
@@added
headers.containsHeader("Content-Type")
```

```diff-card
@@removed
Set<String> names = headers.keySet();
@@added
Set<String> names = headers.headerNames();
```

```diff-card
@@removed
Set<Map.Entry<String, List<String>>> entries = headers.entrySet();
@@added
Set<Map.Entry<String, List<String>>> entries = headers.headerSet();
```

```diff-card
# // Passing HttpHeaders where MultiValueMap is expected
@@removed
void process(MultiValueMap<String, String> map) { ... }
process(headers);
@@added
void process(HttpHeaders headers) { ... }
// or, if the method signature can't change:
process(headers.toMultiValueMap());
```

## How To Fix {.fixes}

#### Replace Map methods with HttpHeaders methods.

<code>containsKey(k)</code> → <code>containsHeader(k)</code>, <code>keySet()</code> → <code>headerNames()</code>, <code>entrySet()</code> → <code>headerSet()</code>. These replacements compile on both 3.5 and 4.0.

#### For method signatures that accept MultiValueMap.

If you have a utility method that accepts <code>MultiValueMap&lt;String, String&gt;</code> and you pass <code>HttpHeaders</code> into it, either change the parameter type to <code>HttpHeaders</code>, or call <code>headers.toMultiValueMap()</code> at the call site.

## Scope Check {.scope-check}

Search for <code>MultiValueMap</code> usages near <code>HttpHeaders</code> across your codebase. Also search for <code>.containsKey(</code>, <code>.keySet()</code>, <code>.entrySet()</code> called on variables of type <code>HttpHeaders</code>.

## Watch Out {.watch-out}

- Code in Spring interceptors, filters, and custom converters that inspects headers using Map idioms is the most common source of these failures. Check <code>HandlerInterceptor</code>, <code>ClientHttpRequestInterceptor</code>, and <code>ResponseBodyAdvice</code> implementations.
- <code>toMultiValueMap()</code> returns a copy, not a live view. Mutations to the returned map do not affect the original <code>HttpHeaders</code>.

## Further Info {.further-info}

Driven by Spring Framework 7.0. HttpHeaders had been a MultiValueMap subtype since early in Spring's history.

## Links {.footer-links}

- [spring-break module: httpheaders-multivaluemap](https://github.com/spoole167/spring-break/tree/main/httpheaders-multivaluemap)

- [Spring Framework 7.0 Release Notes](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-7.0-Release-Notes)

