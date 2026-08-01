---
id: liveness-readiness-probes-default
tier: 3
tier_label: Wrong Results
title: Liveness and Readiness Probes Enabled by Default
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: observability
---

Boot 4.0 enables liveness and readiness probes everywhere, Kubernetes or not. The <code>/actuator/health</code> response changes shape. Downstream parsers and security rules may break.

## What You'll See {.error-output}

```error-output
// Boot 3.5 (outside Kubernetes): /actuator/health response
{"status":"UP"}

// Boot 4.0: /actuator/health response includes probe groups
{
  "status": "UP",
  "groups": ["liveness", "readiness"]
}

// Consumer code that checked for exact JSON equality or did not
// expect the "groups" key now fails.
```

## What Changed {.what-changed}

The <code>management.endpoint.health.probes.enabled</code> property now defaults to <code>true</code> instead of being driven by Kubernetes detection. Both <code>/actuator/health/liveness</code> and <code>/actuator/health/readiness</code> are always exposed, and the main health response always lists them in the <code>groups</code> field.

## Why {.why-changed}

Probes are useful outside Kubernetes: load balancers, service meshes, and health-check scripts benefit from explicit liveness and readiness semantics. Making them universally available removes the need for environment-specific configuration.

## The Fix {.diffs}

```diff-card
# // application.properties — disable probes if not wanted
@@removed
# Boot 3.5: probes off by default outside Kubernetes
# management.endpoint.health.probes.enabled=true  # required to enable
@@added
# Boot 4.0: probes on by default; disable explicitly if not needed
management.endpoint.health.probes.enabled=false
```

## How To Fix {.fixes}

#### Disable probes if your deployment does not use them.

Set <code>management.endpoint.health.probes.enabled=false</code> in <code>application.properties</code> to restore Boot 3.5 behaviour for non-Kubernetes deployments. Alternatively, update downstream health consumers to expect the <code>groups</code> field.

#### Update security rules if probe endpoints were unintentionally exposed.

Review your actuator security configuration. If <code>/actuator/health/liveness</code> and <code>/actuator/health/readiness</code> were not expected to be publicly accessible, add them to your security exclusions or require authentication.

## Scope Check {.scope-check}

Search for code or tests that parse the <code>/actuator/health</code> response body and check for exact JSON structure. Also review Actuator security configurations for endpoint exposure rules. Check monitoring integrations (Datadog, Prometheus, PagerDuty health checks) that consume the health endpoint.

## Watch Out {.watch-out}

- Application liveness state can be changed programmatically via <code>ApplicationAvailability</code>. If your code already does this, the probes are now always reachable: confirm the state transitions are correct before going to production.

## Verify {.verify}

Health endpoint responds correctly and liveness/readiness groups are present or absent as expected after reviewing probe configuration

## Further Info {.further-info}

Boot 3.5 activated the probes only when it detected a Kubernetes environment (via environment variables) or when <code>management.endpoint.health.probes.enabled=true</code> was set explicitly. Boot 4.0 drops the detection, which can expose probe endpoints to networks that never expected them.

## Links {.footer-links}

- [spring-break module: health-probes-default-on](https://github.com/spoole167/spring-break/tree/main/health-probes-default-on)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

- [Spring Boot Liveness and Readiness Probes](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html#actuator.endpoints.kubernetes-probes)

