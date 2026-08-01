---
id: modular-starters
tier: 2
tier_label: Won't Run
title: Modular Auto-Configuration
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: core
no_module: true
no_module_reason: |
  Which auto-configurations go missing depends on each application's mix of starters and third-party libraries; there is no universal "this class is gone" moment to assert in a single test. The canonical example (ObjectMapper missing after declaring only spring-boot-starter-web) does not reproduce on Boot 4.0: the Spring team preserved the Jackson transitive dependency through the web starter. The card stays because the audit advice holds: every team must make implicit auto-configuration dependencies explicit, using the Migration Guide's starter mapping table.
---

Boot 4.0 split auto-configuration into per-feature modules. Free beans (ObjectMapper, Validator) vanish without their starter. The app starts; NoSuchBeanDefinitionException hits at first use.

## What You'll See {.error-output}

```error-output
// App compiles and starts. Failure at first injection or endpoint call:

org.springframework.beans.factory.NoSuchBeanDefinitionException:
  No qualifying bean of type
  'com.fasterxml.jackson.databind.ObjectMapper' available

// Or from a @RestController trying to serialise a response:
org.springframework.http.converter.HttpMessageNotWritableException:
  No converter for [class com.example.OrderDto]
```

## What Changed {.what-changed}

The <code>spring-boot-autoconfigure</code> jar bundled auto-configuration for every Spring Boot feature. Boot 4.0 splits it into focused modules, one per feature area, and each activates only when its corresponding starter is explicitly present. Adding <code>spring-boot-starter-web</code> no longer implicitly enables Jackson, validation, or actuator auto-configuration.

## Why {.why-changed}

The monolithic jar meant any starter pulled in auto-configuration for hundreds of features the application never used. The split cuts startup time and memory, and makes dependencies explicit: you only pay for what you declare.

## The Fix {.diffs}

```diff-card
# // pom.xml — add starters that were previously implicit
@@removed
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
@@added
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- Now required explicitly — no longer implicit via web starter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-json</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## How To Fix {.fixes}

#### Audit and add missing starters.

Run <code>mvn dependency:tree</code> and compare what was transitive on Boot 3.5 with what your Boot 4.0 build pulls in. Add an explicit <code>spring-boot-starter-*</code> dependency for every feature your application uses: <code>spring-boot-starter-json</code> for Jackson, <code>spring-boot-starter-validation</code> for Bean Validation, <code>spring-boot-starter-actuator</code> for Micrometer metrics.

#### Watch @RestController endpoints.

If a controller returns an object that Jackson used to serialise automatically, the endpoint now returns <code>406 Not Acceptable</code> or throws <code>HttpMessageNotWritableException</code> rather than an obvious startup error. Add <code>spring-boot-starter-json</code> to restore the message converter.

## Scope Check {.scope-check}

Run the app and scan for <code>NoSuchBeanDefinitionException</code> in startup logs. Also grep your code for <code>@Autowired ObjectMapper</code>, <code>@Autowired Validator</code>, and similar injections that relied on implicit auto-configuration. Check <code>@RestController</code> methods that return non-String types: these rely on Jackson message converters.

## Watch Out {.watch-out}

- The context loads fine; the missing bean surfaces at first injection or when a converter is needed. Endpoint smoke tests find the gaps fastest.
- <code>spring-boot-starter-web</code> still brings Jackson onto the <em>classpath</em> as a transitive compile dependency: the classes are present but never configured into beans. The app compiles with no import errors and no signal that anything is wrong.
- Each application hits a different subset of missing beans depending on the features it uses. There is no single fix: audit, using the Migration Guide's starter mapping table as a checklist.

## Verify {.verify}

App starts and ObjectMapper, Validator, and other auto-configured beans are present: confirm with a test that autowires each one

## Further Info {.further-info}

In Boot 3.x every starter dragged in the entire auto-configuration bundle, so features like Jackson, Bean Validation, and Micrometer configured themselves without a declared dependency.

## Links {.footer-links}

- [spring-break module: modular-starters](https://github.com/spoole167/spring-break/tree/main/modular-starters)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [Modularizing Spring Boot (Spring Blog)](https://spring.io/blog/2025/10/28/modularizing-spring-boot)

