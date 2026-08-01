---
id: otlp-http-primary
tier: 2
tier_label: Won't Run
title: OTLP/HTTP Now Primary Export Protocol
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: observability
no_module: true
no_module_reason: Port/transport change requires a live OTLP collector; cannot be
  asserted in an isolated Maven test
---

Spring Boot 4.0 changed the default OTLP transport from gRPC to HTTP/protobuf. Exports aimed at gRPC-only collectors get connection refused or vanish without an error.

## What You'll See {.error-output}

```error-output
WARN [otel.exporter] io.opentelemetry.exporter.otlp.http.OtlpHttpSpanExporter:
  Failed to export spans. Server responded with HTTP status 404.
  url=http://otel-collector:4318/v1/traces
---
# Or if collector only exposes gRPC port 4317:
WARN [otel.exporter] io.opentelemetry.exporter.otlp.http.OtlpHttpSpanExporter:
  Failed to export spans.
  java.net.ConnectException: Connection refused
  target=http://otel-collector:4318/v1/traces
---
# Metrics and traces stop arriving in your backend.
# Application continues running but observability data is lost.
```

## What Changed {.what-changed}

Spring Boot 4.0 changed the default OTLP export transport from gRPC (port 4317) to HTTP/protobuf (port 4318). The default endpoint changed from <code>http://localhost:4317</code> to <code>http://localhost:4318/v1/traces</code> (and <code>/v1/metrics</code>, <code>/v1/logs</code>). Applications that relied on the default gRPC transport now send data to the wrong port and protocol.

## Why {.why-changed}

HTTP/protobuf is more universally supported, works through proxies and load balancers without special gRPC configuration, and does not require the gRPC dependency. The OpenTelemetry specification recommends HTTP/protobuf as the default transport.

## The Fix {.diffs}

```diff-card
# // application.properties — keep using gRPC
@@removed
# default was gRPC on port 4317
@@added
management.otlp.tracing.transport=grpc
management.otlp.tracing.endpoint=http://otel-collector:4317
management.otlp.metrics.export.transport=grpc
management.otlp.metrics.export.endpoint=http://otel-collector:4317
```

```diff-card
# // application.properties — accept new HTTP default
@@removed
management.otlp.tracing.endpoint=http://otel-collector:4317
@@added
management.otlp.tracing.endpoint=http://otel-collector:4318/v1/traces
```

```diff-card
# // docker-compose.yml — expose HTTP port on collector
@@removed
otel-collector:
  image: otel/opentelemetry-collector:latest
  ports:
    - "4317:4317"   # gRPC
@@added
otel-collector:
  image: otel/opentelemetry-collector:latest
  ports:
    - "4317:4317"   # gRPC
    - "4318:4318"   # HTTP/protobuf
```

## How To Fix {.fixes}

#### Accept HTTP/protobuf (recommended).

Update your OTLP endpoint to use port 4318 and the HTTP path format. Ensure your collector exposes the HTTP receiver. Most modern collectors (OpenTelemetry Collector, Grafana Agent, Datadog Agent) support both protocols.

#### Explicitly set gRPC transport.

If your collector only supports gRPC, add <code>management.otlp.tracing.transport=grpc</code> and set the endpoint to the gRPC port. This preserves the old behaviour.

#### Update collector configuration.

If using the OpenTelemetry Collector, ensure both the <code>otlp/grpc</code> and <code>otlp/http</code> receivers are enabled in your collector config. This lets you support both old and new applications during migration.

## Scope Check {.scope-check}

Check your <code>application.properties</code> for <code>management.otlp</code> settings. If you have explicit endpoint configuration pointing to port 4317, the application will try HTTP on a gRPC port. If you have no explicit configuration, the default changed from gRPC/4317 to HTTP/4318.

## Watch Out {.watch-out}

- The application starts and runs normally. You only notice when traces and metrics stop appearing in your observability backend. Add health checks for telemetry export.
- If you use a service mesh or network policy that only allows port 4317, you need to open port 4318 as well.
- Some managed observability platforms (Datadog, New Relic) have different endpoints for HTTP vs gRPC OTLP ingestion. Check your provider's documentation for the correct HTTP endpoint URL.
- The gRPC dependency (<code>io.grpc:grpc-netty-shaded</code>) may no longer be pulled in transitively. If you switch back to gRPC, you may need to add it explicitly.

## Verify {.verify}

OTLP exporter sends traces to collector (check collector logs)

## Further Info {.further-info}

Affects spring-boot-starter-actuator consumers using OTLP tracing or metrics export.

## Links {.footer-links}

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

