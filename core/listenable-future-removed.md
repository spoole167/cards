---
id: listenable-future-removed
tier: 1
tier_label: Won't Build
title: ListenableFuture Removed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: core
---

Spring Framework removed ListenableFuture entirely. All async code using it must switch to CompletableFuture.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/service/AsyncService.java:[4,49]
  error: cannot find symbol
    symbol:   class ListenableFuture
    location: package org.springframework.util.concurrent
[ERROR] /src/main/java/com/example/service/AsyncService.java:[5,49]
  error: cannot find symbol
    symbol:   class ListenableFutureCallback
    location: package org.springframework.util.concurrent
```

## What Changed {.what-changed}

<code>ListenableFuture</code>, <code>ListenableFutureCallback</code>, <code>ListenableFutureTask</code>, and related classes in <code>org.springframework.util.concurrent</code> were deleted from Spring Framework 7.0. All Spring APIs that previously returned <code>ListenableFuture</code> now return <code>java.util.concurrent.CompletableFuture</code>.

## Why {.why-changed}

<code>ListenableFuture</code> predated Java 8. Once <code>CompletableFuture</code> arrived in the JDK, maintaining a parallel abstraction added complexity with no benefit.

## The Fix {.diffs}

```diff-card
# // Async method return type
@@removed
import org.springframework.util.concurrent.ListenableFuture;

@Async
public ListenableFuture<String> fetchData() {
    return AsyncResult.forValue(doWork());
}
@@added
import java.util.concurrent.CompletableFuture;

@Async
public CompletableFuture<String> fetchData() {
    return CompletableFuture.completedFuture(doWork());
}
```

```diff-card
# // Callback-based usage
@@removed
ListenableFuture<String> future = asyncService.fetchData();
future.addCallback(
    result -> log.info("Success: {}", result),
    ex -> log.error("Failed", ex)
);
@@added
CompletableFuture<String> future = asyncService.fetchData();
future.whenComplete((result, ex) -> {
    if (ex != null) log.error("Failed", ex);
    else log.info("Success: {}", result);
});
```

## How To Fix {.fixes}

#### Replace with CompletableFuture.

Change return types from <code>ListenableFuture&lt;T&gt;</code> to <code>CompletableFuture&lt;T&gt;</code>. Replace <code>AsyncResult.forValue()</code> with <code>CompletableFuture.completedFuture()</code>. Replace <code>addCallback()</code> with <code>whenComplete()</code> or <code>thenAccept()</code>.

#### Use OpenRewrite.

The <code>org.openrewrite.java.spring.framework.MigrateSpringAssert</code> recipe set includes <code>ListenableFuture</code> migration.

## Scope Check {.scope-check}

Search for <code>ListenableFuture</code>, <code>ListenableFutureCallback</code>, and <code>AsyncResult</code>. Also check for <code>addCallback(</code> calls: these usage sites need rewriting even if the declaration is in a library.

## Watch Out {.watch-out}

- <code>AsyncResult</code> was also removed. If you used <code>new AsyncResult&lt;&gt;(value)</code> or <code>AsyncResult.forValue()</code>, both must change to <code>CompletableFuture.completedFuture()</code>.
- If you used <code>ListenableFuture</code> in a messaging listener (e.g., Spring Kafka's <code>KafkaTemplate.send()</code>), the return type changed upstream too. Check your Kafka and AMQP send-and-confirm patterns.

## Further Info {.further-info}

Deprecated in Spring Framework 6.0, removed in 7.0 (tracked in spring-framework#33808). Spring Kafka and Spring AMQP return types that previously used ListenableFuture changed too.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/listenable-future-removed)

- [Spring Framework ListenableFuture removal](https://github.com/spring-projects/spring-framework/issues/33808)

