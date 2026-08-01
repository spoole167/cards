---
id: maven-aot-plugin
tier: 1
tier_label: Won't Build
title: Maven Plugin Version Alignment for AOT
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: true
subsystem: core
no_module: true
no_module_reason: Requires spring-boot:process-aot goal; not a standard test execution
---

The Spring Boot Maven plugin's AOT processing goals changed in 4.0. A plugin version that lags behind the Spring Boot version fails the build with NoSuchMethodError.

## What You'll See {.error-output}

```error-output
$ mvn spring-boot:process-aot
[ERROR] Failed to execute goal org.springframework.boot:spring-boot-maven-plugin:4.0.0:process-aot
  on project my-service: Execution default of goal
  org.springframework.boot:spring-boot-maven-plugin:4.0.0:process-aot failed:
  An API incompatibility was encountered while executing
  org.springframework.boot:spring-boot-maven-plugin:4.0.0:process-aot:
  java.lang.NoSuchMethodError:
    'void org.springframework.aot.generate.GenerationContext.<init>(java.lang.ClassLoader)'
```

## What Changed {.what-changed}

The <code>spring-boot-maven-plugin</code> must match the Spring Boot version exactly. In 4.0 the AOT processing API changed: the <code>process-aot</code> and <code>process-test-aot</code> goals use new generation context methods. A plugin version mismatch causes <code>NoSuchMethodError</code> at build time.

## Why {.why-changed}

AOT processing was refactored for GraalVM 25 compatibility and to support the new Spring Framework 7.0 code generation model. The plugin version must be kept in lock-step with the framework.

## The Fix {.diffs}

```diff-card
# // pom.xml — plugin version
@@removed
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>3.4.1</version>
</plugin>
@@added
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>4.0.0</version>
</plugin>
```

## How To Fix {.fixes}

#### Align the plugin version with Spring Boot.

If you inherit from <code>spring-boot-starter-parent</code>, the plugin version is managed automatically: remove any explicit <code>&lt;version&gt;</code> tag. If you use a BOM without the parent POM, set the plugin version to <code>4.0.0</code> explicitly.

## Scope Check {.scope-check}

Search for <code>spring-boot-maven-plugin</code> across all POM files and check for explicit <code>&lt;version&gt;</code> tags. Multi-module projects need every module's POM checked.

## Watch Out {.watch-out}

- If you're not using native images, AOT processing is still enabled by default in 4.0 for startup optimisation. You may hit this error even if you never intentionally configured AOT.
- Parent POM version and plugin version must match. If your <code>&lt;parent&gt;</code> points to 4.0.0 but a plugin override still says 3.x, the build will fail with cryptic reflection errors.

## Further Info {.further-info}

AOT processing runs by default in Boot 4.0 even for non-native builds. Stale manual AOT <code>&lt;execution&gt;</code> blocks are a separate break: see maven-aot-execution-blocks. See also: graalvm-25.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

