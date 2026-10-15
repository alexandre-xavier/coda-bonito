---
name: bonito
description: Java and Groovy code style rules for coding (parameter ordering, blank lines, primitives vs wrappers, var, dead-code cleanup, safe logging, constants, lambdas). Use when writing, refactoring, or reviewing Java or Groovy code, including Spock tests.
metadata:
  version: 0.2.0
  updated: 2026-10-15
  origin: develop
---

# Java and Groovy Style Rules (v0.2.0)

Each rule records something a reviewer or a scanner caught. Apply them to new code and to any code you touch.

## How to use this skill

1. Before writing code, read the **Quick reference**.
2. Write the code.
3. Go through the **Verification checklist** on your diff, not on the whole file.

---

## Quick reference

| # | Rule | Category |
|---|------|----------|
| P1 | Request context (`tenantId`, `actor`, `requestId`, `correlationId`, `locale`) goes **last**, in that order | Ordering |
| P2 | Domain data goes **first**, led by the entity's own `id`. Foreign keys count as domain data | Ordering |
| NM1 | Units go in field names with an `In<Unit>` suffix (`timeoutInSeconds`) | Naming |
| NM2 | Name each field after its role (`orderService`, not `service`) | Naming |
| NM3 | Test names are behaviour sentences: camelCase in JUnit, strings in Spock | Naming |
| NM4 | No `is`/`has` prefixes on booleans, for fields, variables, or methods | Naming |
| NM5 | Countable lists use the plural (`orders`). Uncountables and nested lists use a `List` suffix (`contentList`, `ordersList`) | Naming |
| F1 | Declarations put each parameter on its own line, with the closing `)` on its own line | Formatting |
| F2 | Calls go multi-line when they have 2+ arguments, or when a 1-argument call would exceed 80 characters. Exception: F5 (call followed by a chain) | Formatting |
| F3 | Leave a blank line right after a type declaration, before the first member | Formatting |
| F4 | Leave blank lines between groups of related statements, around mid-method `if`/`for`/`try`, and before `return` | Formatting |
| F5 | Line length (~120 chars, indentation included); call+chain may stay on one line if under the limit (exception to F2) | Formatting |
| F6 | One dot per line for chains (2+ dots). Lambda bodies exempt. Nested object construction: at most 3 levels (prefer 2 in new code) | Formatting |
| T1 | Domain records use **primitives** for values that are always present. Request DTOs keep **wrappers** + `@NotNull` | Types |
| T2 | Use `Optional` only as a return type, never as a field or parameter | Types |
| T3 | Prefer `var` (Java and Groovy) when the right-hand side or the method name makes the type obvious | Types |
| C1 | Remove unused imports | Cleanup |
| C2 | Remove unused parameters, private methods, fields, and locals, and update every caller and mock | Cleanup |
| C3 | No factory methods that only forward to the constructor. Factories that add meaning are fine | Cleanup |
| C4 | No wrappers that do nothing and no premature abstractions | Cleanup |
| L1 | Never log secrets or PII. Request-controlled values are allowed only at `DEBUG` or in non-production code | Logging |
| L2 | Put contextual identifiers in MDC, not in the message text | Logging |
| L3 | Comment an MDC entry when it isn't obvious why the value is safe | Logging |
| N1 | Extract magic numbers into named constants. Prefer `Duration` (no unit suffix); a `long` gets `_IN_SECONDS` | Constants |
| N2 | Shared keys go in a `final` holder class with a private constructor | Constants |
| N3 | DTO validation annotations reference the domain constants | Constants |
| LM1 | An exception assertion (`assertThrows`, or Spock `when:` + `thrown()`) contains **one** call that can throw | Lambdas |
| LM2 | Collapse duplicated ternary branches into one expression | Lambdas |
| LM3 | Move a lambda longer than about 3 lines into a well-named method | Lambdas |
| CM1 | Comments explain *why*, not *what* | Comments |
| S1 | JUnit 5 + Mockito for unit tests, Spock for component/integration tests | Tests |
| S2 | Give every Spock block a descriptive label | Tests |
| S3 | Keep `var x = Mockito.verify(...) // NOSONAR` in Spock `then:`/`and:` blocks | Tests |
| S4 | When you delete code, also delete the tests that only covered it | Tests |
| S5 | Stub with `doReturn`/`doThrow(...).when(mock).method(...)` | Tests |
| S6 | Test `Mono`/`Flux` with `StepVerifier` | Tests |
| S7 | Use typed matchers in `verify` calls (`any(Type.class)`, `anyString()`), not bare `any()`. Arguments one per line (F2) | Tests |
| S8 | Three logical blocks in a test (setup, execution, verification) separated by blank lines. F4 grouping applies within a block | Tests |

