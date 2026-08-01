---
id: hibernate-session-delete
tier: 1
tier_label: Won't Build
title: Session.delete() Removed
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: true
subsystem: hibernate
---

Hibernate 7.0 removed Session.delete(): use Session.remove() instead. Any direct Hibernate Session usage for deleting entities won't compile.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/repo/CustomRepoImpl.java:[28,16]
  error: cannot find symbol
    symbol:   method delete(com.example.model.Order)
    location: variable session of type org.hibernate.Session
```

## What Changed {.what-changed}

Hibernate 7.0 removed the legacy <code>Session.delete()</code> method. The JPA-standard <code>EntityManager.remove()</code> (which maps to <code>Session.remove()</code>) is the only way to delete entities. <code>Session.save()</code>, <code>Session.update()</code>, and <code>Session.saveOrUpdate()</code> were also removed.

## Why {.why-changed}

These methods were Hibernate-proprietary duplicates of the JPA standard API. Having two ways to do the same thing caused confusion about cascade behaviour and flush timing. The JPA methods have clearer semantics.

## The Fix {.diffs}

```diff-card
# // Deleting an entity
@@removed
session.delete(order);
@@added
session.remove(order);
```

```diff-card
# // Saving an entity
@@removed
session.save(newOrder);
@@added
session.persist(newOrder);
```

```diff-card
# // Updating an entity
@@removed
session.update(existingOrder);
@@added
session.merge(existingOrder);
```

```diff-card
# // Save-or-update pattern
@@removed
session.saveOrUpdate(order);
@@added
session.merge(order);
```

## How To Fix {.fixes}

#### Replace with JPA standard methods.

<code>delete()</code> becomes <code>remove()</code>, <code>save()</code> becomes <code>persist()</code>, <code>update()</code> becomes <code>merge()</code>, <code>saveOrUpdate()</code> becomes <code>merge()</code>.

#### Prefer EntityManager over Session.

If you're unwrapping the <code>Session</code> just to call these methods, consider using <code>EntityManager</code> directly. It has the same <code>persist()</code>, <code>merge()</code>, and <code>remove()</code> methods.

## Scope Check {.scope-check}

Search for <code>session.delete(</code>, <code>session.save(</code>, <code>session.update(</code>, and <code>session.saveOrUpdate(</code>. Also search for <code>Session.delete</code> in any utility or base repository classes.

## Watch Out {.watch-out}

- <code>merge()</code> returns the managed instance; <code>saveOrUpdate()</code> did not. If your code used the original reference after the call, switch to the instance <code>merge()</code> returns.
- <code>remove()</code> requires a managed entity. If you called <code>delete()</code> on detached entities, <code>merge()</code> or re-fetch them first. Otherwise you get an <code>IllegalArgumentException</code> at runtime.

## Further Info {.further-info}

Driven by Hibernate 7.0, upstream of Spring Boot 4.0. The methods were deprecated in Hibernate 6.0. See also: cascade-save-update, hibernate-dialect-removal.

## Links {.footer-links}

- [spring-break module: hibernate-session-delete](https://github.com/spoole167/spring-break/tree/main/hibernate-session-delete)

- [Hibernate 7 migration guide](https://docs.jboss.org/hibernate/orm/7.0/migration-guide/migration-guide.html)

