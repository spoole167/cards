---
id: null-marked-default
tier: 3
tier_label: Wrong Results
title: '@NullMarked Default — IDE/Compiler Null Safety'
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: core
no_module: true
no_module_reason: 'Compile-time and IDE-only change: runtime behaviour is unchanged,
  so no JUnit assertion is meaningful'
---

Spring Framework 7 applies <code>@NullMarked</code> to every package. IDEs and null-checking tools now flag your existing null-passing code as warnings or errors.

## What You'll See {.error-output}

```error-output
// Your existing code — unchanged, worked fine in Boot 3.5
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
String name = ctx.getEnvironment().getProperty("app.name");
int length = name.length();  // ← IDE warning: name may be null

// IntelliJ inspection results after upgrade
WARNING: Method invocation 'length' may produce NullPointerException
  at MyService.java:42
WARNING: Passing 'null' argument to parameter annotated as @NonNull
  at MyConfig.java:18

// If using -Werror or NullAway/ErrorProne
$ mvn compile
[ERROR] MyService.java:[42,24] error: [NullAway] dereferencing expression
  name, which is @Nullable
[ERROR] MyConfig.java:[18,35] error: [NullAway] passing @Nullable parameter
  where @NonNull is required
```

## What Changed {.what-changed}

Spring Framework 7 added <code>@NullMarked</code> (from JSpecify) at the package level across the entire framework. This means all method parameters and return types are assumed <code>@NonNull</code> by default unless explicitly annotated <code>@Nullable</code>. IDEs, static analysis tools, and compile-time null checkers now see Spring's null contracts and flag violations.

## Why {.why-changed}

Formal null contracts let tools catch <code>NullPointerException</code> bugs at compile time instead of runtime. Kotlin interop also improves: Kotlin's type system can now correctly infer nullability from Spring APIs.

## The Fix {.diffs}

```diff-card
# // Handle nullable return from getProperty()
@@removed
String name = env.getProperty("app.name");
return name.toUpperCase();
@@added
String name = env.getProperty("app.name");
if (name == null) {
    throw new IllegalStateException("app.name not configured");
}
return name.toUpperCase();
```

```diff-card
# // Or use the non-null variant
@@removed
String name = env.getProperty("app.name");
@@added
String name = env.getRequiredProperty("app.name");
```

```diff-card
# // Suppress warnings during migration
@@removed
// (no null warnings in Boot 3.5)
@@added
// Temporarily suppress in build.gradle
tasks.withType(JavaCompile) {
    options.compilerArgs += ['-Xlint:-nullness']
}
```

## How To Fix {.fixes}

#### Fix null handling in your code (recommended).

Follow the IDE warnings and add null checks, use <code>Optional</code>, or switch to non-null API variants like <code>getRequiredProperty()</code>. These are real NPE bugs that were always there; the framework now surfaces them.

#### Suppress during migration.

If the volume of warnings is too high to fix at once, suppress null-checking warnings at the compiler level and create a backlog to address them incrementally. Don't leave the suppression in place permanently.

## Scope Check {.scope-check}

Open your project in IntelliJ or VS Code with Java support and look at the new inspection warnings. Any code that passes <code>null</code> to a Spring API parameter, or uses a Spring API return value without a null check, will be flagged. The volume depends on how much Spring API surface your code touches directly.

## Watch Out {.watch-out}

- If your CI pipeline uses <code>-Werror</code>, NullAway, ErrorProne, or Checker Framework, the new null annotations turn warnings into build failures. The framework's null contracts are now visible to your tools; unchanged code becomes a broken build.
- Kotlin projects are affected differently. Kotlin's compiler uses these annotations to infer platform types. A Spring method that returned <code>String!</code> (platform type, nullable unknown) now returns <code>String</code> (non-null) or <code>String?</code> (nullable). Kotlin code that didn't handle nullability may now fail to compile.

## Verify {.verify}

No new NullPointerException at boundaries where nulls were OK before

## Further Info {.further-info}

Runtime behaviour is unchanged: the impact lands entirely in compilers, IDEs, and static analysis tools.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [JSpecify @NullMarked](https://jspecify.dev/docs/api/org/jspecify/annotations/NullMarked)