---

## 1. Parameter ordering

### P1/P2: domain data first (led by `id`), request context last

**Why:** request context comes from the request or the token. It is not domain data. When context always goes at the end, every signature has the same shape. A reader sees the domain fields first. Adding or removing a context field also doesn't shift the domain arguments.

The order is:
1. **Domain data.** The entity's own `id` comes first, then the other fields. Foreign keys such as `customerId` belong here too.
2. **Context**, always in this order: `tenantId`, `actor`, `requestId`, `correlationId`, `locale`.

```java
// Incorrect
public record Order(
    String id,
    String tenantId,
    String actor,
    String name,
    String description
) { }

// Correct
public record Order(
    String id,
    String name,
    String description,
    String tenantId,
    String actor
) { }
```

The same order applies to method parameters, mappers, and references:
- `toDomain(request, tenantId, actor)`
- `toDomain(id, request, tenantId, actor)`
- `new OrderReference(id, tenantId, actor)`

**Exception:** a service method takes `tenantId` **first** when the tenant is the partition or lookup key: `service.getOrder(tenantId, id)`, `service.createOrder(tenantId, order)`. To tell the two cases apart, ask whether the method looks something up *by* tenant or whether the tenant only travels along with the data. In the first case `tenantId` is a key and goes first. In the second it is context and goes last.

---

## 2. Naming

- **NM1:** units go in field names with an `In<Unit>` suffix: `deliveryTimeoutInSeconds`, `retentionPeriodInDays`. This doesn't apply to `Duration` values (see N1).
- **NM2:** name each field after its role: `orderService`, `orderMapper`, `authenticationResolver`. Avoid generic names like `service` or `helper`.
- **NM3:** test names are behaviour sentences.
  - JUnit: camelCase, e.g. `createIsIsolatedPerTenant`, `updatePreservesStoredStatus`.
  - Spock: string feature names, e.g. `def "create is isolated per tenant"()`.
- **NM4:** no `is`/`has` prefixes on booleans. This covers fields, variables, and methods: `boolean active` and `active()`, not `isActive`. JDK and library APIs (`isEmpty()`) are out of scope.
- **NM5:** collections:
  - A list of countable items uses the plural: `List<Order> orders`.
  - A list of uncountables uses the `List` suffix: `List<Content> contentList`, not `List<Contents> content`.
  - A nested list uses the `List` suffix: `List<List<Order>> ordersList`, not `orders`.

---

## 3. Formatting

### F1: one parameter per line in declarations

**Why:** a diff shows one line per changed parameter, and adding a parameter never reflows the line. This covers **single-parameter** declarations too, in Java and Groovy. Methods with no arguments stay on one line. Lambda parameters and record compact constructors are not affected.

```java
public HttpResponse<Void> deleteOrder(
    UUID id
) {
```

### F2: multi-line call sites

Put each argument on its own line when a call has **2 or more arguments**, or when a call with **one argument** would exceed **80 characters** on one line. The rule applies everywhere, tests included.

```java
var order = orderMapper.toDomain(
    request,
    tenantId,
    actor
);

orderService.deleteOrder(id);
```

### F3: blank line after the type declaration

Leave a blank line right after a class, record, enum, interface, or exception declaration, before its first member. For records, it goes after `) {`. Nested types follow the same rule.

```java
// Incorrect
public class OrderService {
    private static final int MAX_ORDERS_PER_TENANT = 50;

// Correct
public class OrderService {

    private static final int MAX_ORDERS_PER_TENANT = 50;
```

- No blank line before the closing `}`.
- Empty bodies stay on one line: `record Marker() { }`, `class X extends Y { }`.
- Spock spec classes are exempt.

### F4: blank lines inside method bodies

- Group closely related statements and leave a blank line between the groups.
- Leave a blank line before and after an `if`/`for`/`try` block that sits in the middle of a method.
- Leave a blank line before `return`. The exception is a method whose only statement is the `return`.
- In Spock, leave a blank line between `given:`/`when:`/`then:` blocks.

