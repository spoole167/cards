---
id: batch-listener-classes
tier: 1
tier_label: Won't Build
title: Batch Listener Base Classes Removed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: true
subsystem: batch
---

Spring Batch 6.0 removed the abstract listener base classes like JobExecutionListenerSupport and StepExecutionListenerSupport.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/batch/JobCompletionListener.java:[4,52]
  error: cannot find symbol
    symbol:   class JobExecutionListenerSupport
    location: package org.springframework.batch.core.listener
[ERROR] /src/main/java/com/example/batch/StepLoggingListener.java:[4,52]
  error: cannot find symbol
    symbol:   class StepExecutionListenerSupport
```

## What Changed {.what-changed}

Spring Batch 6.0 deleted the <code>*ListenerSupport</code> abstract classes: <code>JobExecutionListenerSupport</code>, <code>StepExecutionListenerSupport</code>, <code>ChunkListenerSupport</code>, <code>ItemReadListenerSupport</code>, and others. Listeners must now implement the interfaces directly, which have had default methods since Batch 5.0.

## Why {.why-changed}

Java 8 default methods made the support classes unnecessary: the interfaces provide no-op defaults for every callback, so there's nothing left to inherit.

## The Fix {.diffs}

```diff-card
# // Job listener
@@removed
import org.springframework.batch.core.listener.JobExecutionListenerSupport;

public class JobCompletionListener extends JobExecutionListenerSupport {
    @Override
    public void afterJob(JobExecution jobExecution) {
        log.info("Job completed: {}", jobExecution.getStatus());
    }
}
@@added
import org.springframework.batch.core.JobExecutionListener;

public class JobCompletionListener implements JobExecutionListener {
    @Override
    public void afterJob(JobExecution jobExecution) {
        log.info("Job completed: {}", jobExecution.getStatus());
    }
}
```

```diff-card
# // Step listener
@@removed
import org.springframework.batch.core.listener.StepExecutionListenerSupport;

public class StepLogger extends StepExecutionListenerSupport {
@@added
import org.springframework.batch.core.StepExecutionListener;

public class StepLogger implements StepExecutionListener {
```

## How To Fix {.fixes}

#### Change extends to implements.

Replace <code>extends JobExecutionListenerSupport</code> with <code>implements JobExecutionListener</code>. Same for all other <code>*ListenerSupport</code> classes. The interfaces have default methods, so you only need to override the callbacks you use.

#### Use @BeforeJob / @AfterJob annotations.

Spring Batch also supports annotation-based listeners. Annotate methods with <code>@BeforeJob</code>, <code>@AfterJob</code>, <code>@BeforeStep</code>, <code>@AfterStep</code> etc. on any POJO; no interface needed.

## Scope Check {.scope-check}

Search for <code>ListenerSupport</code> and <code>extends.*ListenerSupport</code> across your batch job classes. Common hits include <code>JobExecutionListenerSupport</code>, <code>StepExecutionListenerSupport</code>, and <code>ChunkListenerSupport</code>.

## Watch Out {.watch-out}

- If your listener extends a support class and calls <code>super.afterJob()</code> or <code>super.beforeStep()</code>, remove those super calls. The interface default methods are no-ops, but calling <code>super</code> on an interface method looks wrong and may confuse static analysis tools.
- The <code>ItemWriter</code> and <code>ItemProcessor</code> interfaces also changed in Batch 6.0. If you're fixing listeners, audit the whole batch configuration in the same pass.

## Further Info {.further-info}

The *ListenerSupport classes were deprecated in Spring Batch 5.0 when the interfaces gained default methods. See also: batch-schema-rename.

## Links {.footer-links}

- [spring-break module: batch-job-serialisation](https://github.com/spoole167/spring-break/tree/main/batch-listener-classes)

- [Spring Batch 6 migration guide](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-6.0-Migration-Guide)

