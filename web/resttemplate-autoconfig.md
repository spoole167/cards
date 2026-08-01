---
id: resttemplate-autoconfig
tier: 1
tier_label: Won't Build
title: RestTemplate Auto-Configuration Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: web
---

RestTemplateBuilder moved to the new spring-boot-restclient module. Imports from the old org.springframework.boot.web.client package no longer compile.

## What You'll See {.error-output}

```error-output
$ mvn clean compile
[ERROR] COMPILATION ERROR :
[ERROR] OrderClient.java:[3,42] package
  org.springframework.boot.web.client does not exist
[ERROR] OrderClient.java:[12,25] cannot find symbol
    symbol: class RestTemplateBuilder
[INFO] BUILD FAILURE
---
The build stops at compile. The application never starts.
```

## What Changed {.what-changed}

Spring Boot 4.0 removed the <code>org.springframework.boot.web.client</code> package. <code>RestTemplateBuilder</code> and its support classes relocated to the new <code>spring-boot-restclient</code> module, which covers both <code>RestClient</code> and <code>RestTemplate</code>.

## Why {.why-changed}

<code>RestTemplate</code> has been in maintenance mode since Spring 5, and Spring Framework 6.1 introduced <code>RestClient</code> as its fluent replacement. Removing the auto-configuration pushes migration to <code>RestClient</code> or <code>WebClient</code>.

## The Fix {.diffs}

```diff-card
# // Injecting the HTTP client — service class
@@removed
@Service
public class OrderClient {
    private final RestTemplate restTemplate;

    public OrderClient(RestTemplateBuilder builder) {
        this.restTemplate = builder
            .rootUri("https://api.example.com")
            .build();
    }

    public Order getOrder(Long id) {
        return restTemplate.getForObject(
            "/orders/{id}", Order.class, id);
    }
}
@@added
@Service
public class OrderClient {
    private final RestClient restClient;

    public OrderClient(RestClient.Builder builder) {
        this.restClient = builder
            .baseUrl("https://api.example.com")
            .build();
    }

    public Order getOrder(Long id) {
        return restClient.get()
            .uri("/orders/{id}", id)
            .retrieve()
            .body(Order.class);
    }
}
```

```diff-card
# // Test — using @RestClientTest
@@removed
@RestClientTest(OrderClient.class)
class OrderClientTest {
    @Autowired
    private MockRestServiceServer server;
    @Autowired
    private OrderClient client;
@@added
@RestClientTest(OrderClient.class)
class OrderClientTest {
    @Autowired
    private MockRestServiceServer server;
    @Autowired
    private OrderClient client;
    // Works the same — @RestClientTest supports RestClient
```

## How To Fix {.fixes}

#### Migrate to RestClient (recommended).

Replace <code>RestTemplate</code> usage with <code>RestClient</code>. Inject <code>RestClient.Builder</code> (auto-configured by Spring Boot 4) instead of <code>RestTemplateBuilder</code>. The API is fluent and supports the same customisers.

#### Update the RestTemplateBuilder import.

If migration is too large to do now, keep <code>RestTemplate</code>: add the <code>spring-boot-restclient</code> module (or its starter) to your build and update the <code>RestTemplateBuilder</code> import. This restores the old behaviour while you plan the migration.

#### Use WebClient for reactive code.

Reactive applications should use <code>WebClient</code>, which remains auto-configured in Spring Boot 4.0.

## Scope Check {.scope-check}

Search for <code>RestTemplate</code>, <code>RestTemplateBuilder</code>, and <code>RestTemplateCustomizer</code> across your codebase. Every import from the old package is a compile failure. Also check test classes that use <code>TestRestTemplate</code>: its auto-configuration may also be affected.

## Watch Out {.watch-out}

- <code>TestRestTemplate</code> is a separate concern. Check whether it is still auto-configured in your test slices or if you need to provide it manually.
- Third-party libraries that inject <code>RestTemplateBuilder</code> (e.g., Spring Cloud OpenFeign, some tracing libraries) may also break. Check transitive dependencies.
- The <code>RestClient</code> API is different from <code>RestTemplate</code>. Methods like <code>getForObject()</code> and <code>postForEntity()</code> become chained <code>.get().uri().retrieve().body()</code> calls. Budget time for the API rewrite as well as the bean swap.

## Further Info {.further-info}

Part of Spring Boot 4.0's wider modularisation of HTTP client support. See also: okhttp3-removed.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/resttemplate-autoconfig)

- [Testing changes in Spring Boot 4.0](https://rieckpil.de/whats-new-for-testing-in-spring-boot-4-0-and-spring-framework-7/)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

