---
id: okhttp3-removed
tier: 1
tier_label: Won't Build
title: OkHttp3 Support Removed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: web
---

Spring Boot 4.0 dropped the OkHttp3 client factory and BOM entry. Projects using OkHttp3RestClient or the managed dependency won't compile.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/config/RestConfig.java:[5,48]
  package org.springframework.http.client.OkHttp3ClientHttpRequestFactory does not exist
[ERROR] /src/main/java/com/example/config/RestConfig.java:[14,16]
  cannot find symbol
    symbol:   class OkHttp3ClientHttpRequestFactory
    location: class com.example.config.RestConfig
```

## What Changed {.what-changed}

The <code>OkHttp3ClientHttpRequestFactory</code> class was removed from Spring Framework 7.0. Spring Boot 4.0 also removed <code>com.squareup.okhttp3</code> from its managed dependency BOM. Any code creating a <code>RestClient</code> or <code>RestTemplate</code> with the OkHttp3 factory won't compile.

## Why {.why-changed}

The JDK's built-in <code>java.net.http.HttpClient</code> (since Java 11) and the Apache HttpClient 5 cover the same use cases. Maintaining a third HTTP client integration added complexity without clear benefit.

## The Fix {.diffs}

```diff-card
# // RestClient configuration
@@removed
import org.springframework.http.client.OkHttp3ClientHttpRequestFactory;

@Bean
public RestClient restClient() {
    return RestClient.builder()
        .requestFactory(new OkHttp3ClientHttpRequestFactory())
        .build();
}
@@added
import org.springframework.http.client.JdkClientHttpRequestFactory;

@Bean
public RestClient restClient() {
    return RestClient.builder()
        .requestFactory(new JdkClientHttpRequestFactory())
        .build();
}
```

```diff-card
# // pom.xml — remove OkHttp dependency
@@removed
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
</dependency>
@@added
<!-- JdkClientHttpRequestFactory uses java.net.http — no extra dependency needed -->
```

## How To Fix {.fixes}

#### Switch to JdkClientHttpRequestFactory (simplest).

Replace <code>OkHttp3ClientHttpRequestFactory</code> with <code>JdkClientHttpRequestFactory</code>. It uses the JDK's built-in HTTP client and requires no additional dependencies.

#### Switch to Apache HttpClient 5.

If you need connection pooling or advanced proxy configuration, use <code>HttpComponentsClientHttpRequestFactory</code> with Apache HttpClient 5. Add <code>org.apache.httpcomponents.client5:httpclient5</code> to your POM.

## Scope Check {.scope-check}

Search for <code>OkHttp3</code> and <code>com.squareup.okhttp3</code> across your codebase and build files. Check both production code and test configurations: OkHttp's <code>MockWebServer</code> is also affected.

## Watch Out {.watch-out}

- OkHttp interceptors for logging, retries, or authentication have no direct equivalent in the JDK client. Reimplement that logic with Spring's <code>ClientHttpRequestInterceptor</code>.
- OkHttp's <code>MockWebServer</code> is popular in tests. If you only use OkHttp for testing, consider switching to <code>MockRestServiceServer</code> or WireMock instead.

## Further Info {.further-info}

OkHttp itself still works as a plain library; what vanished is Spring's factory integration and the managed BOM version.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/okhttp3-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

