---
id: hibernate-cascade-removal
tier: 2
tier_label: Won't Run
title: CascadeType.SAVE_UPDATE Removed in Hibernate 7
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: true
subsystem: hibernate
---

Hibernate 7 removed the proprietary CascadeType.SAVE_UPDATE. Entities using this cascade type fail at startup or throw MappingException when the SessionFactory is built.

## What You'll See {.error-output}

```error-output
org.springframework.beans.factory.BeanCreationException:
  Error creating bean with name 'entityManagerFactory'
...
Caused by: org.hibernate.MappingException:
  Unrecognized legacy cascade style: save-update
 at org.hibernate.boot.model.internal.EntityBinder.bindEntityAnnotation(...)
 at org.hibernate.boot.model.internal.AnnotationBinder.bindClass(...)
 ...
APPLICATION FAILED TO START
---
Description:
Failed to initialize JPA EntityManagerFactory:
Unrecognized legacy cascade style: save-update
```

## What Changed {.what-changed}

<code>org.hibernate.annotations.CascadeType.SAVE_UPDATE</code> was a Hibernate-specific extension to JPA cascade types. It triggered cascade behaviour on Hibernate's proprietary <code>Session.saveOrUpdate()</code> method. Hibernate 7 removed both <code>saveOrUpdate()</code> and the corresponding cascade type. Code using the Hibernate-specific <code>@Cascade</code> annotation with <code>SAVE_UPDATE</code> breaks at entity mapping time.

## Why {.why-changed}

The <code>saveOrUpdate()</code> method was deprecated in Hibernate 6 as part of aligning with the JPA <code>EntityManager</code> API. Hibernate 7 completed the removal. The standard JPA <code>CascadeType.PERSIST</code> and <code>CascadeType.MERGE</code> cover the same use cases without vendor lock-in.

## The Fix {.diffs}

```diff-card
# // Using Hibernate @Cascade annotation
@@removed
import org.hibernate.annotations.Cascade;
import org.hibernate.annotations.CascadeType;

@OneToMany(mappedBy = "order")
@Cascade(CascadeType.SAVE_UPDATE)
private List<OrderItem> items;
@@added
@OneToMany(mappedBy = "order", cascade = {
    javax.persistence.CascadeType.PERSIST,
    javax.persistence.CascadeType.MERGE
})
private List<OrderItem> items;
```

```diff-card
# // Using JPA cascade with Hibernate extras
@@removed
@OneToMany(mappedBy = "parent")
@Cascade({CascadeType.SAVE_UPDATE, CascadeType.DELETE})
private Set<Child> children;
@@added
@OneToMany(mappedBy = "parent", cascade = {
    CascadeType.PERSIST, CascadeType.MERGE, CascadeType.REMOVE
})
private Set<Child> children;
```

## How To Fix {.fixes}

#### Replace with JPA standard cascades.

Swap <code>CascadeType.SAVE_UPDATE</code> for the combination of <code>CascadeType.PERSIST</code> and <code>CascadeType.MERGE</code>. These two JPA-standard cascades cover the same persist-or-update behaviour that <code>SAVE_UPDATE</code> provided.

#### Remove @Cascade if redundant.

If your entity already has <code>cascade = CascadeType.ALL</code> on the JPA annotation, the Hibernate <code>@Cascade</code> is redundant. Delete it entirely.

## Scope Check {.scope-check}

Search for <code>CascadeType.SAVE_UPDATE</code> and <code>@Cascade</code> (the Hibernate annotation, not JPA). Every hit is a mapping that will fail at startup. Also search for <code>session.saveOrUpdate(</code>; that method is gone too.

## Watch Out {.watch-out}

- <code>CascadeType.PERSIST</code> alone does not cover the "update existing detached entity" case. You need both <code>PERSIST</code> and <code>MERGE</code> to match the old <code>SAVE_UPDATE</code> behaviour.
- If you were using <code>Session.saveOrUpdate()</code> directly in DAO code, that method is also removed. Replace with <code>entityManager.merge()</code> or <code>entityManager.persist()</code>.
- Some code generators and legacy mapping tools emit <code>SAVE_UPDATE</code> by default. Re-run generation after upgrading.

## Further Info {.further-info}

Arrives via spring-boot-starter-data-jpa, which pulls in Hibernate 7.0. See also: session-delete-removed, hibernate-dialect-removal.

## Links {.footer-links}

- [spring-break module: hibernate-cascade-removal](https://github.com/spoole167/spring-break/tree/main/hibernate-cascade-removal)

- [Hibernate 7 migration guide](https://docs.jboss.org/hibernate/orm/7.0/migration-guide/migration-guide.html)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

