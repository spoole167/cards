---
id: classic-uber-jar-loader-removed
tier: 1
tier_label: Won't Build
title: Classic Uber-Jar Loader Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
no_module: true
no_module_reason: |
  The CLASSIC loader mode was a Maven/Gradle plugin setting, not a Java import or API. A Maven test module cannot exercise the jar packaging pipeline in a way that isolates this configuration failure from other build changes.
---

The <code>CLASSIC</code> uber-jar loader option is removed in Boot 4.0: the Maven/Gradle plugin fails the build if it is configured. Remove the setting.

## What You'll See {.error-output}

```error-output
<!-- pom.xml with CLASSIC loader -->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <loaderImplementation>CLASSIC</loaderImplementation>
    </configuration>
</plugin>

// Boot 4.0 build error:
[ERROR] Failed to execute goal org.springframework.boot:spring-boot-maven-plugin
...CLASSIC loader implementation is no longer supported
```

## What Changed {.what-changed}

The <code>CLASSIC</code> loader implementation, the original Spring Boot uber-jar format, was removed from the Maven and Gradle plugins. Any build file with <code>loaderImplementation=CLASSIC</code> fails at the packaging phase. The default nested-jar loader (introduced in Boot 3.2) is the only supported format.

## Why {.why-changed}

The classic loader was deprecated in Spring Boot 3.2 when the nested-jar loader arrived; Boot 4.0 removes it. The new loader offers better compatibility with AOT compilation, virtual threads, and GraalVM native image tooling.

## The Fix {.diffs}

```diff-card
# // Maven — remove CLASSIC loader configuration
@@removed
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <loaderImplementation>CLASSIC</loaderImplementation>
    </configuration>
</plugin>
@@added
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <!-- No loaderImplementation needed — default is the only option -->
</plugin>
```

```diff-card
# // Gradle — remove loaderImplementation property
@@removed
tasks.named("bootJar") {
    loaderImplementation = LoaderImplementation.CLASSIC
}
@@added
// No loaderImplementation configuration needed
```

## How To Fix {.fixes}

#### Remove the CLASSIC loader configuration.

Delete <code>&lt;loaderImplementation&gt;CLASSIC&lt;/loaderImplementation&gt;</code> from the Maven plugin configuration, or remove <code>loaderImplementation = LoaderImplementation.CLASSIC</code> from the Gradle boot jar task. The build then defaults to the modern nested-jar loader.

## Scope Check {.scope-check}

Search build files for <code>CLASSIC</code> and <code>loaderImplementation</code>. Check <code>pom.xml</code>, parent POMs, and any Gradle build scripts that configure the Spring Boot plugin.

## Watch Out {.watch-out}

- The new nested-jar loader changes the path structure inside the uber jar. Tools that inspect the jar directly (custom launch scripts, Docker entrypoints referencing internal paths) may need updates to reflect the <code>BOOT-INF/</code> layout rather than the classic flat layout.

## Verify {.verify}

Application jar is built successfully and starts correctly after removing CLASSIC loader configuration from the build file

## Further Info {.further-info}

The plugin rejects the <code>CLASSIC</code> value with an explicit error rather than ignoring it, so the failure is loud and easy to find.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

