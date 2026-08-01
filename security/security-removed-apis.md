---
id: security-removed-apis
tier: 1
tier_label: Won't Build
title: Security DSL Rewrite
series: spring-boot 3.5 → 4.0
effort: L
openrewrite: true
subsystem: security
max_pages: 2
---

Spring Security removed authorizeRequests(), antMatchers(), and the .and() chaining method. The HTTP security DSL needs a full rewrite.

## What You'll See {.error-output}

```error-output
$ mvn compile
[ERROR] /src/main/java/com/example/SecurityConfig.java:[22,14]
  error: cannot find symbol
    symbol:   method authorizeRequests()
    location: variable http of type HttpSecurity
[ERROR] /src/main/java/com/example/SecurityConfig.java:[23,18]
  error: cannot find symbol
    symbol:   method antMatchers(java.lang.String)
[ERROR] /src/main/java/com/example/SecurityConfig.java:[28,10]
  error: cannot find symbol
    symbol:   method and()
```

## What Changed {.what-changed}

Spring Security 7.0 (shipped with Boot 4.0) removed the deprecated <code>authorizeRequests()</code> method: use <code>authorizeHttpRequests()</code> instead. <code>antMatchers()</code>, <code>mvcMatchers()</code>, and <code>regexMatchers()</code> are replaced by <code>requestMatchers()</code>. The <code>.and()</code> chaining method is gone; the lambda DSL takes its place.

## Why {.why-changed}

The lambda DSL is more readable and avoids a class of configuration bugs where <code>.and()</code> chains silently reset the security context. The new <code>requestMatchers()</code> API auto-detects whether to use Ant or MVC matching based on your classpath.

## The Fix {.diffs}

```diff-card
# // Security filter chain — before
@@removed
http
    .authorizeRequests()
        .antMatchers("/public/**").permitAll()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
        .and()
    .formLogin()
        .loginPage("/login")
        .and()
    .logout()
        .logoutSuccessUrl("/");
@@added
http
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/public/**").permitAll()
        .requestMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated()
    )
    .formLogin(form -> form
        .loginPage("/login")
    )
    .logout(logout -> logout
        .logoutSuccessUrl("/")
    );
```

```diff-card
# // CSRF configuration
@@removed
http.csrf().disable();
@@added
http.csrf(csrf -> csrf.disable());
```

## How To Fix {.fixes}

#### Rewrite to lambda DSL.

Replace every <code>.and()</code> chain with a lambda-based configuration block. Each security concern (authorization, form login, logout, CSRF) gets its own lambda. See the <a href="https://docs.spring.io/spring-security/reference/6.5/migration-7/configuration.html">Spring Security 7 migration guide</a>.

#### Replace matchers.

Change <code>antMatchers()</code> and <code>mvcMatchers()</code> to <code>requestMatchers()</code>. The new method auto-selects the matching strategy. <code>regexMatchers()</code> becomes <code>requestMatchers(new RegexRequestMatcher(...))</code>.

## Scope Check {.scope-check}

Search for <code>authorizeRequests()</code>, <code>antMatchers()</code>, <code>mvcMatchers()</code>, <code>.and()</code> in security config classes, and <code>.csrf().disable()</code>. Every <code>SecurityFilterChain</code> bean needs reviewing.

## Watch Out {.watch-out}

- The <code>.and()</code> removal is the most disruptive part. A long chain with 5+ <code>.and()</code> calls means restructuring the whole method. Review the result carefully: it's easy to nest a lambda inside the wrong block.
- <code>requestMatchers()</code> auto-detects MVC vs Ant matching. If you have Spring MVC on the classpath, it uses MVC matching (which is stricter). If this changes behaviour, use <code>AntPathRequestMatcher</code> explicitly.

## Verify {.verify}

App starts and security filter chain initialises without errors

## Further Info {.further-info}

Driven by Spring Security 7.0. The old DSL was deprecated in Security 5.7/6.0; the lambda DSL has been the recommended replacement since Security 5.2. Affects every spring-boot-starter-security consumer. See also: websecurity-adapter-removed, auth-default-deny.

## Links {.footer-links}

- [spring-break module: security-removed-apis](https://github.com/spoole167/spring-break/tree/main/security-removed-apis)

- [Spring Security 7 migration guide](https://docs.spring.io/spring-security/reference/6.5/migration-7/configuration.html)

