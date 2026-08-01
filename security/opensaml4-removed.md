---
id: opensaml4-removed
tier: 1
tier_label: Won't Build
title: OpenSAML 4 Support Removed
series: spring-boot 3.5 → 4.0
effort: L
openrewrite: false
subsystem: security
max_pages: 3
---

Spring Security 7 dropped OpenSAML 4 support and requires OpenSAML 5. Code referencing OpenSaml4AuthenticationProvider and related classes fails to compile on Boot 4.0.

## What You'll See {.error-output}

```error-output
$ mvn clean compile
[ERROR] COMPILATION ERROR :
[ERROR] SamlConfig.java:[4,63] cannot find symbol
    symbol: class OpenSaml4AuthenticationProvider
[INFO] BUILD FAILURE
---
The build stops at compile. The application never starts.
```

## What Changed {.what-changed}

Spring Security 7 removed the OpenSAML 4 compatibility classes and requires OpenSAML 5. The SAML support classes (<code>OpenSaml4AuthenticationProvider</code>, <code>OpenSaml4AuthenticationRequestResolver</code>, etc.) were replaced with OpenSAML 5 equivalents. OpenSAML 5 changed package structures and removed the marshaller/unmarshaller split.

## Why {.why-changed}

OpenSAML 4 is no longer maintained. OpenSAML 5 brought significant API improvements, including a simplified object-provider model and better XML security defaults. Maintaining dual support across two major OpenSAML versions was unsustainable.

## The Fix {.diffs}

```diff-card
# // Maven dependency
@@removed
<dependency>
    <groupId>org.opensaml</groupId>
    <artifactId>opensaml-saml-api</artifactId>
    <version>4.3.2</version>
</dependency>
<dependency>
    <groupId>org.opensaml</groupId>
    <artifactId>opensaml-saml-impl</artifactId>
    <version>4.3.2</version>
</dependency>
@@added
<dependency>
    <groupId>org.opensaml</groupId>
    <artifactId>opensaml-saml-api</artifactId>
    <version>5.1.2</version>
</dependency>
<dependency>
    <groupId>org.opensaml</groupId>
    <artifactId>opensaml-saml-impl</artifactId>
    <version>5.1.2</version>
</dependency>
```

```diff-card
# // Java config — authentication provider
@@removed
import org.springframework.security.saml2.provider.service.authentication
    .OpenSaml4AuthenticationProvider;

OpenSaml4AuthenticationProvider provider =
    new OpenSaml4AuthenticationProvider();
@@added
import org.springframework.security.saml2.provider.service.authentication
    .OpenSaml5AuthenticationProvider;

OpenSaml5AuthenticationProvider provider =
    new OpenSaml5AuthenticationProvider();
```

```diff-card
# // Java config — custom response validator
@@removed
OpenSaml4AuthenticationProvider provider =
    new OpenSaml4AuthenticationProvider();
provider.setResponseValidator(responseToken -> {
    Saml2ResponseValidatorResult result =
        OpenSaml4AuthenticationProvider
            .createDefaultResponseValidator()
            .convert(responseToken);
@@added
OpenSaml5AuthenticationProvider provider =
    new OpenSaml5AuthenticationProvider();
provider.setResponseValidator(responseToken -> {
    Saml2ResponseValidatorResult result =
        OpenSaml5AuthenticationProvider
            .createDefaultResponseValidator()
            .convert(responseToken);
```

## How To Fix {.fixes}

#### Upgrade OpenSAML dependencies to 5.x.

Update your OpenSAML dependencies to version 5.1.x. If you use Spring Boot's dependency management, the managed version will already be OpenSAML 5 in Spring Boot 4.0. Remove any explicit version overrides pinning to 4.x.

#### Rename Spring Security SAML classes.

Replace all references to <code>OpenSaml4*</code> classes with their <code>OpenSaml5*</code> equivalents. The main classes are <code>OpenSaml5AuthenticationProvider</code> and <code>OpenSaml5AuthenticationRequestResolver</code>.

#### Review custom SAML processing code.

If you directly use OpenSAML APIs (marshallers, unmarshallers, builders), review the OpenSAML 5 migration guide. The marshaller/unmarshaller split was removed and the object provider model changed.

## Scope Check {.scope-check}

Search for <code>opensaml</code> in your Maven/Gradle files, and for <code>OpenSaml4</code> in Java code. If your application uses SAML 2.0 login (<code>saml2Login()</code> in your security config), you are affected. If you only use OAuth 2.0 / OIDC, this card does not apply.

## Watch Out {.watch-out}

- OpenSAML 5 requires Java 17+ and has different XML processing requirements. Verify your XML security libraries are compatible.
- If you have custom <code>Saml2AuthenticationRequestContext</code> processing or assertion decryption, those APIs may have changed signatures in the OpenSAML 5 upgrade.
- The Shibboleth Maven repository is still required for OpenSAML 5 artifacts. Make sure your build still has that repository configured.


## Further Info {.further-info}

The change comes from Spring Security 7.0, which Spring Boot 4.0 ships. Affects spring-boot-starter-security consumers using SAML 2.0 login.

## Links {.footer-links}

- [Spring-Break Demo](https://github.com/spoole167/spring-break/tree/main/opensaml4-removed)

- [Spring Security 7 migration](https://docs.spring.io/spring-security/reference/6.5/migration-7/configuration.html)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

