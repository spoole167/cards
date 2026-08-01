---
id: aspectj-observed
tier: 2
tier_label: Won't Run
title: AspectJ Weaving Required for @Observed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: observability
---

Spring Boot 4.0 requires AspectJ-based weaving for the @Observed annotation to produce spans and metrics. Without the aspect configured, @Observed does nothing.

## What You'll See {.error-output}

```error-output
# No startup error. Detected when @Observed methods
# produce no observations:

@Observed(name = "order.process")
public Order processOrder(OrderRequest request) { ... }

# Expected in metrics:
$ curl -s http://localhost:8080/actuator/metrics/order.process
{"name":"order.process","measurements":[{"statistic":"COUNT","value":42.0}]}

# Actual after upgrade:
$ curl -s http://localhost:8080/actuator/metrics/order.process
{"status":404,"error":"Not Found"}

# No metric registered, no spans created. The method runs
# normally; observability is gone.
```

## What Changed {.what-changed}

In Spring Boot 3.x, <code>@Observed</code> worked via Spring AOP proxies and auto-configuration registered the <code>ObservedAspect</code> bean. In Spring Boot 4.0 that auto-configuration requires explicit AspectJ weaving support. Without the AspectJ weaving dependency and configuration, <code>@Observed</code> is ignored.

## Why {.why-changed}

The change gives <code>@Observed</code> full AspectJ semantics: it now works on non-proxied beans, self-invocations, and private methods, the places where the proxy-based approach failed without warning.

## The Fix {.diffs}

```diff-card
# // Maven dependency — add AspectJ weaver
@@removed
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing</artifactId>
</dependency>
@@added
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing</artifactId>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
</dependency>
```

```diff-card
# // Configuration — register ObservedAspect if not auto-configured
@@removed
// ObservedAspect was auto-configured in Boot 3.x
@@added
@Configuration
public class ObservabilityConfig {
    @Bean
    public ObservedAspect observedAspect(ObservationRegistry registry) {
        return new ObservedAspect(registry);
    }
}
```

```diff-card
# // application.properties — enable AspectJ auto-proxy
@@removed
# proxy-based AOP was sufficient
@@added
spring.aop.auto=true
```

## How To Fix {.fixes}

#### Add AspectJ weaver dependency.

Add <code>org.aspectj:aspectjweaver</code> to your dependencies. Spring Boot's dependency management provides the version.

#### Register ObservedAspect bean explicitly.

If the auto-configuration does not pick up the aspect, register an <code>ObservedAspect</code> bean manually in a <code>@Configuration</code> class. Pass the <code>ObservationRegistry</code> to its constructor.

#### Verify AOP is enabled.

Ensure <code>spring.aop.auto=true</code> (the default) and that you have not disabled AspectJ auto-proxying. Check for <code>@EnableAspectJAutoProxy</code> if you have custom AOP configuration.

## Scope Check {.scope-check}

Search for <code>@Observed</code> annotations across your codebase. Every <code>@Observed</code> method is broken until AspectJ weaving is configured. Also check whether dashboards or alerts depend on metrics or traces it produces.

## Watch Out {.watch-out}

- There is no error and no warning. The application starts, methods execute, and no observations (metrics or traces) are recorded. You discover it when monitoring dashboards show gaps.
- If you use <code>@Observed</code> on <code>@Repository</code> or <code>@Service</code> beans that are JDK dynamic proxies, ensure AspectJ weaving mode is compatible. CGLIB proxies are generally safer.
- Existing <code>@Timed</code> and <code>@Counted</code> annotations from Micrometer are separate from <code>@Observed</code> and may have their own configuration requirements. Check both.
- Load-time weaving (<code>-javaagent:aspectjweaver.jar</code>) gives the broadest coverage but requires JVM argument changes. Compile-time weaving requires build plugin changes. Choose based on your deployment model.

## Verify {.verify}

Custom @Observed aspects fire and metrics appear in /actuator/metrics

## Further Info {.further-info}

Part of the broader Micrometer Observation API refactoring in Spring Framework 7.0. See also: controller-spans-disabled, observability-dashboard-gaps.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/aspectj-observed)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

