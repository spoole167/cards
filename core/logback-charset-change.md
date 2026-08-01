---
id: logback-charset-change
tier: 3
tier_label: Wrong Results
title: Logback Default Charset Changed to UTF-8
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: core
---

Boot 4.0 forces Logback file appenders to UTF-8. Non-ASCII characters appear garbled to tools expecting the platform's default encoding. No error: just wrong characters.

## What You'll See {.error-output}

```error-output
# Boot 3.5 (platform encoding = Windows-1252):
2025-01-01 INFO  Order confirmed: €150.00 for Müller

# Boot 4.0 (UTF-8 forced, consumed by tool expecting Windows-1252):
2025-01-01 INFO  Order confirmed: â‚¬150.00 for MÃ¼ller
# Or if the terminal/tool can't decode UTF-8 sequences:
2025-01-01 INFO  Order confirmed: ?150.00 for M?ller
```

## What Changed {.what-changed}

Spring Boot's default Logback configuration changed the charset for file appenders to UTF-8. Console appenders now use <code>Console#charset()</code> (the JVM's console charset, Java 17+) rather than the platform default. The change affects the default Logback XML provided by Spring Boot; custom Logback configurations that already specify an explicit charset are unaffected.

## Why {.why-changed}

UTF-8 is the universal standard for text encoding. Platform-default encoding produced different bytes on different operating systems, making cross-platform log analysis unreliable.

## The Fix {.diffs}

```diff-card
# // logback-spring.xml — if you need to override the default
@@removed
<!-- Boot 3.5: no charset specified, platform default used for file appender -->
<appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>app.log</file>
    <encoder>
        <pattern>%d{yyyy-MM-dd} %-5level %msg%n</pattern>
    </encoder>
</appender>
@@added
<!-- Boot 4.0: UTF-8 is default; specify charset explicitly if you need platform encoding -->
<appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>app.log</file>
    <encoder>
        <charset>UTF-8</charset>
        <pattern>%d{yyyy-MM-dd} %-5level %msg%n</pattern>
    </encoder>
</appender>
```

## How To Fix {.fixes}

#### Update log consumers to expect UTF-8.

If log files are consumed by external tools, log aggregators, or scripts, ensure those consumers are configured to read UTF-8. This is the correct long-term fix.

#### Restore platform encoding if needed.

If UTF-8 is not acceptable for your environment, add an explicit <code>&lt;charset&gt;</code> to your file appenders in <code>logback-spring.xml</code>. This is a short-term workaround; prefer updating consumers to UTF-8.

## Scope Check {.scope-check}

Check <code>logback-spring.xml</code> and <code>logback.xml</code> for file appenders that do not specify a <code>&lt;charset&gt;</code>. Also audit downstream tools that specify the input encoding for log files: Splunk forwarders, Fluentd parsers, Elastic Filebeat configs.

## Watch Out {.watch-out}

- This only affects file appenders. Console output uses the JVM console charset (usually controlled by the terminal), not forced to UTF-8.
- Custom Logback configurations that already declare <code>&lt;charset&gt;UTF-8&lt;/charset&gt;</code> explicitly are unaffected.

## Verify {.verify}

Log output contains correctly encoded characters (especially non-ASCII) in both console and file appenders after the charset change

## Further Info {.further-info}

Boot 3.5 used the platform's default encoding: often Cp1252 or GBK on Windows, sometimes ISO-8859-1 on Linux. Consumers built around those encodings will misread the new UTF-8 bytes in international text, currency symbols, and emoji.

## Links {.footer-links}

- [spring-break module: logback-charset-change](https://github.com/spoole167/spring-break/tree/main/logback-charset-change)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

