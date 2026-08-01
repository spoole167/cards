---
id: mockito-test-execution-listener
tier: 2
tier_label: Won't Run
title: MockitoTestExecutionListener Removed
series: spring-boot 3.5 → 4.0
effort: S
openrewrite: false
subsystem: testing
---

Spring Boot 4.0 removes MockitoTestExecutionListener. @Mock fields in @SpringBootTest tests stay null and throw NullPointerException unless @ExtendWith(MockitoExtension.class) is added.

## What You'll See {.error-output}

```error-output
@SpringBootTest
class MyTest {
    @Mock
    private MyService myService;

    @Test
    void test() {
        // Boot 3.5: myService is initialised → passes
        // Boot 4.0: myService is null → NullPointerException
        when(myService.greet()).thenReturn("Hello");
    }
}

java.lang.NullPointerException: Cannot invoke method on null object
  at com.example.MyTest.test(MyTest.java:15)
```

## What Changed {.what-changed}

<code>MockitoTestExecutionListener</code> was a Spring Boot-specific bridge that triggered Mockito's annotation processing inside Spring's test lifecycle. Spring Boot 4.0 removed this listener, aligning with the standard JUnit 5 approach of using <code>@ExtendWith(MockitoExtension.class)</code> or <code>MockitoAnnotations.openMocks(this)</code> in a <code>@BeforeEach</code> setup method.

## Why {.why-changed}

The listener duplicated Mockito's own JUnit 5 extension. Removing it pushes tests toward the idiomatic Mockito + JUnit 5 pattern that works in any JUnit 5 project.

## The Fix {.diffs}

```diff-card
# // Add MockitoExtension alongside @SpringBootTest
@@removed
@SpringBootTest
class MyTest {
    @Mock
    private MyService myService;
}
@@added
@SpringBootTest
@ExtendWith(MockitoExtension.class)
class MyTest {
    @Mock
    private MyService myService;
}
```

```diff-card
# // Or switch to @MockitoBean (Boot 4.0 replacement for @MockBean)
@@removed
@SpringBootTest
class MyTest {
    @Mock
    private MyService myService;
}
@@added
@SpringBootTest
class MyTest {
    @MockitoBean
    private MyService myService;  // registered in the Spring context
}
```

## How To Fix {.fixes}

#### Add @ExtendWith(MockitoExtension.class) to the test class.

This is the standard JUnit 5 way to enable Mockito annotation processing. It works alongside <code>@SpringBootTest</code> and needs no new dependency: Mockito's JUnit 5 extension is already on the test classpath via <code>spring-boot-starter-test</code>.

#### Use @MockitoBean instead of @Mock (if the mock belongs in the context).

If the mock is being injected into a Spring bean under test, use <code>@MockitoBean</code> (Boot 4.0's replacement for <code>@MockBean</code>). This registers the mock in the Spring application context rather than just the test instance.

## Scope Check {.scope-check}

Search for <code>@Mock</code>, <code>@Spy</code>, and <code>@Captor</code> annotations in test classes that also use <code>@SpringBootTest</code> or any Spring test slice annotation (<code>@WebMvcTest</code>, <code>@DataJpaTest</code>, etc.). Each such class needs <code>@ExtendWith(MockitoExtension.class)</code> added.

## Watch Out {.watch-out}

- If a test class extends another that has <code>@Mock</code> fields, the parent class also needs the extension. Inheritance doesn't propagate the <code>@ExtendWith</code> from child to parent field initialisation.
- Tests that used <code>@MockBean</code> (Boot 3.5) are affected by a separate change: <code>@MockBean</code> itself is removed in Boot 4.0 in favour of <code>@MockitoBean</code>. See the <code>mockbean-removed</code> card.

## Verify {.verify}

@Mock fields in @SpringBootTest classes are initialised correctly

## Further Info {.further-info}

The listener called MockitoAnnotations.openMocks() on each test instance, initialising @Mock, @Spy, and @Captor fields before each test. Mockito's own JUnit 5 extension has done the same job since Mockito 3.x.

## Links {.footer-links}

- [spring-break module: mockito-test-execution-listener](https://github.com/spoole167/spring-break/tree/main/mockito-test-execution-listener)

- [Spring Boot 4.0 Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

