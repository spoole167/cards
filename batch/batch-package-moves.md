---
id: batch-package-moves
tier: 1
tier_label: Won't Build
title: Spring Batch Core Package Relocations
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: batch
---

Core Spring Batch domain classes (Job, JobExecution, etc.) moved from org.springframework.batch.core to org.springframework.batch.core.job.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/BatchCoreUsage.java:[3,44]
  error: package org.springframework.batch.core does not contain class Job
[ERROR] /src/main/java/com/example/BatchCoreUsage.java:[4,44]
  error: package org.springframework.batch.core does not contain class JobExecution
```

## What Changed {.what-changed}

Core domain classes have moved from <code>org.springframework.batch.core</code> to subpackages, primarily <code>org.springframework.batch.core.job</code>. Affected classes include <code>Job</code>, <code>JobExecution</code>, <code>JobInstance</code>, and <code>JobParameters</code> among others.

## Why {.why-changed}

Spring Batch 6.0 redesigned the domain model to give each concern its own subpackage and make the module boundaries clear.

## The Fix {.diffs}

```diff-card
# // Core domain imports
@@removed
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobExecution;
import org.springframework.batch.core.JobInstance;
import org.springframework.batch.core.JobParameters;
@@added
import org.springframework.batch.core.job.Job;
import org.springframework.batch.core.job.JobExecution;
import org.springframework.batch.core.job.JobInstance;
import org.springframework.batch.core.job.JobParameters;
```

## How To Fix {.fixes}

#### Update imports to include the job subpackage.

Add <code>.job</code> to the package path for the affected classes. This is a mechanical find-and-replace: <code>org.springframework.batch.core.Job</code> becomes <code>org.springframework.batch.core.job.Job</code>, and so on. Check the Spring Batch 6.0 Migration Guide for the complete list of moved classes.

## Scope Check {.scope-check}

Search for <code>import org.springframework.batch.core.</code> across all Java/Kotlin sources. Any import that resolves directly to a class in the root <code>core</code> package is likely affected.

## Watch Out {.watch-out}

- Not all classes in <code>org.springframework.batch.core</code> moved. Some remain at the root level. Verify each import against the 6.0 Javadoc rather than bulk-replacing all root-level imports.

## Further Info {.further-info}

The classes are unchanged; only their packages moved. Expect a large batch of import fixes if your code uses Batch domain types directly.

## Links {.footer-links}

- [spring-break module: batch-package-moves](https://github.com/spoole167/spring-break/tree/main/batch-package-moves)

- [Spring Batch 6.0 Migration Guide](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-6.0-Migration-Guide)

