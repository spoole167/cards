---
id: retryable-transaction-order
tier: 3
tier_label: Wrong Results
title: '@Retryable + @Transactional AOP Ordering Change'
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: core
---

The default AOP advice ordering flipped: retry now wraps outside the transaction. A retried method gets a fresh transaction each attempt instead of retrying inside the same one.

## What You'll See {.error-output}

```error-output
// Before (Spring Boot 3.5) — retry inside transaction
DEBUG Opening transaction
DEBUG  Attempt 1: inserting order
WARN   Optimistic lock failure, retrying...
DEBUG  Attempt 2: inserting order
DEBUG  Attempt 2: success
DEBUG Committing transaction (1 commit total)

// After (Spring Boot 4.0) — retry outside transaction
DEBUG Opening transaction #1
DEBUG  Attempt 1: inserting order
WARN   Optimistic lock failure
DEBUG Rolling back transaction #1
DEBUG Opening transaction #2
DEBUG  Attempt 2: inserting order
DEBUG  Attempt 2: success
DEBUG Committing transaction #2

// Symptom: partial writes visible between retries
// Symptom: retry sees stale data from previous transaction
```

## What Changed {.what-changed}

The default ordering of Spring AOP advice changed so that <code>@Retryable</code> has higher precedence (runs first) than <code>@Transactional</code>. The retry logic now wraps the transaction, so each retry attempt runs in its own transaction.

## Why {.why-changed}

Retrying outside the transaction is safer: a failed transaction rolls back fully before the next attempt, avoiding stale persistence context and dirty session state. The old default caused subtle Hibernate session corruption when retrying inside a failed transaction.

## The Fix {.diffs}

```diff-card
# // Force retry inside transaction (old behaviour)
@@removed
@Retryable(maxAttempts = 3)
@Transactional
public void placeOrder(Order order) {
@@added
@Retryable(maxAttempts = 3)
@Transactional
@Order(Ordered.HIGHEST_PRECEDENCE)  // force tx to wrap retry
public void placeOrder(Order order) {
```

```diff-card
# // Or set global ordering in configuration
@@removed
# (default ordering in Boot 3.5: tx outside, retry inside)
@@added
@EnableRetry(order = Ordered.LOWEST_PRECEDENCE)
// Ensures retry runs inside the transaction
```

```diff-card
# // Best practice: separate the concerns
@@removed
@Retryable @Transactional
public void placeOrder(Order order) { ... }
@@added
// Outer bean: handles retry
@Retryable(maxAttempts = 3)
public void placeOrderWithRetry(Order order) {
    orderService.placeOrder(order);
}
// Inner bean: handles transaction
@Transactional
public void placeOrder(Order order) { ... }
```

## How To Fix {.fixes}

#### Separate retry and transaction into different beans (recommended).

Move <code>@Retryable</code> to an outer service and <code>@Transactional</code> to an inner service. This makes the ordering explicit, avoids AOP precedence surprises, and works the same regardless of framework defaults.

#### Set explicit advice ordering.

Use <code>@EnableRetry(order = ...)</code> and <code>@EnableTransactionManagement(order = ...)</code> to explicitly control which advice runs first. Document the intended order.

## Scope Check {.scope-check}

Search for any method annotated with both <code>@Retryable</code> and <code>@Transactional</code>. Each one has silently changed its transactional behaviour. Also check for <code>@Retryable</code> methods that call <code>@Transactional</code> methods on the same bean (self-invocation doesn't go through the proxy).

## Watch Out {.watch-out}

- If your retry logic depends on reading data written earlier in the same transaction, that data is now rolled back before the retry. The second attempt starts with a clean slate, which may cause duplicate inserts or missing context.
- Optimistic locking exceptions (<code>OptimisticLockException</code>) are now handled differently. Previously the retry happened inside the same persistence context. Now each retry gets a fresh EntityManager, which is usually better but changes the behaviour.

## Verify {.verify}

Retried transactional methods execute retries outside the transaction

## Further Info {.further-info}

Driven by a Spring Framework 7.0 change in default AOP advice ordering. See also: retry-semantics, spring-retry-bom.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/retryable-transaction-order)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

