## Acknowledgement

These notes are based on and adapted from the original course material available in the [Spring Framework Full Course repository](https://github.com/adityatandon15/Spring-Framework-Full-Course/blob/main/Lecture%2023/notes/Notes.pdf).

Credit for the original content belongs to its respective author. The material has been condensed and reorganized for personal learning, reference, and revision purposes.

Please refer to the original source for the complete material and context.

# Spring AOP: Implementing Advice Types

Spring AOP applies additional behavior before, after, or around selected Spring bean method executions without mixing that behavior into business logic. Common cross-cutting concerns include logging, authorization, auditing, metrics, and performance measurement.

## 1. Add the AOP Dependency

The starter depends on the Spring Boot version.

**Spring Boot 3.x**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

**Spring Boot 4.x**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aspectj</artifactId>
</dependency>
```

In a normal Spring Boot application, the starter configures the AOP infrastructure; `@EnableAspectJAutoProxy` is generally not required manually.

## 2. Target Bean, Aspect, and Pointcut

The **target** is the Spring-managed object containing the business logic. For example:

```java
@Service
public class StudentService {

    public String createStudent() {
        System.out.println("Actual createStudent() method executed");
        return "Student created successfully";
    }

    public String getStudent() {
        return "Student details";
    }
}
```

An aspect is a Spring bean that defines cross-cutting behavior and where it applies:

```java
@Aspect
@Component
public class LoggingAspect {

    @Before("execution(String com.coderarmy.studentmanagement.service.StudentService.createStudent())")
    public void logBeforeMethodExecution() {
        System.out.println("createStudent() is about to execute");
    }
}
```

- `@Aspect` identifies the class as an aspect.
- `@Component` registers it as a Spring bean so component scanning can discover it.
- `@Before` runs its advice before matched method executions.

### Reading a Pointcut Expression

```java
execution(* com.coderarmy.studentmanagement.service.PaymentService.*(..))
```

| Part | Meaning |
|---|---|
| `execution` | Match a method execution |
| First `*` | Any return type |
| `PaymentService` | Methods in this class |
| Method `*` | Any method name |
| `(..)` | Any number and type of arguments |

A **pointcut** selects where advice applies. An **advice** is the additional action that executes.

## 3. Target and Proxy

Spring AOP normally exposes a proxy in place of the target bean. Calls pass through the proxy, which intercepts matching method executions and applies the configured advice before delegating to the target.

```text
Caller -> Spring proxy -> Advice/interceptor chain -> Target method
```

The caller normally interacts with the proxy through the usual Spring bean reference.

## 4. The Five Advice Types

### `@Before` — Before the Method

Runs before the matched target method begins. A `JoinPoint` provides information about the current invocation.

```java
@Before("execution(* com.coderarmy.studentmanagement.service.PaymentService.*(..))")
public void logMethodEntry(JoinPoint joinPoint) {
    System.out.println("Entering: " + joinPoint.getSignature().getName());
    System.out.println("Arguments: " + Arrays.toString(joinPoint.getArgs()));
}
```

Useful for method-entry logging, inspecting arguments, and checking common preconditions.

`@Before` cannot directly replace method arguments or return an alternative result. It can stop the target from being reached by throwing an exception. Use `@Around` when you need controlled short-circuiting or argument replacement.

### `@AfterReturning` — After Successful Completion

Runs only when the target method completes normally. Use `returning` to bind the returned value to an advice parameter.

```java
@AfterReturning(
    pointcut = "execution(* com.coderarmy.studentmanagement.service.PaymentService.*(..))",
    returning = "result"
)
public void logReturnedValue(Object result) {
    System.out.println("Method returned: " + result);
}
```

The `returning` name must match the advice parameter name, and the parameter type must be compatible with the result. The advice's own return value does **not** replace the target method's return value. Use this advice mainly for logging, metrics, auditing, or observing successful outcomes.

### `@AfterThrowing` — When an Exception Escapes

Runs when the target method exits by throwing an exception. Use `throwing` to bind the exception to a parameter.

```java
@AfterThrowing(
    pointcut = "execution(* com.coderarmy.studentmanagement.service.PaymentService.*(..))",
    throwing = "exception"
)
public void logException(Throwable exception) {
    System.out.println("Failure: " + exception.getMessage());
}
```

The `throwing` name must match the parameter name. A narrower parameter type, such as `IllegalArgumentException`, restricts the advice to compatible exception types.

`@AfterThrowing` observes the failure; it does not automatically consume the exception or return a fallback value. The original exception continues toward the caller unless other logic handles it.

### `@After` — After Every Outcome

Behaves like a Java `finally` block: it runs whether the target method returns normally or throws an exception.

```java
@After("execution(* com.coderarmy.studentmanagement.service.PaymentService.*(..))")
public void clearContext() {
    System.out.println("Clearing method context");
}
```

Use it for outcome-independent cleanup, such as clearing contextual data or removing a `ThreadLocal` value. It does not directly provide the returned value or thrown exception.

### `@Around` — Control the Complete Invocation

`@Around` surrounds the invocation and receives a `ProceedingJoinPoint`. Calling `proceed()` continues the interceptor chain and eventually invokes the target method.

```java
@Around("execution(* com.coderarmy.studentmanagement.service.PaymentService.*(..))")
public Object measureExecutionTime(ProceedingJoinPoint joinPoint)
        throws Throwable {

    long startTime = System.nanoTime();

    try {
        return joinPoint.proceed();
    } finally {
        long elapsed = System.nanoTime() - startTime;
        System.out.println(
            joinPoint.getSignature().getName()
                + " took " + elapsed + " nanoseconds"
        );
    }
}
```

The `finally` block records timing on both success and failure. Returning the result of `proceed()` preserves the target's return value; returning `null` instead would make the caller receive `null`.

`@Around` can also:

- Replace arguments using `joinPoint.proceed(modifiedArguments)`.
- Replace or wrap a result.
- Catch, rethrow, or convert an exception.
- Skip the target by not calling `proceed()`.
- Call `proceed()` more than once.

These capabilities can change business behavior. Repeating an invocation can duplicate side effects such as charging a card, creating an order, or sending an email, so retries require a carefully designed policy and safe-to-retry operations.

## 5. `JoinPoint` vs. `ProceedingJoinPoint`

| Type | Purpose | Common methods |
|---|---|---|
| `JoinPoint` | Inspect the current intercepted invocation | `getSignature()`, `getArgs()`, `getTarget()`, `getThis()` |
| `ProceedingJoinPoint` | Inspect and control an invocation in `@Around` advice | `proceed()`, `proceed(Object[] arguments)` |

`getTarget()` refers to the underlying target object; `getThis()` refers to the proxy handling the call.

## 6. Advice Comparison

| Requirement | Advice type |
|---|---|
| Run logic before a method | `@Before` |
| Observe a successful result | `@AfterReturning` |
| Observe a failure and access the exception | `@AfterThrowing` |
| Run cleanup regardless of outcome | `@After` |
| Measure complete execution time | `@Around` |
| Replace arguments or the returned value | `@Around` |
| Skip execution or return a fallback value | `@Around` |
| Retry an invocation | `@Around`, with care |

A target method that catches an internal exception and returns a fallback normally is considered to have completed normally: `@AfterReturning` runs, while `@AfterThrowing` does not. Advice reacts to how the target method exits, not to every internal event.

## 7. Interview Questions

**Q1. What is advice in Spring AOP?**

Advice is the additional behavior applied to method executions matched by a pointcut.

**Q2. What is the difference between a pointcut and advice?**

A pointcut defines where behavior applies; advice defines what action runs.

**Q3. What is the difference between `@After` and `@AfterReturning`?**

`@After` runs on success or failure, like `finally`. `@AfterReturning` runs only after normal completion.

**Q4. Does `@AfterThrowing` handle or suppress an exception?**

No. It observes an exception that escapes the target method. The exception continues unless explicitly handled elsewhere.

**Q5. Can `@AfterReturning` replace a return value?**

No. Use `@Around` when the caller must receive a transformed or replacement result.

**Q6. What does `ProceedingJoinPoint.proceed()` do?**

It continues the remaining interceptor chain and eventually invokes the target method.

**Q7. Why should `@Around` be used carefully?**

It can skip or repeat a method, modify arguments and results, or change exception behavior. These changes can alter business semantics or duplicate side effects.

## 8. Key Takeaways

- `@Before`: runs before the target method.
- `@AfterReturning`: runs after normal completion.
- `@AfterThrowing`: runs when an exception escapes the target.
- `@After`: runs regardless of the outcome, like `finally`.
- `@Around`: controls the complete invocation through `ProceedingJoinPoint`.
- Prefer the narrowest advice type that satisfies the requirement; use `@Around` only when invocation control is needed.

> **Interview mental model:** `@Before` = before; `@AfterReturning` = successful return; `@AfterThrowing` = escaping exception; `@After` = finally; `@Around` = full invocation control.
