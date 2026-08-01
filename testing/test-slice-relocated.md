---
id: test-slice-relocated
tier: 1
tier_label: Won't Build
title: Test-Slice Annotation Starters Relocated
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: true
subsystem: testing
---

Spring Boot 4.0 relocated the test-slice annotations. The old packages (e.g. org.springframework.boot.test.autoconfigure.orm.jpa) no longer exist, so imports of @DataJpaTest, @WebMvcTest, and friends fail to compile.

## What You'll See {.error-output}

```error-output
$ mvn clean test
[ERROR] COMPILATION ERROR :
[ERROR] UserRepositoryTest.java:[5,50] package
  org.springframework.boot.test.autoconfigure.orm.jpa does not exist
[ERROR] UserRepositoryTest.java:[12,2] cannot find symbol
    symbol: class DataJpaTest
[INFO] BUILD FAILURE
---
The build stops at test-compile. No tests run.
```

## What Changed {.what-changed}

Spring Boot 4.0 reorganised the test auto-configuration packages. The old test-slice packages, such as <code>org.springframework.boot.test.autoconfigure.orm.jpa</code>, no longer exist, so existing imports of <code>@DataJpaTest</code>, <code>@WebMvcTest</code>, and the other slice annotations fail to compile. The annotations now live in the new per-technology test modules.

## Why {.why-changed}

The test auto-configuration was restructured to support the new bean override mechanism and to align the package structure with Spring Framework 7's test context changes.

## The Fix {.diffs}

```diff-card
# // Custom test-slice annotation
@@removed
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@BootstrapWith(SpringBootTestContextBootstrapper.class)
@OverrideAutoConfiguration(enabled = false)
@TypeExcludeFilters(CustomSliceTypeExcludeFilter.class)
@AutoConfigureCache
@AutoConfigureCustomService
@ImportAutoConfiguration
public @interface CustomServiceTest {
}
@@added
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@BootstrapWith(SpringBootTestContextBootstrapper.class)
@OverrideAutoConfiguration(enabled = false)
@TypeExcludeFilters(CustomSliceTypeExcludeFilter.class)
@AutoConfigureCache
@AutoConfigureCustomService
@ImportAutoConfiguration
public @interface CustomServiceTest {
    // Annotation structure unchanged, but verify your
    // TypeExcludeFilter extends the correct base class
    // and the imported auto-configurations still resolve
}
```

## How To Fix {.fixes}

#### Update the test-slice imports.

Every test importing a slice annotation from the old <code>org.springframework.boot.test.autoconfigure</code> packages needs the import updated to the annotation's new home in the per-technology test modules. The Spring Boot 4.0 OpenRewrite recipe handles this across the whole test codebase; your IDE can re-resolve individual imports.

#### Update custom test-slice annotations.

If you wrote custom test-slice annotations (like <code>@CustomServiceTest</code>), verify that the <code>TypeExcludeFilter</code>, bootstrapper, and auto-configuration imports still resolve. Update fully qualified class references to match the new package locations, then run the full suite to confirm each slice still loads a thin context.

## Scope Check {.scope-check}

Search test sources for imports from <code>org.springframework.boot.test.autoconfigure</code>: every one of them fails to compile on 4.0. Also search for custom annotations that use <code>@TypeExcludeFilters</code>, <code>@OverrideAutoConfiguration</code>, or <code>@ImportAutoConfiguration</code>.

## Watch Out {.watch-out}

- Once the imports compile again, check what context each slice loads. A misconfigured slice can silently load a full application context instead of a thin one, making tests pass but run much slower and test less precisely.
- Libraries that provide custom test slices (Spring Cloud, Spring Modulith) need to be updated too. Check their Spring Boot 4.0 compatibility versions.

## Further Info {.further-info}

Part of Spring Boot 4.0's modularisation: test auto-configuration moved out of the monolithic spring-boot-test-autoconfigure module into per-technology test modules (spring-boot-data-jpa-test, spring-boot-webmvc-test, and so on), and the annotation packages moved with it. Registrations still living in spring.factories are a separate break: see spring-factories-autoconfig-imports.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/test-slice-relocated)

- [Testing changes in Spring Boot 4.0](https://rieckpil.de/whats-new-for-testing-in-spring-boot-4-0-and-spring-framework-7/)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

