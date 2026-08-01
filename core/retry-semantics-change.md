---
id: retry-semantics-change
tier: 3
tier_label: Wrong Results
title: maxAttempts Now Means Retries, Not Total Attempts
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: core
---

<code>@Retryable(maxAttempts = 3)</code> now performs 3 retries (4 total calls) instead of 3 total attempts. Every retry site silently gains an extra call.

## What You'll See {.error-output}

```error-output
// Before (Spring Boot 3.5): 3 total attempts
INFO  Calling payment service (attempt 1/3)
WARN  Payment failed, retrying...
INFO  Calling payment service (attempt 2/3)
WARN  Payment failed, retrying...
INFO  Calling payment service (attempt 3/3)
ERROR Payment failed after 3 attempts

// After (Spring Boot 4.0): 1 initial + 3 retries = 4 total
INFO  Calling payment service (attempt 1/4)
WARN  Payment failed, retrying...
INFO  Calling payment service (attempt 2/4)
WARN  Payment failed, retrying...
INFO  Calling payment service (attempt 3/4)
WARN  Payment failed, retrying...
INFO  Calling payment service (attempt 4/4)
ERROR Payment failed after 4 attempts

// Integration test failure
Expected invocation count: 3
  Actual invocation count: 4
```

## What Changed {.what-changed}

The <code>maxAttempts</code> parameter in <code>@Retryable</code> changed semantics. In Spring Retry 1.x (Boot 3.5) it meant total attempts including the initial call. In Boot 4.0 it means retries after the initial call. <code>maxAttempts = 3</code> now makes 4 calls.

## Why {.why-changed}

The old naming was misleading: "max attempts" of 3 producing only 2 retries was a constant source of off-by-one bugs. The parameter name now matches what it counts, the number of retries.

## The Fix {.diffs}

```diff-card
# // Fix: reduce maxAttempts by 1 to preserve old behaviour
@@removed
@Retryable(maxAttempts = 3)
@@added
@Retryable(maxAttempts = 2)  // 1 initial + 2 retries = 3 total
```

```diff-card
# // Or use the new explicit parameter
@@removed
@Retryable(maxAttempts = 3)
@@added
@Retryable(retries = 2)  // clearer: 2 retries after initial call
```

```diff-card
# // RetryTemplate configuration
@@removed
RetryTemplate template = RetryTemplate.builder()
    .maxAttempts(3)
    .build();
@@added
RetryTemplate template = RetryTemplate.builder()
    .maxAttempts(2)  // was 3 — now means retries, not total
    .build();
```

## How To Fix {.fixes}

#### Subtract 1 from every maxAttempts value.

Search for all <code>@Retryable</code> annotations and <code>RetryTemplate</code> configurations. Reduce each <code>maxAttempts</code> value by 1 to preserve the original total attempt count.

#### Audit retry budgets.

The extra attempt may be acceptable or even desirable. Review each retry site and decide whether the new count is correct. Document the intended behaviour either way.

## Scope Check {.scope-check}

Search for <code>@Retryable</code>, <code>RetryTemplate</code>, and any <code>maxAttempts</code> configuration in properties files. Every instance now makes one more call than before. For services with tight SLAs or rate-limited downstream APIs, this can cause cascading failures.

## Watch Out {.watch-out}

- The extra retry is invisible in logs unless you count carefully. Exponential backoff stretches the delay: <code>maxAttempts = 5</code> with 2-second backoff now takes 62 seconds to fail instead of 30.
- If your downstream service has rate limiting, the extra call per failure scenario may push you over the limit. Multiply the extra attempt by your error rate to estimate the impact.

## Verify {.verify}

Retried operations use correct backoff timing and max attempts

## Further Info {.further-info}

Driven by Spring Retry 2.x (upstream of Spring Boot 4.0). Spring Retry also left the Boot BOM in 4.0, so it needs an explicit version. See also: spring-retry-bom, retryable-transaction-order.

## Links {.footer-links}

- [spring-break module: retry-semantics-change](https://github.com/spoole167/spring-break/tree/main/retry-semantics-change)

- [Spring Framework Resilience docs](https://docs.spring.io/spring-framework/reference/core/resilience.html)

