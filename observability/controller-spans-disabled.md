---
id: controller-spans-disabled
tier: 2
tier_label: Won't Run
title: Controller & View Spans Disabled by Default
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: observability
no_module: true
no_module_reason: Testing absent spans requires a live tracing backend; no practical
  JUnit assertion
---

Spring Boot 4.0 disables controller and view rendering spans by default in Micrometer tracing. Dashboards, alerts, and SLOs built on those spans go dark.

## What You'll See {.error-output}

```error-output
# No startup error. Detected when spans are missing
# in your observability platform:

$ curl -s http://localhost:16686/api/traces?service=my-app | jq '.data[0].spans[].operationName'
"HTTP GET /api/orders"
# Previously also showed:
# "OrderController#getOrders"
# "orders :: view rendering"
# These spans are now GONE.

# Grafana alert fires:
[FIRING] No controller-level spans for service my-app in last 15m
# Or SLO dashboard shows 0% data for controller latency percentiles.
```

## What Changed {.what-changed}

Spring Boot 4.0 changed the default for <code>management.observations.http.server.requests.name</code> and disabled the creation of child spans for Spring MVC controller method invocations and view rendering. Previously, Micrometer Tracing created spans for both the HTTP request and the controller method execution. Now only the HTTP server request span is created by default.

## Why {.why-changed}

The controller and view spans added overhead and noise for applications that did not need sub-request granularity. Disabling them by default reduces trace volume and performance impact. Applications that need them can re-enable them explicitly.

## The Fix {.diffs}

```diff-card
# // application.properties — re-enable controller spans
@@removed
# default: controller spans were enabled
@@added
management.observations.http.server.requests.controller.enabled=true
management.observations.http.server.requests.view.enabled=true
```

```diff-card
# // application.yml equivalent
@@removed
# default: controller spans were enabled
@@added
management:
  observations:
    http:
      server:
        requests:
          controller:
            enabled: true
          view:
            enabled: true
```

## How To Fix {.fixes}

#### Re-enable controller spans.

Add <code>management.observations.http.server.requests.controller.enabled=true</code> to your <code>application.properties</code>. Do the same for <code>view.enabled</code> if you need view rendering spans. This restores the Spring Boot 3.x behaviour.

#### Update dashboards and alerts.

If you decide to accept the new default (no controller spans), update your Grafana dashboards, alert rules, and SLO definitions to use the HTTP server request span instead of controller-level spans. The HTTP span still contains the handler method in its attributes.

#### Use @Observed for targeted spans.

Instead of blanket controller spans, annotate specific controller methods with <code>@Observed</code> to get spans only where you need them. This is more efficient than re-enabling all controller spans globally.

## Scope Check {.scope-check}

Check your observability dashboards and alerts for references to controller-level span names. Search for span names like <code>controller.method.name</code>, <code>HandlerMethod</code>, or view rendering span names in your Grafana/Datadog/New Relic configuration. If you only use HTTP-level metrics (<code>http.server.requests</code>), you are not affected.

## Watch Out {.watch-out}

- The application gives no errors and no warnings. You discover the change when dashboards go blank or missing-data alerts fire.
- If you use distributed tracing to debug latency between the HTTP layer and the controller, the missing span makes it harder to pinpoint where time is spent inside the server.
- Micrometer's <code>@Timed</code> annotation on controller methods still works for metrics. Only tracing spans are affected, not timer metrics.

## Verify {.verify}

Trace dashboard shows controller-level spans after re-enabling

## Further Info {.further-info}

Part of Spring Boot 4.0's Micrometer Tracing defaults overhaul. See also: observability-dashboard-gaps, aspectj-observed.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

