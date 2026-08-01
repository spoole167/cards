---
id: testrest-template-removed
tier: 1
tier_label: Won't Build
title: TestRestTemplate Package Relocated
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: testing
---

TestRestTemplate moved to org.springframework.boot.resttestclient. Every test that imports the old package fails to compile.

## What You'll See {.error-output}

```error-output
$ mvn test-compile
[ERROR] /src/test/java/com/example/MyControllerTest.java:[3,56]
  error: package org.springframework.boot.test.web.client does not exist
[ERROR] /src/test/java/com/example/MyControllerTest.java:[15,5]
  error: cannot find symbol
    symbol: class TestRestTemplate
```

## What Changed {.what-changed}

<code>TestRestTemplate</code> was removed from <code>org.springframework.boot.test.web.client</code> and relocated to <code>org.springframework.boot.resttestclient</code>.

## Why {.why-changed}

Spring Boot 4.0 modularised its testing support. Separating the REST test clients into their own module shrinks the core test-autoconfigure footprint.

## The Fix {.diffs}

```diff-card
# // Test class import
@@removed
import org.springframework.boot.test.web.client.TestRestTemplate;
@@added
import org.springframework.boot.resttestclient.TestRestTemplate;
```

## How To Fix {.fixes}

#### Update the import.

Replace <code>org.springframework.boot.test.web.client.TestRestTemplate</code> with <code>org.springframework.boot.resttestclient.TestRestTemplate</code>. The class itself is functionally equivalent.

#### Consider RestTestClient.

The new <code>RestTestClient</code> provides a more modern, fluent API for REST testing and is the preferred approach in Boot 4.0. If you are refactoring tests anyway, this is a good time to migrate.

## Scope Check {.scope-check}

Search all test sources for <code>org.springframework.boot.test.web.client.TestRestTemplate</code> and for <code>@Autowired TestRestTemplate</code> or <code>@Autowired private TestRestTemplate</code>. Every hit needs the import updated.

## Watch Out {.watch-out}

- Custom configuration classes that create a <code>TestRestTemplate</code> bean (for authentication headers, say) need the import updated too, alongside the test classes that inject it.

## Further Info {.further-info}

Part of Spring Boot 4.0's testing overhaul: the class now ships in the spring-boot-test-rest-client module rather than spring-boot-test-autoconfigure. See also: springboottest-no-mockmvc.

## Links {.footer-links}

- [spring-break module: testrest-template-removed](https://github.com/spoole167/spring-break/tree/main/testrest-template-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

