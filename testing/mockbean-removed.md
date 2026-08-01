---
id: mockbean-removed
tier: 1
tier_label: Won't Build
title: '@MockBean / @SpyBean Removed'
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: true
subsystem: testing
max_pages: 2
---

Spring Boot 4.0 removed the @MockBean and @SpyBean annotations. Test sources that import them fail to compile. Replace with @MockitoBean and @MockitoSpyBean.

## What You'll See {.error-output}

```error-output
$ mvn clean test
[ERROR] COMPILATION ERROR :
[ERROR] OrderServiceTest.java:[3,48] package
  org.springframework.boot.test.mock.mockito does not exist
[ERROR] OrderServiceTest.java:[18,6] cannot find symbol
    symbol: class MockBean
[INFO] BUILD FAILURE
---
The build stops at test-compile. No tests run.
```

## What Changed {.what-changed}

The <code>@MockBean</code> and <code>@SpyBean</code> annotations from <code>org.springframework.boot.test.mock.mockito</code> were deprecated in Spring Boot 3.4 and removed in 4.0. They are replaced by <code>@MockitoBean</code> and <code>@MockitoSpyBean</code> from <code>org.springframework.test.context.bean.override.mockito</code>, which live in Spring Framework 7 itself rather than Spring Boot.

## Why {.why-changed}

The new annotations live in Spring Framework's test context, so they work in plain Framework tests as well as Boot tests. Their cleaner bean override mechanism also avoids some of the context caching issues that plagued <code>@MockBean</code>.

## The Fix {.diffs}

```diff-card
# // Import change
@@removed
import org.springframework.boot.test.mock.mockito.MockBean;
@@added
import org.springframework.test.context.bean.override.mockito.MockitoBean;
```

```diff-card
@@removed
import org.springframework.boot.test.mock.mockito.SpyBean;
@@added
import org.springframework.test.context.bean.override.mockito.MockitoSpyBean;
```

```diff-card
# // Annotation usage
@@removed
@MockBean
private OrderRepository orderRepository;

@SpyBean
private NotificationService notificationService;
@@added
@MockitoBean
private OrderRepository orderRepository;

@MockitoSpyBean
private NotificationService notificationService;
```

## How To Fix {.fixes}

#### Find-and-replace annotations.

Replace <code>@MockBean</code> with <code>@MockitoBean</code> and <code>@SpyBean</code> with <code>@MockitoSpyBean</code>. Update the imports from <code>org.springframework.boot.test.mock.mockito</code> to <code>org.springframework.test.context.bean.override.mockito</code>.

#### OpenRewrite recipe.

The Spring Boot 4.0 OpenRewrite migration recipe handles this rename automatically across the entire test codebase.

#### Check for @MockBean attributes.

If you used <code>@MockBean(name = "...")</code> or <code>@MockBean(classes = ...)</code>, verify the equivalent attributes exist on <code>@MockitoBean</code>. The attribute names may differ slightly.

## Scope Check {.scope-check}

Search for <code>@MockBean</code> and <code>@SpyBean</code> across <code>src/test</code>. Every test class using these annotations needs updating. That can be dozens or hundreds of test files. Count them before starting.

## Watch Out {.watch-out}

- If you upgrade without running <code>mvn clean</code>, stale test classes compiled against 3.5 can still load and run. Spring no longer recognises the old annotations, so the mock fields silently stay <code>null</code> and the first symptom is a <code>NullPointerException</code> mid-test, which looks like a runtime injection failure. Run a clean build and the real compile failure appears.
- The <code>@MockitoBean</code> reset behaviour between tests may differ from <code>@MockBean</code>. If you relied on mocks being reset between test methods, verify that behaviour is preserved.
- The new annotations are in the Spring Framework package, not Spring Boot. If you have test utility classes that reference the old annotation by fully qualified name (e.g., in custom annotations or reflection), those references need updating too.
- IDE templates and test generators may still produce <code>@MockBean</code>. Update your IDE templates after migrating.

## Links {.footer-links}

- [spring-break module: mockbean-removed](https://github.com/spoole167/spring-break/tree/main/mockbean-removed)

- [Testing changes in Spring Boot 4.0](https://rieckpil.de/whats-new-for-testing-in-spring-boot-4-0-and-spring-framework-7/)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

