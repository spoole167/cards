---
id: deprecated-classes-removed
tier: 2
tier_label: Won't Run
title: Deprecated RestTemplateBuilder APIs Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
no_module: true
no_module_reason: |
  The existing module (deprecated-classes-removed) autowires RestTemplate and asserts non-null, the same assertion resttemplate-autoconfig makes, so it does not isolate a specific deprecated-API removal. A dedicated module would need a test that calls a removed method (e.g. additionalMessageConverters()) at compile time. Treat the existing module as a second angle on the resttemplate-autoconfig break, not an independent canonical test.
---

Deprecated <code>RestTemplateBuilder</code> methods are removed in Boot 4.0: code that compiled with warnings on 3.5 no longer compiles. Migrate to <code>RestClient</code>.

## What You'll See {.error-output}

```error-output
// Calling a deprecated builder method
RestTemplate rt = builder
    .additionalMessageConverters(new MappingJackson2HttpMessageConverter())
    .build();

// Boot 3.5: compiles with deprecation warning, runs fine.

// Boot 4.0: compile error.
error: cannot find symbol
  symbol: method additionalMessageConverters(MappingJackson2HttpMessageConverter)
```

## What Changed {.what-changed}

Spring Boot 3.x deprecated several <code>RestTemplateBuilder</code> methods to signal the migration path to <code>RestClient</code> (introduced in Spring Framework 6.1). Boot 4.0 removes these deprecated methods entirely, following the standard deprecation-then-removal cycle.

## Why {.why-changed}

<code>RestTemplate</code> is in maintenance mode. The modern replacement is <code>RestClient</code>, which offers a fluent, functional API and is the recommended HTTP client for new development in Spring Framework 6.x+. Removing the deprecated builder methods accelerates the migration.

## The Fix {.diffs}

```diff-card
# // Migrate from RestTemplate to RestClient
@@removed
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
    return builder
        .additionalMessageConverters(new MappingJackson2HttpMessageConverter())
        .build();
}
@@added
@Bean
public RestClient restClient() {
    return RestClient.builder()
        .messageConverters(converters ->
            converters.add(new MappingJackson2HttpMessageConverter()))
        .build();
}
```

## How To Fix {.fixes}

#### Migrate to RestClient.

<code>RestClient</code> is the modern replacement. It offers a similar fluent builder API and is available from Spring Framework 6.1 onward (Spring Boot 3.2+). Migrating removes the deprecated call and future-proofs the HTTP client code.

#### If RestTemplate must be kept, remove the deprecated builder calls.

Construct a <code>RestTemplate</code> directly (via constructor or with a plain <code>RestTemplateBuilder.build()</code>) and configure message converters via <code>restTemplate.getMessageConverters().add(...)</code> after construction.

## Scope Check {.scope-check}

Run <code>mvn compile</code> with <code>-Xlint:deprecation</code> on Boot 3.5 to find every deprecated API call before upgrading to 4.0. Pay particular attention to <code>RestTemplateBuilder</code> method chains: each deprecated call is a potential compile error on 4.0.

## Watch Out {.watch-out}

- RestTemplate auto-configuration is also removed in Boot 4.0, a separate break from deprecated method removal. If you rely on an auto-configured <code>RestTemplate</code> bean rather than declaring one, see the <code>resttemplate-autoconfig</code> card.

## Further Info {.further-info}

Affected methods include additionalMessageConverters(), messageConverters(), and requestFactory(). See also: resttemplate-autoconfig.

## Links {.footer-links}

- [spring-break module: deprecated-classes-removed](https://github.com/spoole167/spring-break/tree/main/deprecated-classes-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [RestClient migration guide](https://docs.spring.io/spring-framework/reference/integration/rest-clients.html)

