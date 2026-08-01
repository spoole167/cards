---
id: hibernate-query-setorder-removed
tier: 1
tier_label: Won't Build
title: Hibernate SelectionQuery.setOrder() Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: hibernate
---

Hibernate's incubating SelectionQuery.setOrder() was removed in 7.0. Use an ORDER BY clause in HQL instead.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/HibernateQueryUsage.java:[14,15]
  error: cannot find symbol
    symbol:   method setOrder(java.util.List)
    location: interface SelectionQuery<Product>
```

## What Changed {.what-changed}

The <code>@Incubating</code> <code>setOrder(List&lt;Order&gt;)</code> method on <code>SelectionQuery</code> (and <code>Query</code>) was removed in Hibernate 7.0. It had been added in Hibernate 6.x as part of Jakarta Data integration work but was never promoted to stable API.

## Why {.why-changed}

The method was always experimental. Hibernate 7.0 completed the Jakarta Data integration under a more stable API surface and removed the incubating intermediate.

## The Fix {.diffs}

```diff-card
# // Programmatic order via removed setOrder()
@@removed
SelectionQuery<Product> query =
    session.createSelectionQuery("from Product", Product.class);
query.setOrder(List.of(Order.asc(Product.class, "name")));
```

```diff-card
# // Fix: inline ORDER BY in HQL
@@added
SelectionQuery<Product> query =
    session.createSelectionQuery("from Product order by name asc", Product.class);
```

## How To Fix {.fixes}

#### Move ordering into the HQL query string.

Add an <code>ORDER BY</code> clause directly to the HQL/JPQL string. This is the most portable approach and works on both 3.5 and 4.0.

#### Use CriteriaQuery with Order.

For fully programmatic ordering, use the JPA Criteria API: <code>criteriaQuery.orderBy(builder.asc(root.get("name")))</code>.

## Scope Check {.scope-check}

Search for <code>.setOrder(</code> on query objects across all repository and data-access classes. Rare outside code written to try the Hibernate 6.x incubating API.

## Watch Out {.watch-out}

- <code>org.hibernate.query.Order</code> (the parameter type) is also affected. If you imported it, that import can be removed once you stop using <code>setOrder</code>.

## Further Info {.further-info}

setOrder() only existed during the Hibernate 6.x window. Code that adopted it must move to HQL ORDER BY or the SelectionSpecification API.

## Links {.footer-links}

- [spring-break module: hibernate-query-setorder-removed](https://github.com/spoole167/spring-break/tree/main/hibernate-query-setorder-removed)

- [Hibernate 7.0 Migration Guide](https://docs.hibernate.org/orm/7.0/migration-guide/migration-guide.html)

