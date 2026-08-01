---
id: hibernate-where-orderby-removed
tier: 1
tier_label: Won't Build
title: Hibernate @Where and @OrderBy Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: hibernate
---

Hibernate's @Where and @OrderBy are removed. Replace with @SQLRestriction and @SQLOrder.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/User.java:[5,39]
  error: cannot find symbol
    symbol: class Where
    location: package org.hibernate.annotations
[ERROR] /src/main/java/com/example/User.java:[6,39]
  error: cannot find symbol
    symbol: class OrderBy
    location: package org.hibernate.annotations
```

## What Changed {.what-changed}

<code>@org.hibernate.annotations.Where</code> and <code>@org.hibernate.annotations.OrderBy</code> have been removed. The replacements are <code>@org.hibernate.annotations.SQLRestriction</code> and <code>@org.hibernate.annotations.SQLOrder</code>, which take the same SQL fragment arguments.

## Why {.why-changed}

The new annotation names make it explicit that the argument is a raw SQL fragment, not a JPQL or HQL expression. This avoids confusion with the JPA <code>@OrderBy</code> (which takes attribute names) and with Hibernate's HQL-level features.

## The Fix {.diffs}

```diff-card
# // Entity annotations
@@removed
import org.hibernate.annotations.Where;
import org.hibernate.annotations.OrderBy;

@OneToMany
@Where(clause = "deleted = false")
@OrderBy(clause = "name desc")
private List<Post> posts;
@@added
import org.hibernate.annotations.SQLRestriction;
import org.hibernate.annotations.SQLOrder;

@OneToMany
@SQLRestriction("deleted = false")
@SQLOrder("name desc")
private List<Post> posts;
```

## How To Fix {.fixes}

#### Replace @Where with @SQLRestriction.

Change the annotation name and attribute: <code>@Where(clause = "...")</code> becomes <code>@SQLRestriction("...")</code>. The SQL fragment is the same.

#### Replace @OrderBy with @SQLOrder.

Change the annotation name and attribute: <code>@OrderBy(clause = "...")</code> becomes <code>@SQLOrder("...")</code>.

## Scope Check {.scope-check}

Search for <code>@Where</code> and <code>@OrderBy</code> from the <code>org.hibernate.annotations</code> package across all entity classes and embeddables. Also check <code>@WhereJoinTable</code> if used.

## Watch Out {.watch-out}

- Only Hibernate's <code>@OrderBy</code> needs replacing. JPA's <code>@jakarta.persistence.OrderBy</code> takes attribute names, not SQL, and is unaffected.

## Further Info {.further-info}

@SQLRestriction and @SQLOrder arrived in Hibernate 6.3; the old annotations were deprecated then and removed in 7.0.

## Links {.footer-links}

- [spring-break module: hibernate-where-orderby-removed](https://github.com/spoole167/spring-break/tree/main/hibernate-where-orderby-removed)

- [Hibernate 7.0 Migration Guide](https://docs.hibernate.org/orm/7.0/migration-guide/migration-guide.html)

