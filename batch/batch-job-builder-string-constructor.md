---
id: batch-job-builder-string-constructor
tier: 1
tier_label: Won't Build
title: Spring Batch JobBuilder(String) Constructor Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: batch
---

JobBuilder(String) is removed. JobRepository is now mandatory at construction: new JobBuilder("name", jobRepository).

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/JobBuilderUsage.java:[9,33]
  error: no suitable constructor found for JobBuilder(java.lang.String)
```

## What Changed {.what-changed}

<code>new JobBuilder(String name)</code> and the subsequent <code>.repository(JobRepository)</code> method have been removed. <code>JobRepository</code> is now mandatory at construction time: <code>new JobBuilder(String name, JobRepository repository)</code>. The same applies to <code>StepBuilder</code> and other builder classes.

## Why {.why-changed}

Spring Batch 6.0 made <code>JobRepository</code> non-optional throughout the framework to ensure jobs are always backed by a persistent or in-memory store. Deferring the repository assignment was a source of subtle bugs where jobs ran without persistence.

## The Fix {.diffs}

#### Before — repository set after construction. {.before}
```diff-card
new JobBuilder("myJob")
    .repository(jobRepository)
    .start(step)
    .build();
```
#### After — repository required at construction {.after}

```diff-card
new JobBuilder("myJob", jobRepository)
    .start(step)
    .build();
```

#### Before @Bean method pattern
```diff-card

@Bean
public Job myJob(Step step) {
    return new JobBuilder("myJob").start(step).build();
}

```

#### After @Bean method pattern

```diff-card
@Bean
public Job myJob(JobRepository jobRepository, Step step) {
    return new JobBuilder("myJob", jobRepository).start(step).build();
}
```

## How To Fix {.fixes}

#### Inject JobRepository and pass it to the constructor.

Add <code>JobRepository</code> as a parameter to your <code>@Bean</code> method (Spring will inject it) and pass it as the second argument to <code>new JobBuilder(...)</code>. The same change is needed for <code>StepBuilder</code> and any other builder that previously accepted only a name.

## Scope Check {.scope-check}

Search for <code>new JobBuilder(</code>, <code>new StepBuilder(</code>, and <code>.repository(</code> across all <code>@Configuration</code> classes that define Batch jobs and steps.

## Watch Out {.watch-out}

- The <code>.repository()</code> method is also gone, so adding the repository via the fluent chain won't compile either. Pass it in the constructor; there is no fallback.


## Further Info {.further-info}

Spring Batch 6.0 makes the JobRepository compulsory framework-wide; it can no longer be deferred or omitted anywhere.

## Links {.footer-links}

- [spring-break module: batch-job-builder-string-constructor](https://github.com/spoole167/spring-break/tree/main/batch-job-builder-string-constructor)

- [Spring Batch 6.0 Migration Guide](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-6.0-Migration-Guide)

	