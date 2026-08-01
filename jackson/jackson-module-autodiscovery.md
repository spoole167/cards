---
id: jackson-module-autodiscovery
tier: 3
tier_label: Wrong Results
title: Jackson 3 Auto-Discovers All Modules on Classpath
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: jackson
---

Jackson 3 registers every module it finds on the classpath. Modules that arrive as transitive dependencies can silently change your JSON output. Disable with <code>spring.jackson.find-and-add-modules=false</code>.

## What You'll See {.error-output}

```error-output
// Boot 3.5: only well-known modules registered. Output is predictable.
{"amount": 150.00, "currency": "EUR"}

// Boot 4.0: a transitive Jackson module on the classpath changes
// BigDecimal serialization:
{"amount": "150.00", "currency": "EUR"}

// No error, no warning. The JSON schema changed.
```

## What Changed {.what-changed}

Jackson 3 uses the Java ServiceLoader mechanism to find and register all <code>com.fasterxml.jackson.databind.Module</code> implementations on the classpath. Spring Boot no longer controls which modules are registered: every module present is activated.

## Why {.why-changed}

Automatic discovery lets library authors ship Jackson integration without requiring explicit registration. Spring Boot aligned with Jackson 3's convention.

## The Fix {.diffs}

```diff-card
# // application.properties — disable auto-discovery to restore Boot 3.5 behaviour
@@removed
# Boot 3.5: no property needed, only well-known modules registered
@@added
# Boot 4.0: disable auto-discovery to prevent unexpected module activation
spring.jackson.find-and-add-modules=false
```

## How To Fix {.fixes}

#### Audit classpath modules and disable auto-discovery if needed.

Run <code>mvn dependency:tree</code> and look for any artifacts ending in <code>-module</code> or containing <code>jackson</code>. For each one, verify its effect on your serialisation output. If unexpected modules are activating, set <code>spring.jackson.find-and-add-modules=false</code> and register only the modules you need explicitly.

#### Explicitly register required modules.

After disabling auto-discovery, register required modules via a <code>Jackson2ObjectMapperBuilderCustomizer</code> (now renamed <code>JsonMapperBuilderCustomizer</code>) bean that calls <code>modules(new JavaTimeModule(), ...)</code>.

## Scope Check {.scope-check}

Run your application's serialisation tests after upgrading. Compare JSON output field by field for any type that uses custom serialisation or has complex types (dates, BigDecimal, Optional, collections). Grep for <code>ObjectMapper</code> customisation code to understand what was previously registered manually.

## Watch Out {.watch-out}

- This interacts with the Jackson 2 compatibility shim (<code>spring-boot-jackson2</code>). If you have both Jackson 2 and Jackson 3 modules on the classpath, both may be auto-discovered and conflict.
- Test suites that stub or mock the <code>ObjectMapper</code> will not catch this: the change only manifests with a real, auto-configured <code>ObjectMapper</code> bean.

## Verify {.verify}

JSON serialisation output is identical before and after the upgrade, or unexpected module behaviour is controlled with spring.jackson.find-and-add-modules

## Further Info {.further-info}

In Boot 3.5 (Jackson 2), Spring Boot registered a fixed set of well-known modules: Java time, parameter names, and a handful of others. Everything else needed explicit registration. A module a library brought in for its own internal use can now alter your own <code>ObjectMapper</code> output.

## Links {.footer-links}

- [spring-break module: jackson-module-autodiscovery](https://github.com/spoole167/spring-break/tree/main/jackson-module-autodiscovery)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

