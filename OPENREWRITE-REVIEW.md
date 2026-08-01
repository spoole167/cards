# OpenRewrite Accuracy Review — all 87 cards

Reviewed 2026-08-01 against the actual recipe sources: `openrewrite/rewrite-spring` (spring-boot-40.yml, spring-boot-40-properties.yml, spring-boot-40-modular-starters.yml, spring-batch-5.0/6.0.yml, spring-security-58/60/70.yml, spring-framework-60/70.yml), `openrewrite/rewrite-hibernate` (hibernate-7.0/7.1.yml, recipes.csv), and `openrewrite/rewrite-jackson` (jackson-2-3.yml, recipes.csv). "The Boot 4.0 chain" below means `org.openrewrite.java.spring.boot4.UpgradeSpringBoot_4_0` and everything it transitively includes.

**Score: 18 flags wrong, 5 need a caveat, 64 correct.**

## Flag wrong: `openrewrite: false` but a recipe exists (17 cards)

| Card | Recipe that covers it |
|---|---|
| batch/batch-chunkhandler-renamed | `SpringBatch5To6Migration`: ChangeType `ChunkHandler` → `ChunkRequestHandler`. In the Boot 4.0 chain. |
| batch/batch-joblauncher-removed | `SpringBatch5To6Migration`: renames `JobStep.setJobLauncher` → `setJobOperator`, `JobStepBuilder.launcher` → `operator`, plus `JobLauncher` → `JobOperator` type change. |
| batch/batch-package-moves | `SpringBatch5To6Migration` is mostly this: ChangePackage `batch.item` → `batch.infrastructure.item` etc., plus per-class moves for `Job`, `JobExecution`, `JobInstance`, `JobParameters`, `Step`, `StepExecution` and the rest of the card's exact diff. |
| batch/batch-job-serialisation | `ListenerSupportClassToInterface` (via `SpringBatch4To5Migration`, included in 5→6): rewrites `extends JobExecutionListenerSupport` → `implements JobExecutionListener`, same for Step/Chunk/Skip/Repeat variants. This is the card's exact diff. |
| core/aop-starter-rename | spring-boot-40.yml renames `spring-boot-starter-aop` → `spring-boot-starter-aspectj` (ChangeDependency + managed-versions variant), and removes the dependency entirely when no AspectJ annotations are used. |
| core/listenable-future-removed | `org.openrewrite.java.spring.util.concurrent.ListenableToCompletableFuture`, included in `UpgradeSpringFramework_6_0` (in the chain). Migrates types, `addCallback`, and `ListenableFutureCallback`. **Body bug too, see below.** |
| core/gradle-version | spring-boot-40.yml includes `UpdateGradleWrapper: version ^8.14`. The wrapper bump is automated. |
| core/modular-starters | `MigrateToModularStarters` + `MigrateAutoconfigurePackages` exist and are in the chain: detect package usage, add the right starters, relocate autoconfigure packages. The card's `no_module_reason` argues no *test module* is possible, which is fine, but the flag claims no recipe. |
| data-messaging/spring-session-property-renames | `SpringBootProperties_4_0` renames every property in the card's diff: `spring.session.redis.*` → `spring.session.data.redis.*` (all six keys) and `spring.session.mongodb.collection-name` → `spring.session.data.mongodb.collection-name`. |
| data-messaging/kafka-streams-customizer-removed | `MigrateAutoconfigurePackages`: ChangeType `StreamsBuilderFactoryBeanCustomizer` → `org.springframework.kafka.config.StreamsBuilderFactoryBeanConfigurer`. |
| hibernate/entityscan-relocated | `MigrateAutoconfigurePackages`: ChangePackage `org.springframework.boot.autoconfigure.domain` → `org.springframework.boot.persistence.autoconfigure`, which is where `@EntityScan` lives. |
| observability/actuator-nullable-removed | `org.openrewrite.java.jspecify.MigrateFromSpringFrameworkAnnotations` is in `UpgradeSpringFramework_7_0`. It does the exact fix the card recommends (`org.springframework.lang.Nullable` → JSpecify). |
| testing/springboottest-mockmvc-autoconfig | `AddAutoConfigureMockMvc` in spring-boot-40.yml adds the annotation to `@SpringBootTest` classes using `MockMvc`. (The card's own note doubts the premise reproduces; either way the flag is wrong.) |
| testing/testrest-template-removed | Fully automated: ChangePackage `boot.test.web.client` → `boot.resttestclient`, AddDependency `spring-boot-resttestclient`, plus `AddAutoConfigureTestRestTemplate`. |
| web/resttemplate-autoconfig | `MigrateToModularStarters`: AddDependency `spring-boot-starter-restclient` when `org.springframework.boot.web.client.*` is used, plus ChangePackage `boot.web.client` → `boot.restclient`. That is the card's "keep RestTemplate" fix, automated. |
| jackson/jackson-dates-timestamps | `SpringBootProperties_4_0` renames `spring.jackson.serialization.write-dates-as-timestamps` → `spring.jackson.datatype.datetime.write-dates-as-timestamps` (and the other seven datetime keys). That removes the startup break the card describes; the customiser-bean fix is unnecessary. |
| hibernate/hibernate-empty-interceptor-removed | `org.openrewrite.hibernate.EmptyInterceptorToInterface`: rewrites `extends EmptyInterceptor` → `implements Interceptor` (+ `StatementInspector`). In rewrite-hibernate, pulled in by the Hibernate migration chain. |

## Flag wrong: `openrewrite: true` but no recipe exists (1 card)

| Card | Why |
|---|---|
| batch/batch-schema-change | Sequence renames (`BATCH_JOB_SEQ` → `BATCH_JOB_INSTANCE_SEQ`) are database DDL. No OpenRewrite recipe touches schemas; `SpringBatch5To6Migration` is source-code only. Should be `false`. |

## Caveats needed (5 cards)

| Card | Issue |
|---|---|
| hibernate/hibernate-cascade-removal (`true`) | The open-source chain only maps `CascadeType.DELETE` → `REMOVE`. The card is about `SAVE_UPDATE`, which is handled only by `io.moderne.hibernate.update66.MigrateCascadeTypes` in Moderne's commercial catalog, not by plain OpenRewrite. If "true" means the OSS Boot 4.0 recipe, this is wrong. |
| core/spring-jcl-removed (`true`) | A `RemoveSpringJcl` recipe exists but only in Moderne's catalog (docs.moderne.io); it is not in `openrewrite/rewrite-spring` or the Boot 4.0 chain. Same judgment call as above. |
| data-messaging/pulsar-reactive-removed (`false`) | Inverse case: `RemoveSpringPulsarReactive` exists in Moderne's catalog. `false` is right for OSS OpenRewrite; add a note if Moderne counts. |
| core/deprecated-classes-removed (`false`) | Mostly right. `ReplaceRestTemplateBuilderMethods` does cover the removed 3.1-deprecated setters (`setConnectTimeout` etc.), but not `additionalMessageConverters` or the RestClient migration the card shows. Worth a footnote. |
| observability/otlp-http-primary (`false`) | Correct that no recipe flips transport, but `SpringBootProperties_4_0` does rename all the `management.otlp.*` keys the card's fix uses to `management.opentelemetry.*`. The card's "keep using gRPC" diff shows the old key names, which the recipe will rewrite. Check the diff is Boot 4.0-valid. |

## Body-text errors (independent of flags)

- **core/listenable-future-removed**: body says "The `org.openrewrite.java.spring.framework.MigrateSpringAssert` recipe set includes `ListenableFuture` migration." Wrong recipe. It is `org.openrewrite.java.spring.util.concurrent.ListenableToCompletableFuture`. The card also contradicts itself: frontmatter `false`, body has a "Use OpenRewrite" fix section.
- **jackson/jackson-date-serialisation**: fix says add `spring.jackson.serialization.write-dates-as-timestamps=true`, but the sibling card (jackson-dates-timestamps) says that exact property breaks startup on Boot 4.0. One of the two is wrong; the Boot 4.0 key is `spring.jackson.datatype.datetime.write-dates-as-timestamps`.

## Verified correct (spot checks that held up)

`true` flags confirmed against recipe sources: mockbean-removed (`ReplaceMockBeanAndSpyBean`), testcontainers-class-relocation (`Testcontainers2Migration`), test-slice-relocated (`MigrateAutoconfigurePackages` relocates all the test.autoconfigure slice packages), mongodb-property-renames, javax-annotation-removed, javax-inject-removed (jakarta recipes via the 3.0 chain), maven-aot-plugin (plugin version bump), hibernate-dialect-removal (`MigrateDialect`, 26 mappings incl. properties/yaml), hibernate-processor-rename, hibernate-session-delete, jackson-class-renames, jackson-exception-hierarchy (`JsonProcessingException` → `JacksonException` ChangeType), jackson-group-id, security-removed-apis (`AuthorizeHttpRequests`, `UseNewRequestMatchers`, lambda-DSL recipes via the security 5.8 chain), batch-listener-classes. The two docs.openrewrite.org links in the jackson cards resolve to a real recipe (`UpgradeJackson_2_3_TypeChanges`).

`false` flags confirmed correct (no recipe found): both bootstrap relocation cards, propertymapping-relocated, classic-uber-jar, graalvm-25, logback-charset, null-marked, propertymapper, retry-semantics, retryable-transaction-order, spring-retry-removed, maven-aot-execution-blocks, elasticsearch-rest5client, mongodb-uuid-representation, spring-amqp-retry-removed, both spring-session-*-removed cards, all remaining security cards, remaining hibernate cards (where/orderby, setorder, query-type, native-datetime), remaining jackson cards (component-rename's reasoning is sound, locale, module-autodiscovery, property-inclusion), all remaining observability, mockito-test-execution-listener, spock, batch-inmemory-default, batch-job-builder-string-constructor, and all remaining web cards (undertow is explicitly "intentionally not handled" per the `RelocateWebServerClasses` description).
