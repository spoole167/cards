---
id: batch-schema-change
tier: 2
tier_label: Won't Run
title: Spring Batch 6 Schema Sequence Rename
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: true
subsystem: batch
---

Spring Batch 6 renamed its database sequences (e.g., BATCH_JOB_SEQ became BATCH_JOB_INSTANCE_SEQ). Existing databases using the old names fail when Batch tries to generate IDs.

## What You'll See {.error-output}

```error-output
org.springframework.dao.InvalidDataAccessResourceUsageException:
  could not fetch next sequence value
...
Caused by: java.sql.SQLSyntaxErrorException:
  sequence "BATCH_JOB_INSTANCE_SEQ" does not exist
 at org.postgresql.core.v3.QueryExecutorImpl.receiveErrorResponse(...)
 ...
org.springframework.batch.core.launch.JobLaunchException:
  Could not launch job 'importCustomersJob'.
  Failed to create new JobInstance for job [importCustomersJob]
```

## What Changed {.what-changed}

Spring Batch 6 renamed the database sequences used for generating IDs: <code>BATCH_JOB_SEQ</code> became <code>BATCH_JOB_INSTANCE_SEQ</code>, <code>BATCH_JOB_EXECUTION_SEQ</code> stayed the same, and <code>BATCH_STEP_EXECUTION_SEQ</code> stayed the same. The metadata table structures also changed. Applications with existing Batch schema from Spring Batch 5 will fail when the framework tries to use the new sequence names.

## Why {.why-changed}

The rename aligns sequence names with the tables they serve, making the schema self-documenting. The old generic names like <code>BATCH_JOB_SEQ</code> were ambiguous about which table's IDs they generated.

## The Fix {.diffs}

```diff-card
# // SQL migration script — PostgreSQL
@@removed
-- Old sequences from Spring Batch 5
@@added
-- Rename sequences to match Spring Batch 6 expectations
ALTER SEQUENCE BATCH_JOB_SEQ RENAME TO BATCH_JOB_INSTANCE_SEQ;
```

```diff-card
# // SQL migration script — MySQL
@@removed
-- Old sequences from Spring Batch 5
@@added
-- MySQL uses tables for sequence emulation
RENAME TABLE BATCH_JOB_SEQ TO BATCH_JOB_INSTANCE_SEQ;
```

## How To Fix {.fixes}

#### Run a schema migration script.

Create a Flyway or Liquibase migration that renames the sequences. For PostgreSQL: <code>ALTER SEQUENCE BATCH_JOB_SEQ RENAME TO BATCH_JOB_INSTANCE_SEQ;</code>. For MySQL, rename the emulation tables. Apply this before deploying the upgraded application.

#### Let Spring Batch recreate the schema.

If your Batch job data is disposable (e.g., dev/staging), set <code>spring.batch.jdbc.initialize-schema=always</code> to let Spring Batch drop and recreate the metadata tables with the new names. Do not use this in production.

#### Use the migration SQL from Spring Batch.

The Spring Batch 6 migration guide provides official SQL scripts for each database platform. Use those as the basis for your migration rather than writing them by hand.

## Scope Check {.scope-check}

Check if your application uses <code>spring-boot-starter-batch</code> and has a persistent job repository (not in-memory). If you see <code>BATCH_JOB_INSTANCE</code>, <code>BATCH_JOB_EXECUTION</code>, or <code>BATCH_STEP_EXECUTION</code> tables in your database, you need this migration. In-memory job repositories (MapJobRepositoryFactoryBean) are not affected.

## Watch Out {.watch-out}

- Sequences are only part of the schema change. Column types and table structures also changed in Spring Batch 6. Check the official migration SQL scripts for the full set of changes.
- If you run multiple application versions against the same database during a rolling deploy, the old version will break when the sequences are renamed. Plan a maintenance window or use blue-green deployment.
- H2 in-memory databases used in tests auto-create the schema fresh each run, so tests may pass while production fails. Test against a real database with existing Batch tables.

## Verify {.verify}

Spring Batch job launches with no BATCH_ table not found errors

## Further Info {.further-info}

The sequence rename was announced in the Batch 6.0 milestone notes. See also: batch-listener-classes.

## Links {.footer-links}

- [spring-break module: batch-schema-change](https://github.com/spoole167/spring-break/tree/main/batch-schema-change)

- [Spring Batch 6 migration guide](https://github.com/spring-projects/spring-batch/wiki/Spring-Batch-6.0-Migration-Guide)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

