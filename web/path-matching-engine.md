---
id: path-matching-engine
tier: 3
tier_label: Wrong Results
title: 'Path Matching Engine: AntPathMatcher → PathPattern'
series: spring-boot 3.5 → 4.0
effort: M
openrewrite: false
subsystem: web
---

Spring Security now uses <code>PathPattern</code> instead of <code>AntPathMatcher</code>. Some wildcard patterns match differently, causing silent authorisation gaps or false denials.

## What You'll See {.error-output}

```error-output
// Security configuration — unchanged
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/api/**/admin").hasRole("ADMIN")
);

// Before (AntPathMatcher — Spring Boot 3.5)
GET /api/v1/admin         → 403 (matched, requires ADMIN)
GET /api/v1/users/admin   → 403 (matched, ** crosses segments)

// After (PathPattern — Spring Boot 4.0)
GET /api/v1/admin         → 403 (still matched)
GET /api/v1/users/admin   → 200 (NOT matched — pattern behaves differently)

// Security test failure
Expected: request denied (403)
  Actual: request allowed (200)
// Potential security hole: endpoint exposed without authorisation
```

## What Changed {.what-changed}

Spring Boot 4.0 switched the default URL path matching engine from <code>AntPathMatcher</code> to <code>PathPatternParser</code> for both MVC request mapping and Security request matchers. While most patterns work identically, edge cases around <code>**</code> in the middle of patterns, trailing slashes, and encoded path segments differ.

## Why {.why-changed}

<code>PathPatternParser</code> is significantly faster (pre-parsed at startup, no runtime string manipulation), handles path variables and matrix parameters correctly, and is the default in Spring WebFlux. Unifying on one engine simplifies the framework.

## The Fix {.diffs}

```diff-card
# // Fix: use PathPattern syntax for multi-segment match
@@removed
.requestMatchers("/api/**/admin").hasRole("ADMIN")
@@added
.requestMatchers("/api/{*path}/admin").hasRole("ADMIN")
```

```diff-card
# // Trailing slash no longer matches by default
@@removed
.requestMatchers("/api/users").authenticated()
@@added
.requestMatchers("/api/users", "/api/users/").authenticated()
```

```diff-card
# // Explicit matcher construction: AntPathRequestMatcher is gone
@@removed
.requestMatchers(new AntPathRequestMatcher("/public/**"))
    .permitAll()
@@added
.requestMatchers(PathPatternRequestMatcher.withDefaults()
    .matcher("/public/**")).permitAll()
```

## How To Fix {.fixes}

#### Audit all path patterns in security config.

Review every <code>requestMatchers()</code> pattern for uses of <code>**</code> in the middle (not at the end), trailing slashes, and URL-encoded segments. Test each pattern with PathPattern semantics. Use <code>{*varName}</code> for capturing multi-segment path variables.

#### Rewrite every legacy pattern: there is no fallback.

Spring Security 7 removes <code>AntPathRequestMatcher</code> entirely, so you cannot keep the old engine for individual rules. Explicit matcher construction uses <code>PathPatternRequestMatcher.withDefaults().matcher(pattern)</code>, and the pattern itself must be valid PathPattern syntax.

## Scope Check {.scope-check}

Check every <code>.requestMatchers()</code> call in your security configuration and every <code>@RequestMapping</code> pattern in your controllers. Patterns with <code>**</code> in the middle, trailing slashes, or regex segments are the most likely to break. Simple patterns like <code>/api/users/**</code> (at the end) are generally safe.

## Watch Out {.watch-out}

- A pattern that no longer matches is a security hole: paths that <code>/api/**/admin</code> used to protect become unprotected. A Tier 3 change with Tier 0 security implications.
- <code>PathPattern</code> does not support <code>**</code> in the middle of a pattern (e.g. <code>/api/**/admin</code>). It only supports <code>**</code> at the end (e.g. <code>/api/**</code>). Use <code>{*path}</code> for variable-length segment capture in the middle.

## Verify {.verify}

All API endpoints respond (especially wildcard/regex patterns)

## Further Info {.further-info}

PathPatternParser has been WebFlux's default since Spring 5.0; Boot 4.0 brings MVC and Security into line. See also: security-dsl-rewrite, auth-default-deny.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/path-matching-engine)

- [Spring Security 7 migration guide](https://docs.spring.io/spring-security/reference/6.5/migration-7/configuration.html)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

