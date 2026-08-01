---
id: hibernate-processor-rename
tier: 1
tier_label: Won't Build
title: hibernate-jpamodelgen Renamed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: true
subsystem: hibernate
---

Hibernate renamed its annotation processor artifact from hibernate-jpamodelgen to hibernate-processor. Metamodel generation breaks.

## What You'll See {.error-output}

```error-output
$ mvn compile
[WARNING] The POM for org.hibernate.orm:hibernate-jpamodelgen:jar:7.0.0 is missing, no dependency
  information available
[ERROR] Failed to execute goal on project my-service: Could not resolve dependencies:
  Could not find artifact org.hibernate.orm:hibernate-jpamodelgen:jar:7.0.0
  in central (https://repo.maven.apache.org/maven2)
```

## What Changed {.what-changed}

Hibernate 7.0 (shipped with Spring Boot 4.0) renamed the annotation processor artifact from <code>org.hibernate.orm:hibernate-jpamodelgen</code> to <code>org.hibernate.orm:hibernate-processor</code>.

## Why {.why-changed}

The processor now validates HQL/JPQL queries at compile time and generates type-safe query methods as well as the JPA static metamodel. The new name reflects the broader scope.

## The Fix {.diffs}

```diff-card
# // pom.xml — annotation processor dependency
@@removed
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-jpamodelgen</artifactId>
    <scope>provided</scope>
</dependency>
@@added
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-processor</artifactId>
    <scope>provided</scope>
</dependency>
```

```diff-card
# // build.gradle — annotation processor
@@removed
annotationProcessor 'org.hibernate.orm:hibernate-jpamodelgen'
@@added
annotationProcessor 'org.hibernate.orm:hibernate-processor'
```

```diff-card
# // maven-compiler-plugin annotationProcessorPaths
@@removed
<path>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-jpamodelgen</artifactId>
</path>
@@added
<path>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-processor</artifactId>
</path>
```

## How To Fix {.fixes}

#### Rename the artifact.

Replace <code>hibernate-jpamodelgen</code> with <code>hibernate-processor</code> in your POM or Gradle build file. The Spring Boot BOM manages the version: change only the artifact ID.

#### Check compiler plugin configuration.

If you declared the processor in <code>maven-compiler-plugin</code>'s <code>annotationProcessorPaths</code>, update it there too.

## Scope Check {.scope-check}

Search for <code>hibernate-jpamodelgen</code> in all POM and Gradle files. Also check <code>annotationProcessorPaths</code> in <code>maven-compiler-plugin</code> configuration.

## Watch Out {.watch-out}

- The new <code>hibernate-processor</code> validates your HQL queries at compile time by default. Invalid queries that previously failed only at runtime now fail the build. That is a feature, but it can surface hidden bugs.
- If your IDE configures annotation processors separately (e.g., IntelliJ's settings), update that too. Otherwise the metamodel generates in Maven but not in the IDE.

## Further Info {.further-info}

Driven by Hibernate 7.0. See also: cascade-save-update, session-delete-removed.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/hibernate-processor-rename)

- [Hibernate 7 migration guide](https://docs.jboss.org/hibernate/orm/7.0/migration-guide/migration-guide.html)

