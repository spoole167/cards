---
id: batch-job-serialisation
tier: 1
tier_label: Won't Build
title: Spring Batch 6 Listener Support Classes Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: batch
---

<code>JobExecutionListenerSupport</code>, <code>StepExecutionListenerSupport</code>, and <code>ChunkListenerSupport</code> are removed in Spring Batch 6.0. Code extending them fails to compile on Boot 4.0 with "cannot find symbol".

## What You'll See {.error-output}

```error-output
// Boot 3.5 (Batch 5.x): classes present, deprecated since 5.0; compiles fine.

// Boot 4.0 (Batch 6.0): classes removed; compilation fails.
$ mvn clean compile
[ERROR] COMPILATION ERROR :
[ERROR] MyJobListener.java:[4,47] cannot find symbol
    symbol: class JobExecutionListenerSupport
[INFO] BUILD FAILURE
```

## What Changed {.what-changed}

Spring Batch 6.0 (shipped with Spring Boot 4.0) removed three convenience base classes that had been deprecated since Batch 5.0: <code>JobExecutionListenerSupport</code>, <code>StepExecutionListenerSupport</code>, and <code>ChunkListenerSupport</code>. These classes provided empty default implementations of listener methods.

## Why {.why-changed}

Java 8 interface default methods eliminate the need for adapter-style base classes. Batch 5.0 deprecated them with that rationale; Batch 6.0 follows through with removal. Implementing the listener interface directly is both more explicit and compatible with composition rather than inheritance.

## The Fix {.diffs}

```diff-card
# // Replace base class with direct interface implementation
@@removed
public class MyJobListener extends JobExecutionListenerSupport {
    @Override
    public void afterJob(JobExecution jobExecution) {
        // only override what you need
    }
}
@@added
public class MyJobListener implements JobExecutionListener {
    @Override
    public void afterJob(JobExecution jobExecution) {
        // implement the interface directly
    }
}
```

## How To Fix {.fixes}

#### Implement the listener interface directly.

Replace <code>extends JobExecutionListenerSupport</code> with <code>implements JobExecutionListener</code>. Do the same for <code>StepExecutionListenerSupport</code> → <code>StepExecutionListener</code> and <code>ChunkListenerSupport</code> → <code>ChunkListener</code>. Implement only the methods you use: the rest have default no-op implementations on the interface.

## Scope Check {.scope-check}

Search for <code>extends JobExecutionListenerSupport</code>, <code>extends StepExecutionListenerSupport</code>, and <code>extends ChunkListenerSupport</code> in your source. Also check Spring XML configuration files and any reflection-based lookups for these class names.

## Watch Out {.watch-out}

- The compiler only catches direct imports and <code>extends</code> clauses. Listeners referenced by class name in Spring XML config or loaded via reflection bypass the compiler and fail only when that code path runs. Search configuration files as well as source.

## Further Info {.further-info}

Complements batch-listener-classes: both modules demonstrate the same underlying Batch 6.0 removal.

## Links {.footer-links}

- [spring-break module: batch-job-serialisation](https://github.com/spoole167/spring-break/tree/main/batch-job-serialisation)

- [Spring Batch 6.0 Migration Guide](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-6.0-Migration-Guide)

