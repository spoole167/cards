---
id: pulsar-reactive-removed
tier: 2
tier_label: Won't Run
title: Spring Pulsar Reactive Auto-Configuration Removed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: data-messaging
---

Spring Boot 4.0 dropped reactive Pulsar auto-configuration. <code>ReactivePulsarTemplate</code> and <code>ReactivePulsarClient</code> beans are no longer available. The app starts but fails on first use.

## What You'll See {.error-output}

```error-output
// Boot 4.0 startup or runtime failure:
org.springframework.beans.factory.NoSuchBeanDefinitionException:
  No qualifying bean of type
  'org.springframework.pulsar.reactive.core.ReactivePulsarTemplate' available

// Or auto-wiring failure:
Field reactivePulsarTemplate required a bean of type
  'ReactivePulsarTemplate' that could not be found.
```

## What Changed {.what-changed}

<code>spring-boot-starter-pulsar-reactive</code> and the corresponding auto-configuration module were removed from Spring Boot 4.0. The auto-configuration for <code>ReactivePulsarClient</code>, <code>ReactivePulsarTemplate</code>, and <code>ReactiveMessageListenerContainer</code> no longer exists in Boot's auto-configure artifact.

## Why {.why-changed}

Boot 4.0's modular restructuring dropped auto-configurations for reactive integration layers with limited adoption relative to their maintenance cost. The imperative Spring Pulsar integration remains fully supported.

## The Fix {.diffs}

```diff-card
# // Replace reactive auto-config with explicit bean definitions
@@removed
<!-- Boot 3.5: auto-configured reactive Pulsar client -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-pulsar-reactive</artifactId>
</dependency>
@@added
<!-- Boot 4.0: use imperative starter or configure reactive beans manually -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-pulsar</artifactId>
</dependency>
```

## How To Fix {.fixes}

#### Switch to the imperative Pulsar integration or configure reactive beans manually.

If the reactive API is required, declare the reactive Pulsar beans explicitly in a <code>@Configuration</code> class. If reactive messaging is not critical, switch to the imperative <code>spring-boot-starter-pulsar</code> which remains auto-configured in Boot 4.0.

## Scope Check {.scope-check}

Grep for <code>ReactivePulsarTemplate</code>, <code>ReactivePulsarClient</code>, and <code>spring-boot-starter-pulsar-reactive</code> across your source and build files.

## Watch Out {.watch-out}

- The imperative <code>PulsarTemplate</code> and <code>PulsarClient</code> auto-configuration in <code>spring-boot-starter-pulsar</code> is unaffected by this change.

## Verify {.verify}

Reactive Pulsar producers and consumers initialise correctly after removing the dependency on Boot's auto-configuration

## Further Info {.further-info}

The Spring Pulsar reactive library itself survives; only Boot's auto-configuration went away, so the beans must be declared by hand.

## Links {.footer-links}

- [spring-break module: pulsar-reactive-removed](https://github.com/spoole167/spring-break/tree/main/pulsar-reactive-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [Spring Pulsar Documentation](https://docs.spring.io/spring-pulsar/reference/index.html)

