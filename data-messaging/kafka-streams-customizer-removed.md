---
id: kafka-streams-customizer-removed
tier: 1
tier_label: Won't Build
title: Kafka StreamsBuilderFactoryBeanCustomizer Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: org.openrewrite.java.spring.boot4.MigrateAutoconfigurePackages
subsystem: data-messaging
---

Spring Boot's StreamsBuilderFactoryBeanCustomizer is gone. Replace it with Spring Kafka's StreamsBuilderFactoryBeanConfigurer.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/KafkaCustomizerUsage.java:[3,60]
  error: package org.springframework.boot.autoconfigure.kafka does not contain
  StreamsBuilderFactoryBeanCustomizer
```

## What Changed {.what-changed}

<code>org.springframework.boot.autoconfigure.kafka.StreamsBuilderFactoryBeanCustomizer</code> has been removed. The replacement is <code>org.springframework.kafka.config.StreamsBuilderFactoryBeanConfigurer</code> from Spring Kafka itself.

## Why {.why-changed}

Spring Boot's interface was a thin duplicate of one Spring Kafka already provides. Removing it cuts an unnecessary indirection.

## The Fix {.diffs}

#### Removed
```diff-card
# // Import
import org.springframework.boot.autoconfigure.kafka.StreamsBuilderFactoryBeanCustomizer;
```

#### Added
```diff-card
import org.springframework.kafka.config.StreamsBuilderFactoryBeanConfigurer;
```

#### Removed
```diff-card
# // Bean method signature
@Bean
public StreamsBuilderFactoryBeanCustomizer customizer() { ... }
```

#### Added
```diff-card
@Bean
public StreamsBuilderFactoryBeanConfigurer configurer() { ... }
```

#### Removed
```diff-card
# // implements clause
public class MyCustomizer implements StreamsBuilderFactoryBeanCustomizer {
```

#### Added
```diff-card
public class MyCustomizer implements StreamsBuilderFactoryBeanConfigurer {
```

## How To Fix {.fixes}

#### Switch to StreamsBuilderFactoryBeanConfigurer.

Replace the import and the implements / return type declaration. The interface method signature is compatible: the method you override is <code>configure(StreamsBuilderFactoryBean)</code> in both cases.

## Scope Check {.scope-check}

Search for <code>StreamsBuilderFactoryBeanCustomizer</code> in all Java/Kotlin sources. Only Kafka Streams applications are affected.

## Watch Out {.watch-out}

- The method name on <code>StreamsBuilderFactoryBeanConfigurer</code> is <code>configure</code>, same as the old <code>customize</code>. Verify the exact method name in the Spring Kafka version you're using: it differs between minor versions.

## Further Info {.further-info}

The removal is part of shrinking Boot's Kafka auto-configuration surface; the upstream Spring Kafka interface is the long-term home.

## Links {.footer-links}

- [spring-break module: kafka-streams-customizer-removed](https://github.com/spoole167/spring-break/tree/main/kafka-streams-customizer-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