```java
var tenantId = resolveTenantId();
var orders = orderService.listOrders(tenantId);

if (orders.isEmpty()) {
    log.debug("No orders for tenant");
}

var summaries = orderMapper.toSummaryList(orders);

return HttpResponse.ok(summaries);
```

### F5: line length (soft target, ~120 chars; exception to F2 for call + chain)

**Why:** F2 splits every call with 2+ arguments, which makes a call followed by a chain needlessly tall. Lines aim for 120 characters including indentation. When a method call is followed by a chain, the call may stay on one line if the full indented line does not exceed 120 characters, even with 3 or more arguments. The chain goes on the next line, one call per line (F6).

**Call + chain, line fits in 120 characters (4 spaces of indentation, 120 characters):** keep the call on one line.

```java
// Correct: call on one line, chain on the next line
    maybeActionsWithUpdatedTokenSettings(policyRule.getActions(), refreshTokenWindowMinutes, accessTokenLifetimeMinutes)
        .ifPresent(...);
// Because:
// 1. A chained call follows (here, ifPresent)
// 2. The full indented line is 120 characters, so F5 lets it stay on one line
```

**Call + chain, line exceeds 120 characters (8 spaces of indentation, 124 characters):** F2 takes precedence. Extract the call to a variable with one argument per line, then call the chain on the variable.

```java
// Incorrect: 124 characters with its indentation
        maybeActionsWithUpdatedTokenSettings(policyRule.getActions(), refreshTokenWindowMinutes, accessTokenLifetimeMinutes)
            .ifPresent(...);

// Correct
        var maybeActions = maybeActionsWithUpdatedTokenSettings(
            policyRule.getActions(),
            refreshTokenWindowMinutes,
            accessTokenLifetimeMinutes
        );

        maybeActions.ifPresent(...);
// Because:
// 1. A chained call follows, but the line is over 120 characters, so F2 applies
// 2. The variable name is shorter than the method name, so the chain stays readable
```

**Standalone call (nothing chained after it):** F5 does not apply. A standalone call follows F2, and when you assign it to a variable the split is the only change. Shortening names so that the call fits in 120 characters does not exempt it from F2.

```java
// Incorrect: standalone call kept on one line (118 characters with 4 spaces of indentation)
    var maybeActions = maybeActionsWithUpdatedTokenSettings(actions, refreshTokenWindowMins, accessTokenLifetimeMins);

// Correct: F2 splits the arguments, the result is still assigned to a variable
    var maybeActions = maybeActionsWithUpdatedTokenSettings(
        actions,
        refreshTokenWindowMins,
        accessTokenLifetimeMins
    );
// Because: no chain follows, so the F5 exception does not apply
```

- Count the indentation when measuring the line.
- 120 is a soft guideline. Readability trumps the limit: a line a few characters over that is clearer than the split is acceptable, and a line under 120 that is unreadable should still be broken.

### F6: one dot per line for chains (2+ dots), with lambda exemption

**Why:** a chain written on one line hides where each call starts, and deeply nested construction hides the shape of the object.

**One dot per line.** A **chain** is two or more method calls on the same object on one line (`a.b().c()`). Every chain puts each call on its own line, one dot per line, constructors included: `new X()` is followed by `.method()` on the next line. **Exception:** lambda bodies are exempt from this rule. A single-dot call (`receiver.method()`) is not a chain and stays inline. That includes the plain getters in an F2 argument split (`policyRule.getActions()`).

```java
// Incorrect
var rule = new AuthorizationServerPolicyRule().actions(actions).name(name);

// Correct
var rule = new AuthorizationServerPolicyRule()
    .actions(actions)
    .name(name);

// Correct: single-dot getters inside an F2 argument split stay inline
var rule = buildRule(
    policyRule.getActions(),
    policyRule.getName()
);

// Correct: the lambda body is exempt, and the reverse equals is preferred
var tokenSettings = policyRule
    .getActions()
    .stream()
    .filter(action -> "token_settings".equals(action.getType()))
    .findFirst();
```

**Extract to intermediate variables for readability.**

A **level** is any constructor or method call that holds another object construction in its argument list. Count from the outermost call, `withId(...)` included.

- Chained calls on the same object (`.getActions().stream()...`) do **not** add levels.
- A variable or getter argument ends the nesting.
- **Limit: at most 3 levels; prefer 2 in new code.** When there are more, extract the inner levels into variables with a role name.

