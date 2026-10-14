## Acknowledgement

The content in this document is reproduced from the educational materials published in the [Coder Army — Spring Framework Full Course repository](https://github.com/adityatandon15/Spring-Framework-Full-Course/blob/main/Lecture%2007/notes/Notes.pdf).

The material has been formatted in Markdown to support my personal learning and provide a more convenient reading experience in a text-based format.

Full credit for the original content belongs to its respective author. This document is maintained for personal educational reference, and no authorship of the original material is claimed.

**Original Source:** [Lecture 07 — Notes (PDF)](https://github.com/adityatandon15/Spring-Framework-Full-Course/blob/main/Lecture%2007/notes/Notes.pdf)

# Spring Bean Lifecycle - Complete Notes

> **One-Line Takeaway:** Spring Bean Lifecycle is not just object creation. It is the complete process through which Spring discovers, creates, prepares, manages, and destroys a bean.

## Table of Contents

- [1. What is Bean Lifecycle?](#1-what-is-bean-lifecycle)
- [2. Simple Definition](#2-simple-definition)
- [3. Complete Bean Lifecycle Flow](#3-complete-bean-lifecycle-flow)
- [4. Step-by-Step Breakdown](#4-step-by-step-breakdown)
- [5. Singleton vs Prototype Lifecycle](#5-singleton-vs-prototype-lifecycle)
- [6. Important Cases & Pitfalls](#6-important-cases--pitfalls)
- [7. Interview Questions & Answers](#7-interview-questions--answers)
- [8. Quick Revision Cheat Sheet](#8-quick-revision-cheat-sheet)

## 1. What is Bean Lifecycle?

### Normal Java vs Spring

In normal Java, when you create an object manually, you are responsible for everything:

```java
PaymentService paymentService = new PaymentService();
```

You handle the complete object journey:

- Create object
- Set required values
- Call required methods
- Clean up resources if needed

In Spring, we don't create important application objects manually. Instead, we tell Spring which classes/methods should be managed by the container.

```java
@Component
public class PaymentService { }
```

Or using @Bean:

```java
@Configuration
public class AppConfig {
    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }
}
```

Now this object becomes Spring's responsibility. Spring will:

- Discover the bean
- Create the object
- Inject dependencies
- Call lifecycle methods
- Keep the bean ready
- Destroy it when required

## 2. Simple Definition

Bean Lifecycle means the complete journey of a Spring-managed object from the moment Spring discovers its definition until the moment Spring destroys it.

A Spring-managed object is called a bean.

So whenever we say "bean lifecycle," we are talking about:

How Spring creates, prepares, manages, and destroys a bean.

## 3. Complete Bean Lifecycle Flow

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                        SPRING BEAN LIFECYCLE FLOW                           │
└─────────────────────────────────────────────────────────────────────────────┘

    ┌──────────────────┐
    │ 1. Container     │
    │    Starts        │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 2. Read Config   │
    │    & Annotations │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 3. BeanDefinition│
    │    Created       │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 4. Instantiate   │
    │    Bean Object   │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 5. Inject        │
    │    Dependencies  │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 6. Aware         │
    │    Interfaces    │
    │    Called        │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 7. BeanPostProc  │
    │    Before Init   │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 8. Initialization│
    │    Callbacks     │
    │ • @PostConstruct │
    │ • InitializingBean│
    │ • custom init    │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 9. BeanPostProc  │
    │    After Init    │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 10. Bean is      │
    │     READY TO USE │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 11. Destruction  │
    │     Callbacks    │
    │ • @PreDestroy    │
    │ • DisposableBean │
    │ • custom destroy │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ 12. Bean Removed │
    └──────────────────┘
```

## 4. Step-by-Step Breakdown

### Step 0: Spring Container Starts

When we write:

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
```

We are not passing an object of AppConfig. We are passing the class metadata of AppConfig. Spring uses reflection to inspect that class.

Spring checks:

- Does this class have @Configuration?
- Does it have @ComponentScan?
- Does it have @Bean methods?
- Which package should be scanned?
- Which bean definitions should be registered?

This is where the Spring container starts understanding the application structure.

### Step 1: Bean Definition is Read

Before creating objects, Spring first collects metadata about beans. This metadata is called a BeanDefinition.

A BeanDefinition contains:

| Property | Example |
| --- | --- |
| Bean name | paymentService |
| Bean class | PaymentService |
| Scope | singleton |
| Lazy | false |
| Dependencies | check later |
| Init method | check later |
| Destroy method | check later |

**Important:** At this stage, the object is NOT necessarily created. Spring is only registering information about the bean.

#### Why does Spring need BeanDefinition first?

Because Spring is not creating just one object. It is building a complete object network.

Example:

```text
OrderService depends on PaymentService
PaymentService depends on PaymentGateway
PaymentGateway depends on ApiClient
```

Spring first needs to know:

- Which classes should I manage?
- What is the bean name?
- What is the scope?
- What dependencies does it need?
- Is it lazy or eager?
- Does it have init or destroy methods?

Only after understanding this structure does Spring start creating objects.

#### Where does Spring read bean definitions from?

**Annotation-based configuration**

```java
@Configuration
@ComponentScan
public class AppConfig { }

@Component
public class PaymentService { }
```

**Java-based @Bean configuration**

```java
@Bean
public PaymentService paymentService() {
    return new PaymentService();
}
```

**XML-based configuration**

```xml
<bean id="paymentService" class="in.strikes.PaymentService"/>
```

### Step 2: Bean Object is Instantiated

After reading the bean definition, Spring creates the actual object. This step is called instantiation.

```java
@Component
public class PaymentService {
    public PaymentService() {
        System.out.println("PaymentService constructor called");
    }
}
```

When Spring creates this bean, the constructor is called.

**Critical Distinction:**

- Instantiation = object is created
- Initialization = object is prepared after creation

These two are NOT the same. A common beginner mistake is thinking that object creation and initialization are the same thing. They are different phases in the Spring lifecycle.

### Step 3: Dependencies are Injected

After or during object creation, Spring supplies the required dependencies. This is called dependency injection.

The exact timing depends on the type of injection.

#### Constructor Injection (Preferred)

```java
@Component
public class OrderService {
    private final PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**Flow:**

```text
Spring needs OrderService
    → OrderService needs PaymentService
        → Spring creates or finds PaymentService
            → Spring passes PaymentService into OrderService constructor
```

In constructor injection, the dependency is supplied while creating the object.

#### Setter Injection

```java
@Component
public class OrderService {
    private PaymentService paymentService;

    @Autowired
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**Flow:**

```text
Object is created first
    → Spring calls the setter method
        → Dependency is assigned
```

#### Field Injection

```java
@Component
public class OrderService {
    @Autowired
    private PaymentService paymentService;
}
```

**Flow:**

```text
Object is created first
    → Spring uses reflection
        → Dependency is injected directly into the field
```

Note: Field injection is common in examples, but constructor injection is generally preferred for clean and testable code.

### Step 4: Aware Interfaces are Called

Sometimes a bean wants to know something about the Spring container:

- What is my bean name?
- Which BeanFactory created me?
- Which ApplicationContext am I running inside?

Spring provides this information using Aware interfaces.

Aware interfaces are callback interfaces.

A callback means: Spring automatically calls a method at the right time.

#### Common Aware Interfaces

| Interface | Purpose |
| --- | --- |
| BeanNameAware | Gives the bean its name |
| ApplicationContextAware | Gives access to ApplicationContext |
| BeanFactoryAware | Gives access to BeanFactory |
| EnvironmentAware | Gives access to Environment |

#### Example

```java
@Component("myCustomBeanName")
public class MyBean implements BeanNameAware, ApplicationContextAware {

    public MyBean() {
        System.out.println("1. Constructor called");
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("2. BeanNameAware called");
        System.out.println("Bean name: " + name);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        System.out.println("3. ApplicationContextAware called");
        System.out.println("ApplicationContext: " + 
            applicationContext.getClass().getSimpleName());
    }
}
```

**Output:**

```text
1. Constructor called
2. BeanNameAware called
Bean name: myCustomBeanName
3. ApplicationContextAware called
ApplicationContext: AnnotationConfigApplicationContext
```

#### Why is it setBeanName() and not getBeanName()?

Because Spring is giving the bean some information.

The direction is: Spring Container → Bean

So Spring calls the method and sets the bean name into that bean. If we call setBeanName() manually, it will behave like a normal Java method. It will not change the actual name registered inside the Spring container.

#### Do we use Aware interfaces in normal business logic?

Usually, no. In normal service classes, we rarely need them.

They are more useful when building:

- Framework-level code
- Logging utilities
- Custom libraries
- Infrastructure components

For learning Spring lifecycle, they are useful because they show how Spring communicates container information to beans.

### Step 5: Initialization Callbacks

At this point:

- Bean object is created
- Dependencies are injected
- Aware callbacks are completed

Now Spring gives the bean a chance to run some startup logic before it is used. This is called the initialization phase.

Initialization logic can include:

- Validate configuration
- Initialize internal cache
- Check required setup
- Prepare resources
- Fill a HashMap with calculated values
- Log startup state

#### Option 1: @PostConstruct (Recommended)

```java
import jakarta.annotation.PostConstruct;

@Component
public class PaymentService {

    @PostConstruct
    public void init() {
        System.out.println("Bean initialized");
    }
}
```

@PostConstruct tells Spring: After dependencies are injected, call this method once.

This is clean and commonly used.

Spring 6 & 7: Use jakarta.annotation.PostConstruct

If using only Spring Core/Spring Context (not Spring Boot), you may need to add the Jakarta annotation dependency separately:

```xml
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>3.0.0</version>
</dependency>
```

#### Option 2: InitializingBean

```java
import org.springframework.beans.factory.InitializingBean;

@Component
public class PaymentService implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        System.out.println("Bean initialized");
    }
}
```

Here the bean implements a Spring interface. Spring calls afterPropertiesSet() during the initialization phase.

**Drawback:** Tightly couples our class with Spring.

#### Option 3: Custom initMethod

```java
@Configuration
public class AppConfig {

    @Bean(initMethod = "start")
    public PaymentService paymentService() {
        return new PaymentService();
    }
}

public class PaymentService {
    public void start() {
        System.out.println("Bean initialized");
    }
}
```

Here we define our own method name and tell Spring to call it after dependency injection.

**Advantage:** Keeps the class free from Spring-specific interfaces.

#### Initialization Callback Order

If all three are used together, the usual order is:

```text
1. @PostConstruct
2. InitializingBean.afterPropertiesSet()
3. custom initMethod
```

**Best Practice:** In real projects, we usually do not use all three together. Pick one clear approach.

### Step 6: Bean is Ready to Use

After initialization is complete, the bean is fully ready.

Now other beans can use it.

For singleton beans, Spring stores the object and reuses the same instance again and again.

```text
Bean created → dependencies injected → initialization completed → bean ready
```

At this stage, the bean can safely handle business logic.

### Step 7: Destruction Callbacks

When the Spring container is shutting down, Spring gives beans a chance to clean up resources. This is called the destruction phase.

Cleanup logic can include:

- Close database connection
- Stop background thread
- Release file handle
- Clear resource
- Flush pending data

#### Option 1: @PreDestroy (Recommended)

```java
import jakarta.annotation.PreDestroy;

@Component
public class PaymentService {

    @PreDestroy
    public void cleanup() {
        System.out.println("Cleaning up bean");
    }
}
```

Spring calls this method before destroying the bean.

#### Option 2: DisposableBean

```java
import org.springframework.beans.factory.DisposableBean;

@Component
public class PaymentService implements DisposableBean {

    @Override
    public void destroy() {
        System.out.println("Cleaning up bean");
    }
}
```

**Drawback:** Couples our class with Spring.

#### Option 3: Custom destroyMethod

```java
@Configuration
public class AppConfig {

    @Bean(destroyMethod = "stop")
    public PaymentService paymentService() {
        return new PaymentService();
    }
}

public class PaymentService {
    public void stop() {
        System.out.println("Cleaning up bean");
    }
}
```

#### Destruction Callback Order

If all three are used together, the usual order is:

```text
1. @PreDestroy
2. DisposableBean.destroy()
3. custom destroyMethod
```

**Best Practice:** In real projects, we usually choose one approach.

## 5. Singleton vs Prototype Lifecycle

### Singleton Bean Lifecycle (Default)

Singleton means: Spring creates only one object for that bean.

For singleton beans, Spring manages the complete lifecycle:

```text
creation → dependency injection → initialization → usage → destruction
```

Example:

```java
@Component
public class PaymentService {

    public PaymentService() {
        System.out.println("Constructor called");
    }

    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct called");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("@PreDestroy called");
    }
}
```

When the context starts:

```text
Constructor called
@PostConstruct called
```

When the context closes:

```text
@PreDestroy called
```

#### Singleton Characteristics

| Aspect | Behavior |
| --- | --- |
| Creation | Once at startup (eager by default) |
| Storage | Stored in singleton cache |
| Reuse | Same instance reused |
| Destruction | Spring destroys it on context close |

### Prototype Bean Lifecycle

Prototype means: A new object is created every time the bean is requested.

Example:

```java
@Component
@Scope("prototype")
public class PaymentService {

    public PaymentService() {
        System.out.println("Constructor called");
    }
}
```

Whenever we request this bean:

```java
PaymentService p1 = context.getBean(PaymentService.class);
PaymentService p2 = context.getBean(PaymentService.class);
```

Spring creates two different objects. p1 and p2 are different instances.

#### Prototype Lifecycle Flow

```text
Container starts
    → BeanDefinition is read
        → No object is usually created at startup
            → Client requests prototype bean
                → Spring creates a new object
                    → Dependencies are injected
                        → Aware callbacks are called
                            → Initialization callbacks are called
                                → Spring hands object to the client
                                    → Spring does NOT track it for normal destruction
```

#### Important Difference: Singleton vs Prototype

| Aspect | Singleton | Prototype |
| --- | --- | --- |
| Creation | Spring creates it | Spring creates it |
| Storage | Spring stores it | Spring does NOT store it |
| Reuse | Spring reuses it | New object each time |
| Destruction | Spring destroys it | Caller is responsible |
| Full Lifecycle | Yes | Only beginning |

Key Insight: Spring manages the beginning of the prototype lifecycle, but not the complete ending.

### Prototype Destruction Does NOT Happen Automatically

Example:

```java
@Component
@Scope("prototype")
public class PaymentService {

    public PaymentService() {
        System.out.println("Constructor called");
    }

    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct called");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("@PreDestroy called");
    }
}
```

**Main code:**

```java
AnnotationConfigApplicationContext context = 
    new AnnotationConfigApplicationContext(AppConfig.class);

PaymentService p1 = context.getBean(PaymentService.class);

context.close();
```

**Expected output:**

```text
Constructor called
@PostConstruct called
```

Usually, we will NOT see:

```text
@PreDestroy called
```

#### Why does @PreDestroy not run for prototype beans?

Because after Spring creates a prototype bean and gives it to the caller, Spring does not keep tracking that specific object.

For singleton beans, Spring knows:

"I created this one object. It is stored in my singleton cache. When the context closes, I can destroy it."

For prototype beans, Spring says:

"I created this object. I prepared it. I gave it to the caller. Now the caller may use it, store it, discard it, or create many more copies. I will not track all prototype instances for automatic destruction."

#### Why does Spring not keep a list of every prototype object?

Technically, Spring could keep references to every prototype object. But that would create a serious problem:

Prototype beans can be created many times.

If Spring stored every prototype object forever, those objects could not be garbage collected.

That could cause memory leaks.

So Spring does not fully manage prototype destruction.

#### Does context.close() destroy prototype beans?

Usually, no.

context.close() destroys singleton beans because Spring has them registered and tracked.

Prototype instances that were already handed out are not automatically destroyed.

So if a prototype bean holds expensive resources, we should clean them manually:

- Close the resource manually
- Use try-with-resources where possible
- Avoid putting heavy resource cleanup responsibility inside prototype beans

Note: Java can still garbage collect the prototype object when there are no references to it. But garbage collection and Spring destruction callbacks are not the same thing.

## 6. Important Cases & Pitfalls

### Case 1: Singleton Depends on Prototype

**Scenario:**

- OrderService is singleton
- PaymentSession is prototype
- OrderService depends on PaymentSession

Example:

```java
@Component
public class OrderService {

    private final PaymentSession paymentSession;

    public OrderService(PaymentSession paymentSession) {
        this.paymentSession = paymentSession;
    }
}
```

```java
@Component
@Scope("prototype")
public class PaymentSession {

    public PaymentSession() {
        System.out.println("PaymentSession created");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("PaymentSession destroyed");
    }
}
```

What happens?

Since OrderService is singleton, it is created only once.

At the time of creating OrderService, Spring creates one PaymentSession and injects it.

After that, the same PaymentSession instance remains inside OrderService.

So even though PaymentSession is prototype, it does not automatically mean a new object will be used for every method call inside the singleton.

**Question:** Will @PreDestroy be called on that prototype instance?

**Answer:** No, usually Spring will not automatically call @PreDestroy on that prototype instance.

**Reason:** PaymentSession is prototype scoped. Spring created it, initialized it, and handed it to OrderService. After that, Spring does not manage its full destruction lifecycle.

#### Key Learning

Prototype scope means: new object per request from Spring

It does NOT mean: new object every time a singleton method runs

If a singleton needs a fresh prototype instance every time, we need other approaches:

ObjectProvider

```java
@Component
public class OrderService {

    private final ObjectProvider<PaymentSession> paymentSessionProvider;

    public OrderService(ObjectProvider<PaymentSession> paymentSessionProvider) {
        this.paymentSessionProvider = paymentSessionProvider;
    }

    public void createOrder() {
        PaymentSession session = paymentSessionProvider.getObject();
        System.out.println(session);
    }
}
```

- Provider
- Lookup method injection
- applicationContext.getBean()

### Case 2: @Lazy and Bean Lifecycle

By default, singleton beans are eagerly created. That means: when the Spring container starts, singleton beans are usually created immediately.

But if we mark a singleton bean as @Lazy, Spring reads its bean definition but does not create the object immediately.

Example:

```java
@Component
@Lazy
public class PaymentService {

    public PaymentService() {
        System.out.println("PaymentService created");
    }
}
```

**Flow:**

```text
Container starts
    → BeanDefinition is registered
        → Object is NOT created yet
            → Bean is created only when someone asks for it
```

So for a lazy singleton:

- BeanDefinition is read at startup
- Actual object creation is delayed
- Dependency injection and initialization also happen later

### Case 3: @PostConstruct and Circular Dependency

**Problem:**

```java
@Component
public class A {
    B b;
    public A(B b) { }
}

@Component
public class B {
    A a;
    public B(A a) { }
}
```

Here Spring cannot create A without B, and it cannot create B without A. So constructor circular dependency fails.

Where can @PostConstruct help?

```java
@Component
public class A {
    B b;

    public A(B b) {
        this.b = b;
    }

    @PostConstruct
    public void setB() {
        b.setA(this);
    }
}
```

```java
@Component
public class B {
    A a;

    public void setA(A a) {
        this.a = a;
    }
}
```

## 7. Interview Questions & Answers

### Q1: What is the Spring Bean Lifecycle?

**Answer:** The Spring Bean Lifecycle is the complete journey of a Spring-managed object from the moment Spring discovers its definition until the moment Spring destroys it. It includes: BeanDefinition reading → object creation → dependency injection → Aware callbacks → BeanPostProcessor before initialization → initialization callbacks → BeanPostProcessor after initialization → bean ready → destruction callbacks.

### Q2: What is the difference between Instantiation and Initialization?

**Answer:**

Instantiation = object is created (constructor called)

Initialization = object is prepared after creation (e.g., @PostConstruct)

These are different phases in the Spring lifecycle.

### Q3: What are the different ways to define initialization logic?

**Answer:**

- @PostConstruct (recommended)
- InitializingBean.afterPropertiesSet()
- Custom initMethod in @Bean

Order if all used: @PostConstruct → InitializingBean → custom initMethod

### Q4: What are the different ways to define destruction logic?

**Answer:**

- @PreDestroy (recommended)
- DisposableBean.destroy()
- Custom destroyMethod in @Bean

Order if all used: @PreDestroy → DisposableBean.destroy() → custom destroyMethod

### Q5
