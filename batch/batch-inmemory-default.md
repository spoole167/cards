---
id: batch-inmemory-default
tier: 2
tier_label: Won't Run
title: Spring Batch Now Uses In-Memory Job Repository by Default
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: batch
---

Spring Batch defaults to an in-memory job repository in Boot 4.0, so <code>BATCH_JOB_EXECUTION</code> and related tables are no longer created. Add <code>spring-boot-starter-batch-jdbc</code> to restore database persistence.

## What You'll See {.error-output}

```error-output
// Boot 3.5: BATCH_JOB_EXECUTION table created, job persisted.

// Boot 4.0 with only spring-boot-starter-batch:
// No schema created. Query against batch tables fails:
org.springframework.jdbc.BadSqlGrammarException:
  PreparedStatementCallback; bad SQL grammar
  [SELECT JOB_INSTANCE_ID, JOB_NAME from BATCH_JOB_INSTANCE ...];
  Table "BATCH_JOB_INSTANCE" not found

// Or job restart logic ignores execution history.
```

## What Changed {.what-changed}

Spring Boot 4.0 introduced <code>spring-boot-starter-batch-jdbc</code> as the explicit opt-in for database-backed job execution persistence. <code>spring-boot-starter-batch</code> now configures an in-memory <code>JobRepository</code> backed by a <code>ConcurrentHashMap</code>. The Batch schema DDL is no longer applied automatically unless the JDBC starter is on the classpath.

## Why {.why-changed}

Many Batch use cases (short-lived jobs, unit tests, development) do not need database persistence, yet the old default forced every Batch application to carry a datasource and manage schema migrations. The in-memory default is the right starting point; persistence is an explicit opt-in.

## The Fix {.diffs}

```diff-card
# // pom.xml — add the JDBC starter to restore database persistence
@@removed
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
@@added
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch-jdbc</artifactId>
</dependency>
```

## How To Fix {.fixes}

#### Add spring-boot-starter-batch-jdbc for database-backed persistence.

If your application relies on job execution history, restartability, or the Batch admin tables, add <code>spring-boot-starter-batch-jdbc</code> alongside <code>spring-boot-starter-batch</code>. This restores the Boot 3.5 behaviour.

#### Embrace the in-memory default if persistence isn't needed.

For stateless or ephemeral jobs (one-shot CLI jobs, test fixtures, data imports) the in-memory repository is correct and removes the need for a datasource. Just remove any schema initialisation configuration.

## Scope Check {.scope-check}

Check for code that queries Batch tables directly via JDBC, uses <code>JobExplorer</code> to inspect execution history, or relies on Batch restart semantics (same job parameters, incomplete executions). All of these require the JDBC starter.

## Watch Out {.watch-out}

- This interacts with the <code>batch-schema-change</code> break: Boot 4.0 also renamed the schema sequence from <code>BATCH_STEP_EXECUTION_SEQ</code> to <code>BATCH_STEP_EXECUTION_SEQUENCE</code>. If you add <code>spring-boot-starter-batch-jdbc</code>, make sure your schema migration handles both changes.

## Verify {.verify}

Batch jobs persist execution history to the database when spring-boot-starter-batch-jdbc is declared

## Further Info {.further-info}

In Boot 3.5, spring-boot-starter-batch with a JDBC datasource created the Batch schema and persisted job executions automatically. Code that relies on the Batch tables existing (auditing, restart logic, admin UI) breaks at runtime.

## Links {.footer-links}

- [spring-break module: batch-in-memory-default](https://github.com/spoole167/spring-break/tree/main/batch-in-memory-default)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

