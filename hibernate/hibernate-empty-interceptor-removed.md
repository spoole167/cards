---
id: hibernate-empty-interceptor-removed
tier: 1
tier_label: Won't Build
title: Hibernate EmptyInterceptor Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: hibernate
---

Hibernate's EmptyInterceptor is gone. Implement Interceptor directly: it has default methods for everything you don't need to override.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/MyHibernateInterceptor.java:[5,43]
  error: cannot find symbol
    symbol: class EmptyInterceptor
    location: package org.hibernate
```

## What Changed {.what-changed}

<code>org.hibernate.EmptyInterceptor</code> has been removed. The <code>org.hibernate.Interceptor</code> interface now has default (no-op) implementations for all its methods, so the abstract base class serves no purpose.

## Why {.why-changed}

When Hibernate 6.0 added default methods to <code>Interceptor</code>, <code>EmptyInterceptor</code> became redundant. Hibernate 7.0 removed it as part of the overall API cleanup.

## The Fix {.diffs}

```diff-card
# // Before — extending EmptyInterceptor
@@removed
import org.hibernate.EmptyInterceptor;

public class MyInterceptor extends EmptyInterceptor {
    @Override
    public boolean onLoad(Object entity, Object id,
            Object[] state, String[] propertyNames, Type[] types) {
        // custom logic
        return false;
    }
}
```

```diff-card
# // After — implementing Interceptor directly
@@added
import org.hibernate.Interceptor;

public class MyInterceptor implements Interceptor {
    @Override
    public boolean onLoad(Object entity, Object id,
            Object[] state, String[] propertyNames, Type[] types) {
        // custom logic
        return false;
    }
    // All other methods have default no-op implementations
}
```

## How To Fix {.fixes}

#### Implement Interceptor directly.

Replace <code>extends EmptyInterceptor</code> with <code>implements Interceptor</code> and remove the <code>EmptyInterceptor</code> import. Override only the methods you use.

## Scope Check {.scope-check}

Search for <code>EmptyInterceptor</code> and <code>extends EmptyInterceptor</code> across all Java/Kotlin sources.

## Watch Out {.watch-out}

- Some method signatures on <code>Interceptor</code> changed in Hibernate 6.x (e.g., <code>Serializable id</code> became <code>Object id</code>). Verify your overridden signatures match exactly. A mismatch compiles as a new overload and Hibernate never calls it.

## Further Info {.further-info}

EmptyInterceptor was a convenience class giving no-op implementations of all Interceptor methods. Deprecated in Hibernate 6.0, removed in 7.0.

## Links {.footer-links}

- [spring-break module: hibernate-empty-interceptor-removed](https://github.com/spoole167/spring-break/tree/main/hibernate-empty-interceptor-removed)

- [Hibernate 7.0 Migration Guide](https://docs.hibernate.org/orm/7.0/migration-guide/migration-guide.html)