```java
// Incorrect: four levels
// 1
return withId(
    // 2
    new AuthorizationServerPolicyRule()
        .actions(
            // 3
            new AuthorizationServerPolicyRuleActions()
                .token(
                    // 4
                    new TokenAuthorizationServerPolicyRuleAction()
                        .refreshTokenWindowMinutes(refreshTokenWindowMinutes)
                        .accessTokenLifetimeMinutes(accessTokenLifetimeMinutes)
                )
        ),
    id
);

// Correct (preferred): two levels, full extraction
var tokenAction = new TokenAuthorizationServerPolicyRuleAction()
    .refreshTokenWindowMinutes(refreshTokenWindowMinutes)
    .accessTokenLifetimeMinutes(accessTokenLifetimeMinutes);

var actions = new AuthorizationServerPolicyRuleActions()
    .token(tokenAction);

var rule = new AuthorizationServerPolicyRule()
    .actions(actions);

return withId(rule, id);
// 2 levels: withId and Rule

// Correct (allowed, less preferred): three levels, partial extraction
var tokenAction = new TokenAuthorizationServerPolicyRuleAction()
    .refreshTokenWindowMinutes(refreshTokenWindowMinutes)
    .accessTokenLifetimeMinutes(accessTokenLifetimeMinutes);

// 1
var rule = new AuthorizationServerPolicyRule()
    .actions(
        // 2
        new AuthorizationServerPolicyRuleActions()
            .token(
                // 3
                tokenAction
            )
    );

return withId(rule, id);
// 3 levels: withId, Rule, Actions
```

Chains do not add levels: the `policyRule.getActions().stream()...` chain above is 0 levels, because every call is on the same object.

---

## 4. Types

### T1: primitives in domain records, wrappers at the HTTP boundary

**Why:** a domain value that can't be absent shouldn't be nullable. Primitives let you delete every `x != null && ...` guard in the compact constructor. The request DTO keeps the wrapper because `@NotNull` can only reject a missing JSON field when the type is nullable. With a primitive, a missing field silently becomes `0`.

```java
// Request DTO: wrapper + @NotNull
public record OrderRequest(
    @NotNull
    @Min(Order.MIN_QUANTITY)
    Long quantity
) { }

// Domain record: primitive, no null checks
public record Order(
    String id,
    long quantity,
    String tenantId,
    String actor
) {

    public Order {
        if (quantity < MIN_QUANTITY) { ... }   // before: quantity != null && quantity < 1
    }
}
```

How to decide:
- If the value can legitimately be absent in the domain, use a wrapper.
- If it is a request field that must be present, use a wrapper + `@NotNull` in the DTO and a primitive in the domain.
- If it is always present in the domain, use a primitive.

### T2: `Optional` only as a return type

Use `Optional` to signal that a method may return no value. Never use it for a field or a parameter.

### T3: prefer `var`

Use `var` whenever the right-hand side or the method name makes the type obvious. This applies to Java and to Groovy locals. In Groovy, use `var`, not `def`.

```java
try (var ignored = MdcScope.of(order.toLogContext())) { ... }

var reference = orderMapper.toReference(order);

for (var order : orders) { ... }
```

---

## 5. Code cleanup

### C1: unused imports

After every refactor, recheck the imports in each file you touched, tests included. When you delete a class or a test, remove the imports that only it needed.

### C2: unused parameters, private methods, fields, and locals

```java
// Incorrect: actor already lives on the order
public Order createOrder(
    String tenantId,
    String actor,
    Order order
)

// Correct
public Order createOrder(
    String tenantId,
    Order order
)
```

Removing a parameter means fixing **every** caller and every mock or stub that refers to it. That includes positional argument indexes: when the middle parameter goes, `getArgument(2)` becomes `getArgument(1)`.

### C3: no forwarding factory methods

**Why:** a factory such as `Order.create(...)` that only adds `UUID.randomUUID()` repeats the constructor signature, so every field change has to be made twice. Use the canonical constructor, and generate the ID in the code that decides to create the entity (e.g. the mapper).

Factories that **add meaning** are fine: `Order.empty(tenantId, actor)`, `Settings.defaults(...)`.

### C4: no wrappers that do nothing

- Don't `return Map.copyOf(context)` when `context` is a local map you just built. Return `context`.
- Don't hand-roll sanitizers to quiet a scanner (see L1).
- Keep code factual and concise. No premature abstractions.

---

## 6. Logging

### L1: what may and may not be logged

