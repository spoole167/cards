---
id: devtools-livereload-disabled
tier: 3
tier_label: Wrong Results
title: DevTools Live Reload Disabled by Default
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: web
no_module: true
no_module_reason: |
  DevTools live reload only functions when the application is running in a development server process. A standard Maven compile+test module cannot exercise the DevTools restart/live-reload lifecycle. The break is a configuration default change, not a code compilation or unit-testable runtime failure.
---

Boot 4.0 disables DevTools Live Reload by default: no automatic browser refresh after code changes. Set <code>spring.devtools.livereload.enabled=true</code> in your dev profile to restore it.

## What You'll See {.error-output}

```error-output
# Boot 3.5: live reload works automatically with devtools on classpath.
# Boot 4.0: live reload disabled. Browser does not refresh.
# No error: the server restarts but the browser stays on the old page.
```

## What Changed {.what-changed}

The <code>spring.devtools.livereload.enabled</code> property now defaults to <code>false</code>. The Live Reload server is not started unless this property is explicitly set to <code>true</code>.

## Why {.why-changed}

Live Reload opens a port (35729) on the developer's machine and needs a browser extension. Starting it unasked surprised developers who did not know about or want it. Opt-in gives explicit control over which DevTools features are active.

## The Fix {.diffs}

```diff-card
# // application-dev.properties or application.properties
@@removed
# Boot 3.5: live reload on by default, no property needed
@@added
# Boot 4.0: must opt in
spring.devtools.livereload.enabled=true
```

## How To Fix {.fixes}

#### Add spring.devtools.livereload.enabled=true to your dev profile.

Add the property to <code>application-dev.properties</code> or your development-time configuration. Keep it out of production profiles: DevTools should not be on the production classpath anyway, and scoping the property to dev makes the intent clear.

## Scope Check {.scope-check}

Check <code>application.properties</code> and <code>application-dev.properties</code> for any existing DevTools configuration.

## Watch Out {.watch-out}

- The DevTools automatic restart (triggered by classpath changes) is separate from Live Reload and unaffected. The app still restarts; the browser just does not receive the push notification to refresh.

## Verify {.verify}

Browser refreshes automatically after code changes when spring.devtools.livereload.enabled=true is set in development profile

## Further Info {.further-info}

In Boot 3.5, adding <code>spring-boot-devtools</code> to the classpath automatically started a Live Reload server on port 35729, which pushed browser refresh signals after application restarts.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [Spring Boot DevTools Documentation](https://docs.spring.io/spring-boot/reference/using/devtools.html)

