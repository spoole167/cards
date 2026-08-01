---
id: elasticsearch-rest5client
tier: 1
tier_label: Won't Build
title: Elasticsearch RestClient Renamed to Rest5Client
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: data-messaging
---

Spring Boot 4.0 auto-configures Rest5Client instead of RestClient for Elasticsearch. Code injecting the old type won't compile.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/ElasticsearchUsage.java:[5,35]
  error: cannot find symbol
    symbol:   class RestClient
    location: package org.elasticsearch.client
```

## What Changed {.what-changed}

Spring Boot 4.0 auto-configures <code>org.elasticsearch.client.Rest5Client</code> instead of <code>org.elasticsearch.client.RestClient</code>. Components that inject <code>RestClient</code> directly will fail because the auto-configured bean is now of type <code>Rest5Client</code>.

## Why {.why-changed}

The Elasticsearch Java client library introduced <code>Rest5Client</code> as the successor to the original <code>RestClient</code>. Spring Boot 4.0 adopted the newer type to track the upstream library.

## The Fix {.diffs}

```diff-card
# // Import
@@removed
import org.elasticsearch.client.RestClient;
@@added
import org.elasticsearch.client.Rest5Client;
```

```diff-card
# // Injection
@@removed
@Autowired
private RestClient restClient;
@@added
@Autowired
private Rest5Client restClient;
```

```diff-card
# // Customizer bean (if used)
@@removed
@Bean
public RestClientBuilderCustomizer customizer() { ... }
@@added
@Bean
public Rest5ClientBuilderCustomizer customizer() { ... }
```

## How To Fix {.fixes}

#### Rename RestClient to Rest5Client.

Replace <code>RestClient</code> with <code>Rest5Client</code> in all injection points, customiser beans, and direct usages. The API is broadly compatible, but check the Elasticsearch client changelog for subtle differences.

## Scope Check {.scope-check}

Search for <code>org.elasticsearch.client.RestClient</code> and <code>RestClientBuilderCustomizer</code> in your Java sources. Any direct injection of the low-level client is affected.

## Watch Out {.watch-out}

- Do not confuse Elasticsearch's <code>RestClient</code> with Spring Framework's <code>org.springframework.web.client.RestClient</code>: they are different classes, and only the Elasticsearch one changed.

## Further Info {.further-info}

The high-level RestHighLevelClient was already removed in Spring Boot 3.x; this is the next step in the same direction.

## Links {.footer-links}

- [spring-break module: elasticsearch-rest5client](https://github.com/spoole167/spring-break/tree/main/elasticsearch-rest5client)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