**Why:** scanners such as CodeQL don't recognise custom sanitizers. A hand-rolled `Sanitizer.clean(request.getPath())` leaves the alert open and adds a utility class to maintain. The fix is to **stop logging the value**.

```java
// Incorrect
log.info("Order operation rejected: {} {}", exception.reason(), request.getPath());

// Also incorrect (still flagged)
log.info("... {}", Sanitizer.clean(request.getPath()));

// Correct
log.info("Order operation rejected: {}", exception.reason());   // enum, server-side
log.error("Unexpected error in orders API", exception);          // static message names the API
```

- **Never, at any level:**
  - secrets (keys, client secrets, tokens)
  - PII (emails, names, IP addresses)
  - `requestId`/`correlationId` in the message text. They belong in MDC, set at the entrypoint.
- **Only at `DEBUG` or in non-production code:** values the request controls, i.e. paths, methods, query strings, headers, and free-text body fields.
- **Safe:**
  - enums and constants you define
  - values from a verified token (`tenantId`, `actor`), added through MDC
  - UUIDs that path binding validated or that the server generated
  - the exception, passed as the last argument

### L2/L3: context goes through MDC

Put contextual identifiers in MDC, not in the message text. One pattern is a scoped MDC helper, such as `MdcScope`, around the call, fed by a `toLogContext()` method on the domain object:

```java
try (var ignored = MdcScope.of(order.toLogContext())) {
    ...
}
```

Use `Map.of(...)` when every value is guaranteed to be non-null. Use a `LinkedHashMap` with null checks only for values that can be absent. Add a comment only when it isn't obvious why an entry is safe to log:

```java
public Map<String, String> toLogContext() {
    return Map.of(
        MdcKeys.ORDER_ID_KEY, id.toString(),
        MdcKeys.TENANT_KEY, tenantId,   // validated and extracted from the JWT
        MdcKeys.ACTOR_KEY, actor        // JWT username, not free text
    );
}
```

---

## 7. Constants

### N1: magic numbers become named constants

```java
public static final int MAX_ORDERS_PER_TENANT = 50;
public static final Duration DEFAULT_TIMEOUT = Duration.ofMinutes(15);
public static final long MIN_TIMEOUT_IN_SECONDS = 60L;   // used in @Min, which needs a compile-time long
```

- Names are `UPPER_SNAKE_CASE` and start with the role: `MAX_`, `MIN_`, `DEFAULT_`.
- Prefer `Duration` for durations. A `Duration` constant has **no** unit suffix.
- When a duration must be a primitive (annotation limits, wire formats), use a `long` with the `_IN_SECONDS` suffix.
- Build error messages from the constant (`"must be at least " + MIN_TIMEOUT_IN_SECONDS + " seconds"`), so the message and the check can't drift apart.
- Test constants (`TENANT_ID`, `ACTOR`, `BASE_URL`) are recommended but not required.

### N2: shared keys go in a holder class

Keys used by several classes (MDC keys, header names) live in one `final` class with a private constructor. Don't copy them into each record. When a constant must stay in step with another one, a comment says so.

```java
public final class MdcKeys {

    // Must match RequestFilter.TENANT_KEY
    public static final String TENANT_KEY = "tenant.id";
    public static final String ACTOR_KEY = "actor";

    private MdcKeys() {
    }
}
```

### N3: one source of truth for limits

DTO validation annotations reference the domain constants (`@Min(Order.MIN_QUANTITY)`) instead of repeating `@Min(1)`.

---

## 8. Lambdas and complexity

### LM1: one call that can throw per exception assertion

Build the arguments before the assertion, `UUID.randomUUID()` included.

```java
// Incorrect: the constructor and toDomain can both throw
assertThrows(OrderException.class, () -> mapper.toDomain(new OrderRequest(0L), TENANT_ID, ACTOR));

// Correct
var request = new OrderRequest(0L);

assertThrows(
    OrderException.class,
    () -> mapper.toDomain(
        request,
        TENANT_ID,
        ACTOR
    )
);
```

In Spock, the `when:` block before `thrown()` holds the single call that can throw. Setup goes in `given:`:

```groovy
given: "a request below the minimum quantity"
var request = new OrderRequest(0L)

when: "the request is mapped"
mapper.toDomain(
    request,
    TENANT_ID,
    ACTOR
)

then: "the mapping is rejected"
thrown(OrderException)
```

### LM2: collapse duplicated ternary branches

