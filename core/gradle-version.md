---
id: gradle-version
tier: 1
tier_label: Won't Build
title: Gradle 8.14+ / Gradle 9 Required
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
no_module: true
no_module_reason: 'Gradle plugin version constraint: only observable via a Gradle
  build, not mvn test'
---

Spring Boot 4.0's Gradle plugin requires Gradle 8.14 or later. Older Gradle wrappers fail immediately at configuration time.

## What You'll See {.error-output}

```error-output
$ ./gradlew build
FAILURE: Build failed with an exception.
* Where:
Build file '/project/build.gradle' line: 3
* What went wrong:
An exception occurred applying plugin request [id: 'org.springframework.boot', version: '4.0.0']
> Failed to apply plugin 'org.springframework.boot'.
   > Spring Boot plugin requires Gradle 8.14+. Found: 8.5.
```

## What Changed {.what-changed}

The Spring Boot Gradle plugin now requires Gradle 8.14 as a minimum. Gradle 9.0 is fully supported and recommended for new projects.

## Why {.why-changed}

Gradle 8.14 introduced configuration cache stability and improved dependency resolution APIs that the Spring Boot plugin now relies on. Supporting older Gradle versions held back build performance improvements.

## The Fix {.diffs}

```diff-card
# // gradle/wrapper/gradle-wrapper.properties
@@removed
distributionUrl=https\://services.gradle.org/distributions/gradle-8.5-bin.zip
@@added
distributionUrl=https\://services.gradle.org/distributions/gradle-8.14-bin.zip
```

```diff-card
# // Alternative: jump to Gradle 9
@@removed
distributionUrl=https\://services.gradle.org/distributions/gradle-8.5-bin.zip
@@added
distributionUrl=https\://services.gradle.org/distributions/gradle-9.0-bin.zip
```

## How To Fix {.fixes}

#### Update the Gradle wrapper.

Run <code>./gradlew wrapper --gradle-version 8.14</code> (or <code>9.0</code>) to update the wrapper in place. Commit the updated wrapper files.

#### Check plugin compatibility.

After upgrading Gradle, run a build to verify all other plugins are compatible. The <a href="https://docs.gradle.org/current/userguide/upgrading_version_8.html">Gradle upgrade guide</a> lists deprecations to address.

## Scope Check {.scope-check}

Check <code>gradle/wrapper/gradle-wrapper.properties</code> for the current Gradle version. Also check CI caches and any shared Gradle distributions. Multi-module projects only need one wrapper update.

## Watch Out {.watch-out}

- If you're on Gradle 7.x, you can't jump straight to 8.14. Gradle deprecation cycles mean you need to fix warnings at each major version boundary. Plan for a staged upgrade: 7.x to 8.0, then 8.0 to 8.14+.
- The Gradle configuration cache is now stable and on by default in Gradle 9. If your build scripts use project-level state at execution time, you'll hit configuration cache errors after upgrading.

## Further Info {.further-info}

Maven builds are unaffected.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

