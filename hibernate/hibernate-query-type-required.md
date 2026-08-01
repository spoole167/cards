---
id: hibernate-query-type-required
tier: 2
tier_label: Won't Run
title: Hibernate Untyped Join Query Rejected at Runtime
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: hibernate
---

Hibernate 7 rejects untyped join queries with no explicit SELECT clause. A query that returned Object[] on Boot 3.5 throws SemanticException at runtime on Boot 4.0.

## What You'll See {.error-output}

```error-output
// Query without explicit result type
session.createQuery("from Product p join p.category c").getResultList();

// Boot 3.5 (Hibernate 6.x): runs fine, returns List<Object[]>

// Boot 4.0 (Hibernate 7.x): throws at runtime
org.hibernate.query.SemanticException:
  Query has no explicit SELECT and multiple query roots are defined.
  An explicit SELECT clause is required when multiple query roots exist
  or when the query contains a JOIN.
```

## What Changed {.what-changed}

Hibernate 7.0 tightened validation of HQL queries. A query like <code>from Product p join p.category c</code> has two implicit query roots (Product and the joined Category). Without an explicit <code>SELECT</code> clause or result type, the return type is ambiguous. Hibernate 6.x resolved this as <code>Object[]</code>; Hibernate 7.0 rejects it at parse time with a <code>SemanticException</code>.

## Why {.why-changed}

Silent return of <code>Object[]</code> hides intent and causes surprising <code>ClassCastException</code> at the point of use rather than at the query. A hard error forces explicit, clearer queries.

## The Fix {.diffs}

```diff-card
# // Add an explicit SELECT and result type
@@removed
session.createQuery("from Product p join p.category c")
       .getResultList();
@@added
session.createQuery(
    "SELECT p FROM Product p JOIN p.category c",
    Product.class
).getResultList();
```

```diff-card
# // Or use a typed tuple query
@@removed
session.createQuery("from Product p join p.category c")
       .getResultList();
@@added
session.createQuery(
    "SELECT p, c FROM Product p JOIN p.category c",
    Object[].class
).getResultList();
```

## How To Fix {.fixes}

#### Add explicit SELECT clause and result type (recommended).

All <code>Session.createQuery(String)</code> calls with join queries must include an explicit <code>SELECT</code> clause. Pass the expected result class as the second argument: <code>createQuery(hql, Entity.class)</code>.

#### Search for bare 'from' queries.

Grep for <code>createQuery("from</code> and <code>createQuery('from</code> in your codebase. Any query that starts with <code>from</code> and contains a join is at risk.

## Scope Check {.scope-check}

Search for <code>createQuery(</code> calls. JPQL queries via <code>EntityManager.createQuery(String, Class)</code> are also affected if they omit the <code>SELECT</code> clause with joins.

## Watch Out {.watch-out}

- The query compiles and runs on Boot 3.5 without any warning. The failure only surfaces on Boot 4.0 at the first call site, not at startup. Tests that exercise query paths are the only way to catch this before production.
- Simple queries without joins are unaffected: <code>from Product p</code> with a single root remains valid. Only multi-root or joined queries trigger the error.

## Verify {.verify}

HQL join queries without explicit SELECT or result type execute without SemanticException

## Further Info {.further-info}

Introduced in Hibernate 7.0, upstream of Spring Boot 4.0. Affects any application using Session.createQuery(String) with join queries and no result type.

## Links {.footer-links}

- [spring-break module: hibernate-query-type-required](https://github.com/spoole167/spring-break/tree/main/hibernate-query-type-required)

- [Hibernate 7.0 Migration Guide](https://docs.jboss.org/hibernate/orm/7.0/migration-guide/migration-guide.html)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

