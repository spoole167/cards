---
id: graalvm-25
tier: 1
tier_label: Won't Build
title: GraalVM 25 for Native Image Builds
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: core
no_module: true
no_module_reason: Requires native:compile with GraalVM 25; not invocable from mvn
  test
---

Spring Boot 4.0 requires GraalVM 25 for native image compilation. Older GraalVM versions produce build failures or runtime crashes.

## What You'll See {.error-output}

```error-output
$ mvn -Pnative native:compile
[ERROR] Error: Unsupported native-image version: 22.3.5
[ERROR] Spring Boot 4.0 requires GraalVM 25.0 or later.
[ERROR] Please upgrade your GraalVM installation.
[ERROR]
[ERROR] -> [Help 1]
[ERROR] org.graalvm.buildtools:native-maven-plugin:0.10.6:compile failed
```

## What Changed {.what-changed}

Spring Boot 4.0 targets GraalVM 25 (based on JDK 25) for native image builds. The <code>native-maven-plugin</code> and <code>org.graalvm.buildtools.native</code> Gradle plugin must be updated to version 0.10.6+. Older GraalVM versions (22.x, 23.x) are no longer compatible with the reachability metadata shipped in starters.

## Why {.why-changed}

GraalVM 25 introduced a new metadata format and improved closed-world analysis. Spring's reachability metadata was rewritten to use these features, making it incompatible with older native-image compilers.

## The Fix {.diffs}

```diff-card
# // pom.xml — native build tools plugin
@@removed
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
    <version>0.9.28</version>
</plugin>
@@added
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
    <version>0.10.6</version>
</plugin>
```

```diff-card
# // build.gradle — GraalVM plugin
@@removed
plugins {
    id 'org.graalvm.buildtools.native' version '0.9.28'
}
@@added
plugins {
    id 'org.graalvm.buildtools.native' version '0.10.6'
}
```

```diff-card
# // JAVA_HOME / GRAALVM_HOME
@@removed
export GRAALVM_HOME=/opt/graalvm-ce-java17-22.3.5
@@added
export GRAALVM_HOME=/opt/graalvm-jdk-25+35
```

## How To Fix {.fixes}

#### Install GraalVM 25.

Download GraalVM 25 from <a href="https://www.graalvm.org/downloads/">graalvm.org</a> and set <code>GRAALVM_HOME</code>. If using SDKMAN: <code>sdk install java 25-graal</code>.

#### Update native build tools.

Bump the <code>native-maven-plugin</code> or Gradle <code>org.graalvm.buildtools.native</code> plugin to 0.10.6+. The parent POM manages this if you inherit from <code>spring-boot-starter-parent</code>.

## Scope Check {.scope-check}

Search for <code>native-maven-plugin</code> and <code>org.graalvm.buildtools.native</code> in build files. Check CI pipelines for <code>GRAALVM_HOME</code> or <code>native-image</code> version references.

## Watch Out {.watch-out}

- GraalVM 25 is based on JDK 25, but the native image it produces still runs on any compatible OS. Don't confuse the build-time JDK requirement with the runtime requirement.
- If you use custom reachability metadata in <code>META-INF/native-image/</code>, verify it still works. The metadata format added new fields in GraalVM 25 and some older entries may be silently ignored.

## Further Info {.further-info}

Applies only to native image builds via the spring-boot-starter-parent native profile; standard JVM deployments are unaffected.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

