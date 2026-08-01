---
id: hibernate-dialect-removal
tier: 2
tier_label: Won't Run
title: Version-Specific Hibernate Dialects Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: true
subsystem: hibernate
---

Hibernate 7 removed version-specific dialect classes like MySQL57Dialect and PostgreSQL95Dialect. Applications that explicitly configure a dialect fail at startup with ClassNotFoundException.

## What You'll See {.error-output}

```error-output
org.springframework.beans.factory.BeanCreationException:
  Error creating bean with name 'entityManagerFactory'
...
Caused by: java.lang.ClassNotFoundException:
  org.hibernate.dialect.MySQL57Dialect
 at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:641)
 at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:188)
 ...
APPLICATION FAILED TO START
---
Description:
Failed to configure a DataSource: Hibernate dialect
'org.hibernate.dialect.MySQL57Dialect' could not be found.
```

## What Changed {.what-changed}

Hibernate 7 removed all version-specific dialect classes (<code>MySQL57Dialect</code>, <code>MySQL8Dialect</code>, <code>PostgreSQL95Dialect</code>, <code>PostgreSQL10Dialect</code>, <code>Oracle12cDialect</code>, <code>SQLServer2016Dialect</code>, etc.). Only the base dialect classes remain (<code>MySQLDialect</code>, <code>PostgreSQLDialect</code>, <code>OracleDialect</code>, <code>SQLServerDialect</code>). Hibernate now auto-detects the database version from the JDBC connection and configures features accordingly.

## Why {.why-changed}

A dialect class per database version created a combinatorial explosion. Runtime detection removes the maintenance burden and the risk of choosing the wrong class.

## The Fix {.diffs}

```diff-card
# // application.properties
@@removed
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL57Dialect
@@added
# Remove explicit dialect — Hibernate auto-detects from JDBC connection
```

```diff-card
# // application.properties — if you must be explicit
@@removed
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQL95Dialect
@@added
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```

```diff-card
# // persistence.xml
@@removed
<property name="hibernate.dialect" value="org.hibernate.dialect.Oracle12cDialect"/>
@@added
<property name="hibernate.dialect" value="org.hibernate.dialect.OracleDialect"/>
```

## How To Fix {.fixes}

#### Remove explicit dialect (recommended).

Delete the <code>spring.jpa.properties.hibernate.dialect</code> property entirely. Hibernate 7 detects the correct dialect from the JDBC connection metadata.

#### Switch to base dialect class.

If you need an explicit dialect for testing or CI (e.g., H2 in-memory), use the unversioned base class: <code>MySQLDialect</code>, <code>PostgreSQLDialect</code>, <code>OracleDialect</code>, or <code>SQLServerDialect</code>.

## Scope Check {.scope-check}

Search for <code>hibernate.dialect</code> in <code>application.properties</code>, <code>application.yml</code>, <code>persistence.xml</code>, and any programmatic <code>LocalContainerEntityManagerFactoryBean</code> configuration. Also check test configurations and Docker Compose profiles.

## Watch Out {.watch-out}

- If you use Testcontainers or H2 for tests, those test configs often set an explicit dialect. Check <code>src/test/resources</code> too.
- Some connection pools and frameworks set the dialect programmatically. Search Java code for <code>setProperty("hibernate.dialect"</code> too.
- The auto-detection requires an active JDBC connection at startup. If your application uses lazy datasource initialisation, you may still need an explicit base dialect class.

## Verify {.verify}

App starts with no DialectResolutionInfoMissingException

## Further Info {.further-info}

Driven by Hibernate 7.0, upstream of Spring Boot 4.0. The classes were deprecated in Hibernate 6.0. Affects spring-boot-starter-data-jpa consumers. See also: hibernate-date-types, cascade-save-update.

## Links {.footer-links}

- [spring-break module: hibernate-dialect-removal](https://github.com/spoole167/spring-break/tree/main/hibernate-dialect-removal)

- [Hibernate 7 migration guide](https://docs.jboss.org/hibernate/orm/7.0/migration-guide/migration-guide.html)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

