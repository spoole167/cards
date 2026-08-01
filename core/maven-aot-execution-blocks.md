---
id: maven-aot-execution-blocks
tier: 1
tier_label: Won't Build
title: Manual AOT Execution Blocks Now Redundant
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
no_module: true
no_module_reason: Requires spring-boot:process-aot goal; not a standard test execution
---

The Boot 4.0 parent POM binds the AOT goals automatically. Explicit process-aot execution blocks left over from 3.x make the goal run twice, or run with stale configuration that no longer matches the plugin.

## What You'll See {.error-output}

```error-output
$ mvn package
[INFO] --- spring-boot-maven-plugin:4.0.0:process-aot (process-aot) @ my-service ---
[INFO] --- spring-boot-maven-plugin:4.0.0:process-aot (default-process-aot) @ my-service ---
...
[ERROR] Failed to execute goal ...:process-aot (process-aot) on project my-service:
  configuration parameter no longer supported by the 4.0 goal
---
Symptoms vary: duplicated AOT runs, longer builds, or a hard failure
when the old execution carries configuration the 4.0 goal rejects.
```

## What Changed {.what-changed}

In Boot 3.x, running AOT processing usually meant declaring an explicit <code>&lt;execution&gt;</code> block for the <code>process-aot</code> and <code>process-test-aot</code> goals. In 4.0 the parent POM binds these goals automatically, so a leftover manual block declares the goal a second time and applies configuration written for the 3.x goal.

## Why {.why-changed}

AOT processing runs by default in Boot 4.0, even for non-native builds, so the build lifecycle now owns the goal binding. Manual blocks were only ever a workaround for AOT being opt-in.

## The Fix {.diffs}

```diff-card
# // pom.xml — remove explicit AOT execution if using parent POM
@@removed
<execution>
    <id>process-aot</id>
    <goals><goal>process-aot</goal></goals>
</execution>
@@added
<!-- AOT goals are now bound automatically by the parent POM -->
```

## How To Fix {.fixes}

#### Delete manual AOT execution blocks.

Remove <code>&lt;execution&gt;</code> blocks for <code>process-aot</code> and <code>process-test-aot</code> from every POM that inherits the parent. Keep any non-AOT configuration by moving it to the plugin's top-level <code>&lt;configuration&gt;</code> section.

#### Not using the parent POM?

If you build from the BOM without <code>spring-boot-starter-parent</code>, you still own the goal binding. Keep the execution block but review its configuration against the 4.0 goal's parameters.

## Scope Check {.scope-check}

Search all POM files for <code>process-aot</code> and <code>process-test-aot</code>. Any hit inside an <code>&lt;execution&gt;</code> block in a module that inherits <code>spring-boot-starter-parent</code> is redundant.

## Watch Out {.watch-out}

- A duplicated goal that happens to succeed still doubles your AOT processing time; slow builds are the quiet version of this break.
- CI pipelines that call <code>mvn spring-boot:process-aot</code> directly are also redundant under the parent POM and can conflict with the automatic binding.

## Further Info {.further-info}

Split out from the plugin version alignment break: a project with a correctly managed plugin version can still carry stale execution blocks, and vice versa. See also: maven-aot-plugin, graalvm-25.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)
