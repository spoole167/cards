---
id: batch-joblauncher-removed
tier: 1
tier_label: Won't Build
title: Spring Batch JobStep.setJobLauncher Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: batch
---

JobStep.setJobLauncher replaced by setJobOperator. Fails to compile on Boot 4.0.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/BatchRenamingUsage.java:[18,19]
  error: cannot find symbol
    symbol:   method setJobLauncher(JobLauncher)
```

## What Changed {.what-changed}

<code>JobStep.setJobLauncher(JobLauncher)</code> has been replaced by <code>JobStep.setJobOperator(JobOperator)</code>.

## Why {.why-changed}

<code>setJobLauncher</code> was removed as part of the broader move away from <code>JobLauncher</code> toward <code>JobOperator</code> as the public API for job execution control.

## The Fix {.diffs}

```diff-card
# // JobStep launcher → operator
@@removed
jobStep.setJobLauncher(jobLauncher);
@@added
jobStep.setJobOperator(jobOperator);
```

## How To Fix {.fixes}

#### Replace setJobLauncher with setJobOperator.

Inject a <code>JobOperator</code> bean instead of <code>JobLauncher</code> and pass it to <code>setJobOperator()</code>. <code>JobOperator</code> is available as a Spring-managed bean when Spring Batch autoconfiguration is active.

## Scope Check {.scope-check}

Search for <code>setJobLauncher</code> across all Java/Kotlin sources. Affects custom job step configuration.

## Watch Out {.watch-out}

- <code>JobLauncher</code> itself still exists. Only its use in <code>JobStep</code> was removed; other usages are unaffected.

## Further Info {.further-info}

Verified in the same spring-break test module as the companion ChunkHandler rename: the two renames fail in the same compile run but are otherwise unrelated.

## Links {.footer-links}

- [spring-break module: batch-chunkhandler-renamed](https://github.com/spoole167/spring-break/tree/main/batch-chunkhandler-renamed)

- [Spring Batch 6.0 Migration Guide](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-6.0-Migration-Guide)

