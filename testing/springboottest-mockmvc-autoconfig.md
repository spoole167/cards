---
id: springboottest-mockmvc-autoconfig
tier: 2
tier_label: Won't Run
title: '@SpringBootTest No Longer Auto-Configures MockMvc'
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: testing
no_module: true
no_module_reason: |
  Premise unverifiable as a 3.5 → 4.0 break. Boot 3.5 also does not auto-configure MockMvc without @AutoConfigureMockMvc: the @Autowired field is null on both versions. @AutoConfigureMockMvc has always been required to get a MockMvc bean in a @SpringBootTest. The migration guide entry appears to describe behaviour that was never present in 3.5.
---

<code>@SpringBootTest</code> no longer auto-configures <code>MockMvc</code> in Boot 4.0. Add <code>@AutoConfigureMockMvc</code> to your test class or the <code>MockMvc</code> bean won't be available.

## What You'll See {.error-output}

```error-output
@SpringBootTest
class MyControllerTest {
    @Autowired
    MockMvc mockMvc;  // null in Boot 4.0

    @Test
    void test() throws Exception {
        mockMvc.perform(get("/api/hello"))  // NullPointerException
               .andExpect(status().isOk());
    }
}

// Boot 4.0 failure:
org.springframework.beans.factory.NoSuchBeanDefinitionException:
  No qualifying bean of type
  'org.springframework.test.web.servlet.MockMvc' available
```

## What Changed {.what-changed}

MockMvc auto-configuration was decoupled from <code>@SpringBootTest</code>. The MOCK webEnvironment still creates a mock servlet context, but the <code>MockMvc</code> bean must now be opted into with <code>@AutoConfigureMockMvc</code>.

## Why {.why-changed}

The implicit bean was a hidden dependency: tests got MockMvc without declaring they needed it. Opt-in matches Boot 4.0's explicit-over-implicit theme and keeps the test context leaner when MockMvc isn't needed.

## The Fix {.diffs}

```diff-card
# // Add @AutoConfigureMockMvc
@@removed
@SpringBootTest
class MyControllerTest {
    @Autowired
    MockMvc mockMvc;
}
@@added
@SpringBootTest
@AutoConfigureMockMvc
class MyControllerTest {
    @Autowired
    MockMvc mockMvc;
}
```

```diff-card
# // @WebMvcTest is unaffected — MockMvc is its primary purpose
@@added
// @WebMvcTest already includes MockMvc auto-configuration.
// Only @SpringBootTest is affected.
```

## How To Fix {.fixes}

#### Add @AutoConfigureMockMvc to the test class.

This is a one-annotation fix. Add <code>@AutoConfigureMockMvc</code> alongside <code>@SpringBootTest</code> on any test class that autowires <code>MockMvc</code>.

## Scope Check {.scope-check}

Search for <code>@SpringBootTest</code> test classes that also autowire <code>MockMvc</code> (look for <code>@Autowired</code> fields or constructor parameters of type <code>MockMvc</code>). Each needs <code>@AutoConfigureMockMvc</code> added. <code>@WebMvcTest</code> tests are unaffected.

## Watch Out {.watch-out}

- The same change applies to <code>WebTestClient</code> and <code>TestRestTemplate</code>. Boot 4.0 also no longer auto-configures these in <code>@SpringBootTest</code>: add <code>@AutoConfigureRestTestClient</code> or <code>@AutoConfigureTestRestTemplate</code> respectively.

## Verify {.verify}

MockMvc is available in @SpringBootTest tests after adding @AutoConfigureMockMvc

## Further Info {.further-info}

Per the migration guide, a Boot 3.5 @SpringBootTest with the default MOCK webEnvironment supplied MockMvc without any extra annotation; tests relying on that fail with NoSuchBeanDefinitionException on 4.0.

## Links {.footer-links}

- [spring-break module: springboottest-mockmvc-autoconfig](https://github.com/spoole167/spring-break/tree/main/springboottest-mockmvc-autoconfig)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

