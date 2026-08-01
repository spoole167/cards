---
id: spring-amqp-retry-removed
tier: 1
tier_label: Won't Build
title: Spring AMQP RabbitRetryTemplateCustomizer Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: data-messaging
---

Boot 4.0 removed <code>RabbitRetryTemplateCustomizer</code>. Every bean implementing it fails to compile; publisher and consumer retry now have separate customiser interfaces.

## What You'll See {.error-output}

```error-output
@Bean
public RabbitRetryTemplateCustomizer retryCustomizer() {
    return (target, retryTemplate) -> { ... };
}

// Boot 4.0 compile error:
error: cannot find symbol
  symbol: class RabbitRetryTemplateCustomizer
```

## What Changed {.what-changed}

Spring AMQP 4.0 dropped its dependency on Spring Retry and now uses Spring Framework's retry abstraction. <code>RabbitRetryTemplateCustomizer</code> is replaced by <code>RabbitTemplateRetrySettingsCustomizer</code> (publisher-side) and <code>RabbitListenerRetrySettingsCustomizer</code> (consumer-side).

## Why {.why-changed}

Spring Boot 4.0 dropped Spring Retry across the board in favour of Spring Framework's native retry support. Spring AMQP followed the same path.

## The Fix {.diffs}

```diff-card
# // Publisher-side retry customizer
@@removed
import org.springframework.amqp.rabbit.retry.RabbitRetryTemplateCustomizer;

@Bean
public RabbitRetryTemplateCustomizer retryCustomizer() {
    return (target, retryTemplate) -> {
        retryTemplate.setRetryPolicy(...);
    };
}
@@added
import org.springframework.boot.autoconfigure.amqp.RabbitTemplateRetrySettingsCustomizer;

@Bean
public RabbitTemplateRetrySettingsCustomizer retryCustomizer() {
    return settings -> {
        settings.setMaxAttempts(3);
    };
}
```

```diff-card
# // Consumer-side retry customizer
@@removed
// Previously both publisher and consumer shared RabbitRetryTemplateCustomizer
@@added
import org.springframework.boot.autoconfigure.amqp.RabbitListenerRetrySettingsCustomizer;

@Bean
public RabbitListenerRetrySettingsCustomizer listenerRetryCustomizer() {
    return settings -> {
        settings.setMaxAttempts(5);
    };
}
```

## How To Fix {.fixes}

#### Replace with the split customiser interfaces.

Replace each <code>RabbitRetryTemplateCustomizer</code> bean with the publisher or consumer variant, whichever side it configured. The new interfaces set retry settings directly rather than manipulating a <code>RetryTemplate</code>.

## Scope Check {.scope-check}

Grep for <code>RabbitRetryTemplateCustomizer</code> across your source tree. Also check <code>import org.springframework.amqp.rabbit.retry</code> imports.

## Watch Out {.watch-out}

- The new customiser interfaces accept a settings object rather than a <code>RetryTemplate</code>. Retry policy logic expressed as <code>RetryTemplate</code> configuration must be re-expressed using the properties on the settings object.

## Further Info {.further-info}

One of several retry-related breaks in Boot 4.0. See also: spring-retry-removed, retry-semantics-change.

## Links {.footer-links}

- [spring-break module: spring-amqp-retry-removed](https://github.com/spoole167/spring-break/tree/main/spring-amqp-retry-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

