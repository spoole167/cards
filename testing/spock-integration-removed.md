---
id: spock-integration-removed
tier: 2
tier_label: Won't Run
title: Spring Boot Spock Integration Removed
series: spring-boot 3.5 → 4.0
effort: L
openrewrite: false
max_pages: 2
subsystem: testing
no_module: true
no_module_reason: |
  A module demonstrating this break would itself be a Spock test, written in Groovy. The spring-break suite uses JUnit 5 throughout and has no Groovy build infrastructure; a mixed-language module would obscure the break. The break is well-defined: Spock does not yet support Groovy 5, Spring Boot 4.0's baseline, so any @SpringBootTest Spock specification does not run.
---

Spring Boot 4.0 requires Groovy 5; Spock does not support it yet. Spock-based <code>@SpringBootTest</code> specifications stop running.

## What You'll See {.error-output}

```error-output
// Spock specification in a Boot 4.0 project:
@SpringBootTest
class MyServiceSpec extends Specification {
    @Autowired MyService service

    def "should do something"() {
        expect: service.doThing() == "result"
    }
}

// Fails at compile or runtime — Spock and Groovy 5 are incompatible.
// Exact error depends on Groovy/Spock versions on the classpath.
```

## What Changed {.what-changed}

Spring Boot 4.0 upgraded its Groovy baseline to Groovy 5. Spock Framework (as of this writing) requires Groovy 4.x or earlier, so Spring Boot removed the Spock integration.

## Why {.why-changed}

The Groovy 5 upgrade aligned Boot with the current Groovy ecosystem. Spock's Groovy 5 support was not ready in time for the Boot 4.0 release, making the integration impossible to ship.

## The Fix {.diffs}

```diff-card
# // Spock specification → JUnit 5 migration
@@removed
// MyServiceSpec.groovy
@SpringBootTest
class MyServiceSpec extends Specification {
    @Autowired MyService service

    def "should return result"() {
        expect: service.doThing() == "result"
    }
}
@@added
// MyServiceTest.java
@SpringBootTest
class MyServiceTest {
    @Autowired MyService service;

    @Test
    void shouldReturnResult() {
        assertThat(service.doThing()).isEqualTo("result");
    }
}
```

## How To Fix {.fixes}

#### Migrate Spock specifications to JUnit 5.

Rewrite Spock-based Spring integration tests using JUnit 5 and AssertJ. The <code>given/when/then</code> blocks map to JUnit 5 methods and AssertJ assertions. Data-driven tests (<code>where:</code> blocks) translate to <code>@ParameterizedTest</code> with <code>@MethodSource</code> or <code>@CsvSource</code>.

#### Wait for Spock's Groovy 5 support if migration is impractical.

If the volume of Spock tests makes immediate migration impractical, track the Spock project's Groovy 5 roadmap and defer the Boot 4.0 upgrade for those modules.

## Scope Check {.scope-check}

Find all <code>.groovy</code> test files that extend <code>Specification</code> and carry <code>@SpringBootTest</code>, <code>@SpringRunner</code>, or any Spring test slice annotation. These are the tests that break.

## Watch Out {.watch-out}

- Pure unit Spock tests, those that never load a Spring context, still work as long as Groovy compilation itself succeeds. The breakage is in tests that depend on Spring Boot's test infrastructure.

## Verify {.verify}

Spock specifications that use @SpringBootTest or @SpringRunner pass after migrating to JUnit 5 or waiting for Spock's Groovy 5 support

## Further Info {.further-info}

The removal covers the <code>spock-spring</code> module and Boot's <code>@SpringBootTest</code> support for Spock specifications. Failures appear at Groovy compile time or at runtime before any test method executes.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [Spock Framework GitHub](https://github.com/spockframework/spock)