```java
// Incorrect
List<Order> orders = existing != null ? new ArrayList<>(existing) : new ArrayList<>(defaults(tenantId));

// Correct
List<Order> orders = new ArrayList<>(
    existing != null ? existing : defaults(tenantId)
);
```

### LM3: long lambdas become named methods

Move a lambda longer than about 3 lines into a method, in Java and in Spock. The method name must be cohesive and make the call site easier to read: `orders.forEach(this::archiveExpiredOrder)`.

### General

- Use a method reference or a single-expression lambda when a block lambda adds nothing.
- When the same logic appears in **2 or more** methods, extract a well-named private helper. Never extract one just to satisfy a metric (see C3/C4).

---

## 9. Comments

### CM1: explain why, not what

Comments explain *why*: why a logged value is safe, a constant that must stay in step with another one, or a non-obvious decision. Don't write a comment that repeats what the code says.

---

## 10. Tests (Java + Groovy)

### S1: frameworks

- **Unit tests:** JUnit 5 + Mockito, named `*Test.java`, under `src/test/java`.
- **Component and integration tests:** Spock, named `*CompSpec.groovy`, extending the project's base spec.

### S2: Spock block labels

Every block (`given:`, `when:`, `then:`, `and:`, `expect:`, `where:`) gets a label that states its intent: `when: "the orders of a new tenant are listed"`.

### S3: keep the Spock verify assignment

In `then:`/`and:`, Spock evaluates a bare `Mockito.verify(...)` statement as a condition, and its `null` return fails the test. Assign it and suppress the "unused" warning. Don't delete the line.

```groovy
then: "the order is saved once"
var saved = Mockito.verify(repository).save(order) // NOSONAR
```

### S4: delete tests for deleted code

When you delete production code, delete the tests that only covered it.

### S5: `doReturn`/`doThrow` stubbing

Prefer `doReturn(...)`/`doThrow(...)` with `.when(mock).method(...)` over `when(...).thenReturn(...)`/`.thenThrow(...)`. It reads better and keeps setup separate from assertions.

```java
doReturn(createdOrder)
    .when(service)
    .createOrder(
        tenantId,
        order
    );

doThrow(new OrderException(OrderErrorReason.NOT_FOUND))
    .when(service)
    .getOrder(
        tenantId,
        id
    );
```

**Exception:** parameterless getters on value objects can use `when().thenReturn()`:

```java
when(valueObjectMock.getSomeValue())
    .thenReturn(someValue);
```

### S6: `StepVerifier` for reactive types

Always test `Mono` and `Flux` with `StepVerifier`. Never call `.block()` in a test.

```java
var result = service.findOrder(
    tenantId,
    id
);

StepVerifier.create(result)
    .expectNext(order)
    .verifyComplete();
```

### S7: typed matchers in `verify`

A bare `any()` matches anything, so a verification that uses it hides a wrong argument type or a swapped argument. Use the typed matcher: `any(Type.class)`, `anyString()`, `anyList()`, `anyInt()`. Keep `eq(...)` or the literal for arguments the test cares about. The verified call goes on the line after `verify(...)` (F6, one dot per line), with one argument per line (F2).

```java
// Incorrect
verify(repository, never()).save(any(), any());

// Correct
verify(repository, never())
    .save(
        any(AgentTokenSettings.class),
        anyString()
    );
```

Typed matchers also apply to Mockito `doReturn(...).when(mock).method(...)` stubs when you are touching them anyway. A bare `any()` is acceptable only for a parameter that can be `null`.

### S8: logical blocks in a test

A test has three blocks: setup, execution, verification. Separate them with a blank line (the F4 rule for Spock `given:`/`when:`/`then:`, applied to JUnit). Several setup statements, or several verifications, that belong together stay together. Within a test block, F4 logical grouping still applies. F2 applies to every call in the test, so constructors and assertions with 2+ arguments are split one argument per line.

```java
@Test
void getReturnsExistingSettingsWithoutCreatingAnything() {
    var existing = new AgentTokenSettings(
        TENANT_ID,
        3,
        600,
        60
    );

    doReturn(Optional.of(existing))
        .when(repository)
        .fetch(TENANT_ID);

    var settings = service.getAgentTokenSettings(TENANT_ID);

    assertSame(
        existing,
        settings
    );
    verify(repository, never())
        .save(
            any(AgentTokenSettings.class),
            anyString()
        );
}
```

