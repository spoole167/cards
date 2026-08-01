---
id: webjars-locator-core-removed
tier: 1
tier_label: Won't Build
title: webjars-locator-core Removed from BOM
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: web
---

Spring Boot 4.0 removes <code>webjars-locator-core</code> from the BOM. Version-agnostic WebJar URLs (<code>/webjars/jquery/jquery.min.js</code>) stop resolving. Switch to <code>webjars-locator-lite</code>.

## What You'll See {.error-output}

```error-output
// pom.xml — dependency with no version (relies on BOM)
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator-core</artifactId>
</dependency>

// Boot 3.5: BOM provides version 0.59 — resolves fine.

// Boot 4.0: BOM no longer manages webjars-locator-core.
[ERROR] 'dependencies.dependency.version' for
  org.webjars:webjars-locator-core:jar is missing.
[ERROR] BUILD FAILURE

// Or if version is pinned in pom.xml — compiles but WebJarAssetLocator
// is no longer registered by Boot's auto-configuration:
java.lang.ClassNotFoundException: org.webjars.WebJarAssetLocator
```

## What Changed {.what-changed}

<code>org.webjars:webjars-locator-core</code> was the original WebJar asset locator, using full classpath scanning to find versioned WebJar paths. Spring Boot 4.0 removes it from the BOM and its auto-configuration wiring, replacing it with <code>org.webjars:webjars-locator-lite</code>, which uses a pre-generated manifest file (<code>webjars-requirejs.js</code> or <code>META-INF/resources/webjars/</code>) to locate assets without scanning.

## Why {.why-changed}

The classpath scanning approach in <code>webjars-locator-core</code> added startup overhead proportional to the number of JARs on the classpath and was incompatible with GraalVM native image compilation. The lite variant eliminates both problems: manifest-based lookup is O(1) and works in native images.

## The Fix {.diffs}

```diff-card
# // pom.xml — replace locator dependency
@@removed
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator-core</artifactId>
</dependency>
@@added
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator-lite</artifactId>
</dependency>
```

```diff-card
# // Code using WebJarAssetLocator directly
@@removed
import org.webjars.WebJarAssetLocator;
WebJarAssetLocator locator = new WebJarAssetLocator();
String path = locator.getFullPath("jquery", "jquery.min.js");
@@added
import org.webjars.lite.WebJarAssetLocator;
WebJarAssetLocator locator = new WebJarAssetLocator();
String path = locator.getFullPath("jquery", "jquery.min.js");
```

## How To Fix {.fixes}

#### Replace webjars-locator-core with webjars-locator-lite.

Update <code>pom.xml</code> or <code>build.gradle</code> to declare <code>org.webjars:webjars-locator-lite</code>. The API is compatible for common use cases: Spring MVC's WebJar resource handler works with both.

#### Update direct WebJarAssetLocator imports.

If your code imports <code>org.webjars.WebJarAssetLocator</code> directly, change the import to the <code>org.webjars.lite</code> package. The method signatures for <code>getFullPath()</code> are compatible.

## Scope Check {.scope-check}

Search for <code>webjars-locator-core</code> in <code>pom.xml</code> and <code>build.gradle</code> files. Also grep your Java source for <code>import org.webjars.WebJarAssetLocator</code>. Check Thymeleaf templates and JSPs for version-agnostic WebJar URLs (<code>th:href="@{/webjars/bootstrap/css/bootstrap.min.css}"</code>): these require a locator to resolve and will 404 if no locator is present.

## Watch Out {.watch-out}

- If you pin an explicit version of <code>webjars-locator-core</code> in your pom, the build succeeds, but Boot 4.0 no longer wires the auto-configuration. WebJar URLs that worked without a version segment silently return 404 instead of failing at startup.
- <code>webjars-locator-lite</code> requires that WebJar dependencies include a <code>webjars-requirejs.js</code> manifest. Older WebJar artifacts may not include this file. Check the WebJar version you are using is compatible with the lite locator.

## Verify {.verify}

WebJar assets resolve correctly via the webjars-locator-lite replacement

## Further Info {.further-info}

Both locators come from the WebJars project itself; Spring Boot switched which one it manages and wires.

## Links {.footer-links}

- [spring-break module: webjars-locator-core-removed](https://github.com/spoole167/spring-break/tree/main/webjars-locator-core-removed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [webjars-locator-lite on GitHub](https://github.com/webjars/webjars-locator-lite)

