---
id: observability-dashboard-gaps
tier: 3
tier_label: Wrong Results
title: Controller/View Spans Silently Disabled
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: observability
no_module: true
no_module_reason: Same break as controller-spans-disabled from a dashboard perspective;
  untestable without live tracing infrastructure
---

Spring Boot 4.0 disables automatic controller and view rendering spans by default. Observability dashboards show gaps where traces used to appear; the app itself works fine.

## What You'll See {.error-output}

```error-output
// Grafana / Jaeger trace — before (Spring Boot 3.5)
── HTTP GET /api/orders ─────────────────────────────
  └── controller: OrderController.getOrders  [12ms]
      └── view: orders/list                  [3ms]
          └── jdbc: SELECT * FROM orders     [8ms]

// Grafana / Jaeger trace — after (Spring Boot 4.0)
── HTTP GET /api/orders ─────────────────────────────
  └── jdbc: SELECT * FROM orders             [8ms]
// controller and view spans MISSING

// Dashboard alert
"No data" for panel "Controller Response Time p99"
// SLO breach: "99th percentile latency unknown"
```

## What Changed {.what-changed}

Spring Boot 4.0 disabled the automatic instrumentation for Spring MVC controller method spans and view rendering spans by default. The properties <code>management.observations.http.server.requests.enabled</code> and related <code>spring.mvc.observation</code> settings changed their defaults. The traces still record HTTP and JDBC spans, but the controller-level granularity is gone.

## Why {.why-changed}

Controller spans added an extra span per request and duplicated information already in the HTTP server span. The new default cuts overhead and storage costs, with opt-in for teams that need the detail.

## The Fix {.diffs}

```diff-card
# // application.properties — re-enable controller spans
@@removed
# (default in Boot 3.5: controller spans enabled)
@@added
management.tracing.enabled=true
spring.mvc.observation.auto-configuration.enabled=true
```

```diff-card
# // Or enable via ObservationRegistry customiser
@@removed
# (auto-instrumentation was on by default)
@@added
@Bean
public ObservationRegistryCustomizer<ObservationRegistry> serverSpans() {
    return registry -> registry.observationConfig()
        .observationHandler(new DefaultMeterObservationHandler(meterRegistry));
}
```

```diff-card
# // Micrometer tracing config
@@removed
# (no explicit config needed in Boot 3.5)
@@added
management.observations.http.server.requests.enabled=true
management.observations.http.client.requests.enabled=true
```

## How To Fix {.fixes}

#### Re-enable controller observation spans.

Add <code>spring.mvc.observation.auto-configuration.enabled=true</code> and <code>management.observations.http.server.requests.enabled=true</code> to your <code>application.properties</code>. This restores the controller-level spans in your traces.

#### Update dashboards to use HTTP spans.

If the overhead matters, update your Grafana/Datadog dashboards to use the HTTP server span (<code>http.server.requests</code>) instead of the controller span. This span is still present and includes the handler information as attributes.

## Scope Check {.scope-check}

Check your observability dashboards, SLO definitions, and alerting rules. Any panel or alert that queries for controller-level span names (e.g. <code>controller.*</code>) or view rendering spans will show "No data" after the upgrade. Also check custom <code>SpanExporter</code> or <code>ObservationHandler</code> beans that filter by span name.

## Watch Out {.watch-out}

- Logs and functionality look normal. The only symptom is missing data in your observability platform, often unnoticed until an incident needs the traces that are gone.
- SLO alerts based on controller span latency percentiles now fire as "no data", which some alerting systems treat as "OK". Your SLO monitoring silently disappears.

## Verify {.verify}

All Grafana/Jaeger panels show data: no 'No data' gaps

## Further Info {.further-info}

Part of Boot 4.0's Micrometer Tracing defaults overhaul. See also: controller-spans-disabled, aspectj-observed.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