---

## Decision trees

**Adding a parameter**
```
Is it request context (tenantId, actor, requestId, correlationId, locale)?
 ├─ yes -> END, in that order
 └─ no  -> domain data: entity id first, then the other fields (foreign keys included) in API-contract order
Is the value always present in the domain? -> primitive (T1). Optional only as a return type (T2)
```

**Writing a `log.*` call**
```
Is it a secret or PII?                                    -> never, at any level
Is it a path, method, header, or body text?               -> DEBUG / non-production only. Otherwise remove it
Is it an enum, a constant, token-derived, or a validated UUID? -> OK (prefer MDC for IDs)
Unsure?                                                   -> don't log it
```

**Scanner or linter finding**
```
Unused import/param/field/method/local -> remove it, fix callers and mocks (except a Spock verify, S3)
Magic number                           -> named constant, Duration where possible (N1)
Lambda with several calls that can throw -> move setup out of the lambda (LM1)
Log injection                          -> stop logging the value. Never write a sanitizer
```

**Should this helper or factory exist?**
```
Does it only forward to a constructor or wrap something that does nothing? -> delete it
Is it a factory that adds meaning (Order.empty)?                           -> keep it
Is the logic repeated in 2+ methods, or is the lambda longer than ~3 lines? -> extract a well-named method
```

---

## Verification checklist (run on your diff)

Formatting
- [ ] A blank line follows each type declaration (not in Spock specs, not in empty bodies)
- [ ] Declarations put one parameter per line, with `)` on its own line
- [ ] Calls with 2+ arguments, or with 1 argument over 80 characters, are multi-line (F2), except a call followed by a chain that fits in ~120 characters (F5)
- [ ] Line fits within ~120 characters (indentation included); a call with a subsequent chain may stay on one line
- [ ] A call with a chain over ~120 characters is extracted to a variable (one argument per line)
- [ ] A standalone call (no chain) follows F2, even when it fits in 120 characters
- [ ] Every dot on a new line for chains (2+ dots). Lambdas exempt. Single-dot getters inline in F2 splits
- [ ] Nested object construction is at most 3 levels deep (prefer 2 in new code); chained calls do not add levels. Deeper levels are extracted into variables
- [ ] Related statements are grouped, with blank lines between groups, around mid-method `if`/`for`/`try`, before `return`, and between Spock blocks

Ordering, naming, and types
- [ ] Domain data comes first (led by `id`), and context comes last: `tenantId, actor, requestId, correlationId, locale`
- [ ] Units use the `In<Unit>` suffix. No `is`/`has` prefixes. Lists are named `orders` / `contentList` / `ordersList`
- [ ] Domain records use primitives for required values, and request DTOs keep wrappers + `@NotNull`
- [ ] `Optional` appears only as a return type
- [ ] `var` is used where the type is obvious (Java and Groovy)

Cleanup
- [ ] No unused imports, parameters, private methods, fields, or locals in touched files, tests included
- [ ] Every caller, mock, stub, and argument index is updated
- [ ] No forwarding factories, no wrappers that do nothing, no premature abstractions
- [ ] Tests for deleted code are deleted too

Logging
- [ ] No secrets or PII at any level
- [ ] Request-controlled values appear only at `DEBUG`
- [ ] Identifiers go through MDC, with a comment where it isn't obvious why they are safe

Constants and lambdas
- [ ] No bare numeric limits. Durations use `Duration`, or a `long` with `_IN_SECONDS`
- [ ] DTO validation annotations reference the domain constants
- [ ] Shared keys live in a holder class
- [ ] Each exception assertion contains one call that can throw. Lambdas longer than ~3 lines are named methods

Tests
- [ ] Every Spock block has a descriptive label
- [ ] Stubs use `doReturn`/`doThrow`, except for parameterless value-object getters
- [ ] `Mono`/`Flux` are tested with `StepVerifier`
- [ ] `verify` calls use typed matchers (`any(Type.class)`, `anyString()`), not bare `any()`. Arguments one per line in the `verify()` call (F2 applies)
- [ ] Three logical blocks (setup, execution, verification) separated by blank lines; within a block, F4 grouping applies

Build
- [ ] The project's test suite passes

---

## Exceptions and special cases

- **Service lookup methods** take `tenantId` first when it is the lookup key (see P1).
- **Spock `var x = Mockito.verify(...) // NOSONAR`** stays even when it looks unused (S3).
