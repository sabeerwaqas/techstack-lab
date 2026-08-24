# Spring Boot HTTP Request Lifecycle

> A practical guide to understanding how an HTTP request travels through a Spring Boot application — from the client to the database and back.

---

## Table of Contents

1. [Overview](#overview)
2. [The Big Picture](#the-big-picture)
3. [HTTP Request](#http-request)
4. [Tomcat / Servlet Container](#tomcat--servlet-container)
5. [Servlet Filters](#servlet-filters)
6. [Spring Security Filter Chain](#spring-security-filter-chain)
7. [DispatcherServlet](#dispatcherservlet)
8. [HandlerMapping](#handlermapping)
9. [HandlerAdapter](#handleradapter)
10. [Controller](#controller)
11. [Service Layer](#service-layer)
12. [Repository Layer](#repository-layer)
13. [JPA and Hibernate](#jpa-and-hibernate)
14. [Database](#database)
15. [HttpMessageConverter](#httpmessageconverter)
16. [Filters vs Interceptors](#filters-vs-interceptors)
17. [Transactions](#transactions)
18. [Exception Handling](#exception-handling)
19. [Validation](#validation)
20. [Request Body Deserialization](#request-body-deserialization)
21. [Response Serialization](#response-serialization)
22. [Complete Request + Response Flow](#complete-request--response-flow)
23. [What Happens When Security Rejects a Request](#what-happens-when-security-rejects-a-request)
24. [Backend Developer Knowledge Checklist](#backend-developer-knowledge-checklist)
25. [Interview Explanation](#interview-explanation)
26. [Final Mental Model](#final-mental-model)
27. [Recommended Learning Order](#recommended-learning-order)

---

## Overview

When a client sends an HTTP request to a Spring Boot application, the request passes through several components before a response is returned.

A simplified lifecycle is:


Client
  ↓
Tomcat / Jetty / Undertow
  ↓
Servlet Filters
  ↓
Spring Security Filter Chain
  ↓
DispatcherServlet
  ↓
HandlerMapping
  ↓
HandlerAdapter
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
JPA / Hibernate
  ↓
Database
  ↓
JPA / Hibernate
  ↓
Repository
  ↓
Service
  ↓
Controller
  ↓
HttpMessageConverter
  ↓
HTTP Response
  ↓
Tomcat
  ↓
Client

The exact flow can vary depending on the application configuration.

For example:

- Security may not be configured.
- A request may not need a database.
- Interceptors may execute.
- Transactions may be involved.
- Exception handling may change the response flow.
- Different authentication mechanisms may use different security filters.

Therefore, the diagram should be treated as a mental model, not as a rigid list of steps that every request always follows.

---

## The Big Picture

Consider this request:

GET /api/customers/10 HTTP/1.1
Host: api.example.com
Authorization: Bearer <JWT>
Accept: application/json

The request travels approximately like this:

React / Browser / Postman
          │
          │ HTTP Request
          ▼
       Tomcat
          │
          ▼
   Servlet Filters
          │
          ▼
Spring Security Filter Chain
          │
          ▼
  DispatcherServlet
          │
          ▼
    HandlerMapping
          │
          ▼
    HandlerAdapter
          │
          ▼
 CustomerController
          │
          ▼
   CustomerService
          │
          ▼
 CustomerRepository
          │
          ▼
   JPA / Hibernate
          │
          ▼
      Database

The result then travels back:

Database
   │
   ▼
Hibernate / JPA
   │
   ▼
Repository
   │
   ▼
Service
   │
   ▼
Controller
   │
   ▼
HttpMessageConverter
   │
   ▼
JSON
   │
   ▼
Tomcat
   │
   ▼
Client

---

## HTTP Request

The lifecycle begins when a client sends an HTTP request.

The client can be:

- Browser
- React application
- Next.js application
- Mobile application
- Postman
- Another backend service
- CLI tool

Example:

GET /api/customers/10 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOi...
Accept: application/json

An HTTP request contains several important parts:

HTTP Request
│
├── HTTP Method
│   ├── GET
│   ├── POST
│   ├── PUT
│   ├── PATCH
│   └── DELETE
│
├── URL / Path
│
├── Headers
│
├── Query Parameters
│
├── Path Variables
│
└── Request Body

For example:

POST /api/customers?sendEmail=true
Authorization: Bearer <JWT>
Content-Type: application/json

{
  "name": "John",
  "email": "john@example.com"
}

Here:

POST
→ HTTP method

/api/customers
→ Request path

sendEmail=true
→ Query parameter

Authorization
→ HTTP header

Content-Type
→ HTTP header

JSON object
→ Request body

---

## Tomcat / Servlet Container

Spring Boot applications commonly use an embedded web server such as:

- Tomcat
- Jetty
- Undertow

Tomcat is a Servlet container and commonly acts as the HTTP server for a Spring Boot MVC application.

Conceptually:

Client
   ↓
HTTP Request
   ↓
Tomcat :8080
   ↓
Spring Boot Application

If you see:

Tomcat started on port 8080

it means the embedded Tomcat server is listening for incoming requests.

---

### What Does Tomcat Do?

Tomcat handles low-level HTTP and Servlet responsibilities.

Conceptually:

HTTP request
     ↓
Tomcat
     ↓
Servlet API
     ↓
Spring MVC

Tomcat deals with things such as:

- Listening for connections
- HTTP parsing
- Request/response handling
- Servlet lifecycle
- Request processing threads
- Network communication

It eventually invokes the appropriate Servlet.

In a Spring MVC application, the important Servlet is:

DispatcherServlet

---

### Servlet Container vs Spring

This distinction is important.

Tomcat
    ↓
Servlet Container

Spring MVC
    ↓
Framework running inside the Servlet container

Spring Boot does not replace the Servlet container.

Instead:

Spring Boot Application
        runs inside
Tomcat

---

## Servlet Filters

Before the request reaches "DispatcherServlet", it can pass through Servlet filters.

A filter implements:

jakarta.servlet.Filter

Example:

@Component
public class MyFilter implements Filter {

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain chain)
            throws IOException, ServletException {

        System.out.println("Before request");

        chain.doFilter(request, response);

        System.out.println("After request");
    }
}

The important concept is:

Request
   ↓
Filter
   ↓
chain.doFilter()
   ↓
Next component
   ↓
Response
   ↑
Filter

A filter can therefore execute logic:

BEFORE request
       ↓
Next component
       ↓
AFTER request

---

### What Are Filters Used For?

Common use cases include:

- Authentication
- Logging
- Request tracing
- CORS
- Security
- Request modification
- Response modification
- Metrics
- Correlation IDs

Example:

HTTP Request
     ↓
Logging Filter
     ↓
Security Filter
     ↓
DispatcherServlet

---

## Spring Security Filter Chain

If Spring Security is installed and configured, the request normally passes through Spring Security's filter chain.

This is one of the most important concepts for backend developers.

Conceptually:

HTTP Request
     ↓
Spring Security Filter Chain
     ↓
Authentication
     ↓
Authorization
     ↓
Continue OR Reject

Think of the security filter chain as a security checkpoint.

---

### What Happens Inside the Security Filter Chain?

The exact filters depend on your Spring Security configuration.

A JWT-based API may conceptually perform:

Request
   ↓
Security filters
   ↓
Extract credentials
   ↓
Validate JWT
   ↓
Create Authentication
   ↓
Store Authentication in SecurityContext
   ↓
Authorization check
   ↓
Continue to DispatcherServlet

Do not memorize one fixed filter order.

The actual chain depends on:

- Spring Security version
- Authentication mechanism
- Application configuration
- Enabled security features
- Custom filters

---

### Authentication vs Authorization

These two concepts must be clearly understood.

#### Authentication

Authentication asks:

«"Who are you?"»

For example:

JWT
 ↓
Validate token
 ↓
Extract user identity
 ↓
Authenticated user = user 123

Authentication establishes an identity.

---

#### Authorization

Authorization asks:

«"Are you allowed to perform this operation?"»

For example:

Authenticated user
       ↓
ROLE_USER
       ↓
GET /api/customers/10
       ↓
Allowed?

Or:

ROLE_USER
       ↓
DELETE /api/users/10
       ↓
Requires ADMIN
       ↓
403 Forbidden

Important distinction:

401 Unauthorized
→ Authentication problem

403 Forbidden
→ Authorization problem

---

### JWT Authentication Flow

Suppose the client sends:

Authorization: Bearer eyJhbGciOi...

A JWT authentication mechanism may:

Request
   ↓
Extract Authorization header
   ↓
Extract Bearer token
   ↓
Validate token
   ↓
Read claims
   ↓
Create Authentication
   ↓
Store authentication in SecurityContext
   ↓
Continue request

Conceptually:

JWT
 ↓
Authentication
 ↓
SecurityContext

The "SecurityContext" represents the security identity associated with the current execution context.

---

### What If Authentication Fails?

If the request requires authentication and authentication fails:

Request
   ↓
Security Filter Chain
   ↓
JWT invalid / missing
   ↓
Authentication failure handling
   ↓
401 Unauthorized

The request may never reach the controller.

---

### What If Authorization Fails?

The user might be authenticated but not permitted to perform the operation.

Example:

JWT valid
   ↓
User authenticated
   ↓
User role = USER
   ↓
Endpoint requires ADMIN
   ↓
Access denied
   ↓
403 Forbidden

Again, the controller may never execute.

---

## DispatcherServlet

If security allows the request to continue, Spring MVC takes over.

The central component is:

DispatcherServlet

"DispatcherServlet" is Spring MVC's Front Controller.

Think of it as the central traffic controller for Spring MVC.

Its job is to coordinate request processing.

Simplified:

Request
   ↓
DispatcherServlet
   ↓
Find handler
   ↓
Invoke handler
   ↓
Process result
   ↓
Create response

---

### Why Does DispatcherServlet Exist?

Instead of every controller directly dealing with the web server, Spring MVC has a central entry point.

                DispatcherServlet
                 /      |       \
                /       |        \
        Controller   Controller   Controller

This gives Spring a central place to coordinate:

- Handler mapping
- Handler invocation
- Argument resolution
- Return value handling
- Exception handling
- Message conversion
- View rendering

---

## HandlerMapping

Now Spring needs to answer:

«"Which controller method should handle this request?"»

This is the responsibility of "HandlerMapping".

Suppose you have:

@RestController
@RequestMapping("/api/customers")
public class CustomerController {

    @GetMapping("/{id}")
    public CustomerDTO getCustomer(
            @PathVariable Long id) {

        return customerService.getCustomer(id);
    }
}

The mapping is conceptually:

GET /api/customers/{id}
        ↓
CustomerController.getCustomer()

If the request is:

GET /api/customers/10

Spring needs to find:

CustomerController.getCustomer()

---

### How Does HandlerMapping Know About Controllers?

When the Spring application starts, Spring processes controller classes and their mappings.

For example:

@GetMapping("/customers/{id}")

creates a mapping between:

HTTP request pattern
        ↓
Controller method

Conceptually:

GET /api/customers/{id}
          ↓
CustomerController.getCustomer()

Spring maintains this mapping information internally.

When a request arrives, "HandlerMapping" looks for a matching handler.

---

### What Does HandlerMapping Consider?

Depending on the mapping mechanism and configuration, Spring can consider:

- HTTP method
- Request path
- Path variables
- Request parameters
- Headers
- "consumes"
- "produces"
- Other mapping conditions

Example:

@GetMapping(
    value = "/customers/{id}",
    produces = "application/json"
)

Spring can use these conditions when determining the appropriate handler.

---

### Example

Suppose you have:

@GetMapping("/customers")
public List<CustomerDTO> getCustomers() {
    ...
}

and:

@PostMapping("/customers")
public CustomerDTO createCustomer(
        @RequestBody CreateCustomerRequest request) {
    ...
}

Then:

GET /customers
    ↓
getCustomers()

POST /customers
    ↓
createCustomer()

Same path.

Different HTTP methods.

Spring can distinguish them.

---

### Important

"HandlerMapping" does not execute your controller.

Its responsibility is essentially:

"Which handler should process this request?"

Then:

HandlerMapping
      ↓
Handler found
      ↓
HandlerAdapter

---

## HandlerAdapter

Once Spring has found the appropriate handler, it needs a mechanism to invoke it.

That's where:

HandlerAdapter

comes in.

Conceptually:

HandlerMapping
      ↓
Find handler
      ↓
HandlerAdapter
      ↓
Invoke handler

---

### Why Do We Need HandlerAdapter?

Spring MVC supports different types of handlers.

"HandlerAdapter" provides an abstraction for invoking them.

For annotation-based controllers, Spring MVC uses an appropriate adapter to invoke methods such as:

@GetMapping(...)
public CustomerDTO getCustomer(...)

You usually don't interact directly with "HandlerAdapter".

It is mainly an internal Spring MVC mechanism.

---

### HandlerAdapter and Method Arguments

Consider:

@GetMapping("/customers/{id}")
public CustomerDTO getCustomer(
        @PathVariable Long id,
        @RequestHeader("Authorization") String token,
        @RequestParam String type) {
    ...
}

Spring needs to populate:

id
token
type

from the HTTP request.

Spring MVC uses argument resolvers and related infrastructure to turn HTTP request data into Java method arguments.

Conceptually:

HTTP Request
     ↓
Argument Resolution
     ↓
Java method parameters

---

### Common Controller Argument Sources

#### Path Variable

Request:

/api/customers/10

Controller:

@GetMapping("/customers/{id}")
public CustomerDTO get(
        @PathVariable Long id) {
}

Result:

id = 10

---

#### Query Parameter

Request:

/api/customers?status=ACTIVE

Controller:

@GetMapping("/customers")
public List<CustomerDTO> get(
        @RequestParam String status) {
}

Result:

status = ACTIVE

---

#### Request Body

Request:

{
  "name": "John",
  "email": "john@example.com"
}

Controller:

@PostMapping("/customers")
public CustomerDTO create(
        @RequestBody CreateCustomerRequest request) {
}

Spring converts the JSON into:

CreateCustomerRequest

---

#### Request Header

Request:

Authorization: Bearer <JWT>

Controller:

@GetMapping
public CustomerDTO get(
        @RequestHeader("Authorization") String authorization) {
}

---

## Controller

Now your controller method executes.

Example:

@RestController
@RequestMapping("/api/customers")
public class CustomerController {

    @GetMapping("/{id}")
    public CustomerDTO getCustomer(
            @PathVariable Long id) {

        return customerService.getCustomer(id);
    }
}

The controller represents the HTTP/API layer.

Generally, it should:

Receive HTTP request
        ↓
Extract request data
        ↓
Perform appropriate API-level validation
        ↓
Call service
        ↓
Return response

---

### What Should Not Usually Go in a Controller?

Avoid putting large amounts of business logic inside controllers.

Avoid:

@GetMapping("/{id}")
public CustomerDTO getCustomer(
        @PathVariable Long id) {

    // 100 lines of business logic

    // Complex calculations

    // Database operations

    // Business rules
}

Prefer:

Controller
    ↓
Service
    ↓
Repository

---

## Service Layer

The service layer normally contains business logic.

Example:

@Service
public class CustomerService {

    public CustomerDTO getCustomer(Long id) {

        Customer customer = customerRepository
                .findById(id)
                .orElseThrow(
                    () -> new ResourceNotFoundException(
                        "Customer not found"
                    )
                );

        return convertToDTO(customer);
    }
}

The service may handle:

- Business rules
- Validation
- Calculations
- Orchestration
- Multiple repository calls
- Transactions
- DTO conversion
- Domain operations

---

### Controller vs Service

A useful mental model:

Controller
→ "How do I communicate over HTTP?"

Service
→ "What should the application actually do?"

Example:

GET /customers/10
       ↓
Controller
       ↓
CustomerService.getCustomer(10)
       ↓
Business logic

---

## Repository Layer

The service may call a repository:

customerRepository.findById(id);

For example:

public interface CustomerRepository
        extends JpaRepository<Customer, Long> {
}

The repository provides persistence-related operations.

Typical operations:

find
save
update
delete
exists
count

---

## JPA and Hibernate

When using:

JpaRepository<Customer, Long>

Spring Data JPA provides repository functionality.

For example:

customerRepository.findById(10L);

You don't necessarily need to write SQL manually.

The conceptual architecture is:

Your Application
      ↓
Spring Data JPA
      ↓
JPA
      ↓
Hibernate
      ↓
JDBC
      ↓
Database

---

### JPA vs Hibernate

#### JPA

JPA is a Java persistence specification/API.

It defines concepts such as:

@Entity
@Id
@OneToMany
@ManyToOne
EntityManager

#### Hibernate

Hibernate is a popular implementation of JPA.

Therefore:

JPA
→ Specification/API

Hibernate
→ Implementation

A useful statement to remember:

«JPA defines the standard; Hibernate provides an implementation.»

---

### Hibernate and SQL

Suppose your code says:

customerRepository.findById(10L);

Hibernate may execute SQL conceptually similar to:

SELECT
    c.id,
    c.name,
    c.email
FROM customer c
WHERE c.id = 10;

The exact SQL depends on:

- Entity mapping
- Database dialect
- Hibernate version
- Fetch strategy
- Query
- Configuration

---

### Entity Mapping

Suppose:

@Entity
@Table(name = "customer")
public class Customer {

    @Id
    private Long id;

    private String name;

    private String email;
}

Hibernate maps:

Java object
    ↕
Database row

For example:

Customer Java object
       ↕
customer database table

---

## Database

The database executes the SQL.

For example:

SELECT *
FROM customer
WHERE id = 10;

The database might return:

id | name | email
---+------+------------------
10 | John | john@example.com

The result travels back through the persistence layer.

---

## HttpMessageConverter

Suppose the controller returns:

CustomerDTO

but the client expects JSON.

Spring can use an "HttpMessageConverter" to convert:

Java object
     ↓
JSON

Spring Boot commonly uses Jackson for JSON serialization and deserialization.

---

### Java Object → JSON

Example:

CustomerDTO(
    10,
    "John",
    "john@example.com"
)

becomes:

{
  "id": 10,
  "name": "John",
  "email": "john@example.com"
}

Conceptually:

Controller return value
        ↓
HttpMessageConverter
        ↓
Jackson
        ↓
JSON

---

### @RestController

When using:

@RestController

Spring treats controller return values as response bodies by default.

Example:

@GetMapping("/{id}")
public CustomerDTO getCustomer(
        @PathVariable Long id) {

    return customerService.getCustomer(id);
}

The returned object can be serialized into JSON.

---

### ResponseEntity

You can also explicitly control the HTTP response:

@GetMapping("/{id}")
public ResponseEntity<CustomerDTO> getCustomer(
        @PathVariable Long id) {

    CustomerDTO customer =
            customerService.getCustomer(id);

    return ResponseEntity.ok(customer);
}

"ResponseEntity" allows you to control:

HTTP status
Headers
Body

Example:

return ResponseEntity
        .status(HttpStatus.CREATED)
        .body(customer);

---

### Final HTTP Response

The response might look like:

HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 10,
  "name": "John",
  "email": "john@example.com"
}

The response travels through the servlet container and eventually reaches the client.

---

## Filters vs Interceptors

Spring MVC also provides interceptors.

An interceptor is different from a Servlet filter.

Conceptually:

Filter
   ↓
DispatcherServlet
   ↓
Interceptor
   ↓
Controller

Spring MVC interceptors commonly provide:

preHandle()
postHandle()
afterCompletion()

Conceptually:

Request
   ↓
preHandle()
   ↓
Controller
   ↓
postHandle()
   ↓
Response processing
   ↓
afterCompletion()

Typical use cases:

- Logging
- Auditing
- Request timing
- MVC-related checks
- Request-specific processing

Security should generally be handled by Spring Security rather than recreating authentication/authorization inside an interceptor.

---

### Filter vs Interceptor

Feature| Filter| Interceptor
Layer| Servlet| Spring MVC
Runs before DispatcherServlet?| Yes| No
Runs before controller?| Yes| Yes
Spring MVC specific?| No| Yes
Common use| Security, CORS, logging| MVC request processing
API| "Filter"| "HandlerInterceptor"

Mental model:

Servlet Container
       ↓
Filter
       ↓
DispatcherServlet
       ↓
Interceptor
       ↓
Controller

---

## Transactions

Suppose your service has:

@Transactional
public void createInvoice(...) {
    ...
}

Spring's transaction infrastructure can wrap the service method using proxies/AOP.

Conceptually:

Controller
    ↓
Service Proxy
    ↓
BEGIN TRANSACTION
    ↓
Service method
    ↓
Repository
    ↓
Database
    ↓
COMMIT

If an appropriate exception causes rollback:

BEGIN
  ↓
Database operations
  ↓
Exception
  ↓
ROLLBACK

This is why "@Transactional" is commonly associated with the service/business operation boundary.

---

### AOP and Proxies

Spring uses proxies and AOP-style infrastructure for many cross-cutting concerns.

Examples:

@Transactional
@Cacheable
@PreAuthorize
Custom aspects

Conceptually:

Service Proxy
      ↓
Cross-cutting logic
      ↓
Actual Service Method

For example:

Service Proxy
    ↓
Start transaction
    ↓
Call service method
    ↓
Commit / rollback

The important concept is:

«Spring can wrap your objects with proxies to execute additional behavior around method calls.»

---

## Exception Handling

What happens if something goes wrong?

For example:

customerRepository.findById(id)
    .orElseThrow(
        () -> new ResourceNotFoundException(
            "Customer not found"
        )
    );

The exception can be handled by Spring MVC's exception-handling infrastructure.

A common approach is:

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException ex) {

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(ex.getMessage()));
    }
}

The API can return:

HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "message": "Customer not found"
}

---

### Exception Flow

Conceptually:

Controller
    ↓
Service
    ↓
Repository
    ↓
Exception
    ↓
Spring MVC exception handling
    ↓
@RestControllerAdvice
    ↓
Error Response

---

## Validation

Request validation commonly happens near the controller boundary.

Example:

@PostMapping
public CustomerDTO createCustomer(
        @Valid @RequestBody CreateCustomerRequest request) {

    return customerService.createCustomer(request);
}

DTO:

public class CreateCustomerRequest {

    @NotBlank
    private String name;

    @Email
    private String email;
}

Conceptually:

HTTP JSON
    ↓
Deserialize JSON
    ↓
Create DTO
    ↓
Validate DTO
    ↓
Valid?
 ┌──┴───┐
No     Yes
↓       ↓
400    Controller

Validation errors can be handled globally through exception handling.

---

## Request Body Deserialization

Suppose the client sends:

{
  "name": "John",
  "email": "john@example.com"
}

Your controller expects:

CreateCustomerRequest

Spring uses an HTTP message converter to deserialize:

JSON
 ↓
Jackson
 ↓
Java object

This happens before your controller method executes.

---

## Response Serialization

The opposite happens for the response.

Java object
 ↓
Jackson
 ↓
JSON
 ↓
HTTP response

Remember:

REQUEST:

JSON → Java

RESPONSE:

Java → JSON

---

### Content-Type vs Accept

HTTP clients communicate representation information using headers.

Example:

Content-Type: application/json

means:

«"The request body I'm sending is JSON."»

While:

Accept: application/json

means:

«"I want the response in JSON."»

Remember:

Content-Type
→ What format am I sending?

Accept
→ What format do I want to receive?

---

### Path Variables vs Query Parameters

These are commonly confused.

#### Path Variable

Request:

GET /customers/10

Controller:

@GetMapping("/customers/{id}")
public CustomerDTO get(
        @PathVariable Long id) {
}

Result:

id = 10

---

#### Query Parameter

Request:

GET /customers?status=ACTIVE

Controller:

@GetMapping("/customers")
public List<CustomerDTO> get(
        @RequestParam String status) {
}

Result:

status = ACTIVE

---

### POST Request Lifecycle

Consider:

POST /api/customers
Content-Type: application/json

{
  "name": "John",
  "email": "john@example.com"
}

The flow becomes:

Client
 ↓
Tomcat
 ↓
Security Filters
 ↓
DispatcherServlet
 ↓
HandlerMapping
 ↓
HandlerAdapter
 ↓
@RequestBody
 ↓
Jackson JSON → DTO
 ↓
Validation
 ↓
Controller
 ↓
Service
 ↓
@Transactional
 ↓
Repository
 ↓
Hibernate
 ↓
SQL INSERT
 ↓
Database
 ↓
Result
 ↓
Service
 ↓
Controller
 ↓
DTO
 ↓
Jackson
 ↓
JSON
 ↓
HTTP Response
 ↓
Client

---

### Requests That Do Not Access the Database

Not every request goes all the way to the database.

For example:

@GetMapping("/health")
public String health() {
    return "OK";
}

The flow can be:

Client
 ↓
Tomcat
 ↓
Security
 ↓
DispatcherServlet
 ↓
HandlerMapping
 ↓
HandlerAdapter
 ↓
Controller
 ↓
HttpMessageConverter
 ↓
Client

There is no need for:

Service
Repository
Hibernate
Database

unless the endpoint actually needs them.

---

## What Happens When Security Rejects a Request

Example:

GET /api/admin/users
Authorization: Bearer invalid-token

Flow:

Client
 ↓
Tomcat
 ↓
Security Filter Chain
 ↓
Authentication fails
 ↓
Security failure handling
 ↓
401 Unauthorized
 ↓
Client

The request may never reach:

DispatcherServlet
Controller
Service
Repository

---

### What Happens When Authorization Fails?

Example:

Authenticated user
       ↓
ROLE_USER
       ↓
GET /api/admin/users
       ↓
Requires ROLE_ADMIN
       ↓
Access denied
       ↓
403 Forbidden

The controller may not execute.

---

### What Happens When No Controller Mapping Exists?

Suppose the client requests:

GET /api/does-not-exist

HandlerMapping cannot find an appropriate handler.

Conceptually:

Request
 ↓
DispatcherServlet
 ↓
HandlerMapping
 ↓
No matching handler
 ↓
404 Not Found

---

### Important Spring MVC Components

A backend developer should know these names at least conceptually:

Component| Main Responsibility
"DispatcherServlet"| Central Spring MVC request coordinator
"HandlerMapping"| Finds the appropriate handler
"HandlerAdapter"| Invokes the selected handler
"HandlerMethod"| Represents a controller method
"HandlerMethodArgumentResolver"| Resolves controller method parameters
"HttpMessageConverter"| Converts HTTP body ↔ Java objects
"HandlerExceptionResolver"| Helps resolve controller exceptions
"ViewResolver"| Resolves views for traditional MVC applications

You do not need to implement all of these manually.

The important thing is knowing why they exist and where they participate.

---

### REST API vs Traditional MVC

Spring MVC can be used for both REST APIs and traditional server-rendered applications.

#### REST API

@RestController

Response:

{
  "id": 10,
  "name": "John"
}

---

#### Traditional MVC

@Controller

Response may be:

HTML View

Traditional MVC can involve:

ViewResolver
    ↓
Template
    ↓
HTML

For modern Spring Boot REST APIs, focus more on:

@RestController
HttpMessageConverter
JSON

---

### Complete Request + Response Flow

┌─────────────────────────────┐
│           CLIENT            │
│ React / Browser / Postman   │
└──────────────┬──────────────┘
               │
               │ HTTP Request
               ▼
┌─────────────────────────────┐
│           TOMCAT            │
│       Servlet Container     │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│       SERVLET FILTERS       │
│ Logging / CORS / etc.       │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│   SPRING SECURITY FILTER    │
│           CHAIN             │
│                             │
│ Authentication              │
│ Authorization               │
│ SecurityContext             │
└──────────────┬──────────────┘
               │
               │ Allowed
               ▼
┌─────────────────────────────┐
│      DISPATCHERSERVLET      │
│       Front Controller      │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│       HANDLER MAPPING       │
│                             │
│ "Which controller method?"  │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│       HANDLER ADAPTER       │
│                             │
│ Invoke handler              │
│ Resolve arguments           │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│         CONTROLLER          │
│          HTTP Layer         │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│          SERVICE            │
│        Business Logic       │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│         REPOSITORY          │
│       Data Access Layer     │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│       JPA / HIBERNATE       │
│        Java ↔ SQL           │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│          DATABASE           │
│ PostgreSQL / MySQL / etc.   │
└──────────────┬──────────────┘
               │
               │ Result
               ▼
        RESPONSE FLOW
               │
               ▼
┌─────────────────────────────┐
│      HTTP MESSAGE           │
│        CONVERTER            │
│                             │
│       Java → JSON           │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│           TOMCAT            │
└──────────────┬──────────────┘
               │
               │ HTTP Response
               ▼
┌─────────────────────────────┐
│           CLIENT            │
└─────────────────────────────┘

---

### The Five Major Zones

A useful way to understand the entire lifecycle is to divide it into five zones.

#### Zone 1 — HTTP / Web Server

Client
 ↓
Tomcat

Question:

«How does the HTTP request enter the application?»

---

#### Zone 2 — Security

Filters
 ↓
Spring Security

Question:

«Is this request authenticated and authorized?»

---

#### Zone 3 — Spring MVC

DispatcherServlet
 ↓
HandlerMapping
 ↓
HandlerAdapter
 ↓
Controller

Question:

«Which controller should handle this request and how should it be invoked?»

---

#### Zone 4 — Application

Controller
 ↓
Service
 ↓
Repository

Question:

«What should the application actually do?»

---

#### Zone 5 — Persistence

Repository
 ↓
JPA
 ↓
Hibernate
 ↓
JDBC
 ↓
Database

Question:

«How does the application read and write persistent data?»

---

### What You Should NOT Memorize

Do not try to memorize every internal Spring class.

For example, you don't initially need to memorize:

DispatcherServlet.doDispatch()

or the exact order of every Spring Security filter.

Instead, understand:

Who receives the request?
        ↓
Who authenticates it?
        ↓
Who authorizes it?
        ↓
Who finds the controller?
        ↓
Who invokes the controller?
        ↓
Where does business logic live?
        ↓
How does data reach the database?
        ↓
How does the Java result become JSON?
        ↓
How are exceptions handled?

Once these concepts are clear, the implementation details become much easier.

---

## Backend Developer Knowledge Checklist

### HTTP

- [ ] HTTP request and response
- [ ] HTTP methods
- [ ] HTTP status codes
- [ ] Headers
- [ ] Cookies
- [ ] Query parameters
- [ ] Path variables
- [ ] Request body
- [ ] Response body
- [ ] Content-Type
- [ ] Accept
- [ ] JSON
- [ ] HTTPS basics

---

### Servlet / Web Layer

- [ ] What Tomcat does
- [ ] What a Servlet is
- [ ] What a Filter is
- [ ] Filter chain concept
- [ ] Request/response lifecycle
- [ ] Servlet container concept

---

### Spring MVC

- [ ] DispatcherServlet
- [ ] HandlerMapping
- [ ] HandlerAdapter
- [ ] HandlerMethod
- [ ] Controller
- [ ] Argument resolution
- [ ] HttpMessageConverter
- [ ] Exception handling
- [ ] "@RestController"
- [ ] "@RequestMapping"
- [ ] "@GetMapping"
- [ ] "@PostMapping"
- [ ] "@PutMapping"
- [ ] "@PatchMapping"
- [ ] "@DeleteMapping"

---

### Spring Security

- [ ] Authentication
- [ ] Authorization
- [ ] Security Filter Chain
- [ ] SecurityContext
- [ ] JWT authentication
- [ ] Roles
- [ ] Authorities
- [ ] 401 vs 403
- [ ] CORS basics
- [ ] CSRF basics
- [ ] Method-level security
- [ ] OAuth2 basics

---

### Application Layer

- [ ] Controller responsibility
- [ ] Service responsibility
- [ ] Repository responsibility
- [ ] DTOs
- [ ] Entity vs DTO
- [ ] Validation
- [ ] Global exception handling
- [ ] API error responses

---

### Persistence

- [ ] JPA
- [ ] Hibernate
- [ ] Entity
- [ ] Primary key
- [ ] Relationships
- [ ] Lazy vs eager loading
- [ ] JPQL
- [ ] Native SQL
- [ ] Transactions
- [ ] "@Transactional"
- [ ] N+1 query problem
- [ ] Entity lifecycle
- [ ] Persistence context
- [ ] Dirty checking
- [ ] JDBC basics
- [ ] Connection pooling

---

### Production-Level Knowledge

- [ ] Logging
- [ ] Metrics
- [ ] Distributed tracing basics
- [ ] Connection pooling
- [ ] Database transactions
- [ ] Caching
- [ ] API versioning
- [ ] Rate limiting
- [ ] Error handling
- [ ] Timeouts
- [ ] Retries
- [ ] Idempotency
- [ ] Health checks
- [ ] Graceful shutdown
- [ ] Observability

---

## Interview Explanation

If an interviewer asks:

«"What happens when an HTTP request reaches your Spring Boot application?"»

A strong answer is:

«The request first reaches the embedded servlet container, such as Tomcat. It can pass through Servlet filters and, when Spring Security is configured, through the Spring Security filter chain, where authentication and authorization checks occur.

If the request is allowed to continue, it reaches Spring MVC's "DispatcherServlet". "DispatcherServlet" uses "HandlerMapping" to find the controller method that matches the HTTP method and request path.

A "HandlerAdapter" then invokes the selected controller method, while Spring resolves things such as path variables, query parameters, headers, and request bodies.

The controller normally delegates business logic to the service layer. The service may call a repository, which uses Spring Data JPA and JPA/Hibernate to communicate with the database.

The result travels back to the controller. Spring's "HttpMessageConverter", commonly using Jackson for JSON, serializes the Java object into JSON. The servlet container then sends the HTTP response back to the client.

Exceptions can be handled through Spring MVC's exception-handling infrastructure, commonly using "@ExceptionHandler" and "@RestControllerAdvice".»

---

## Final Mental Model

If you remember only one thing, remember this:

                    HTTP REQUEST
                         ↓
                      TOMCAT
                         ↓
                      FILTERS
                         ↓
               SPRING SECURITY
                         ↓
                 DISPATCHERSERVLET
                         ↓
                  HANDLER MAPPING
                         ↓
                  HANDLER ADAPTER
                         ↓
                    CONTROLLER
                         ↓
                      SERVICE
                         ↓
                    REPOSITORY
                         ↓
                  JPA / HIBERNATE
                         ↓
                     DATABASE
                         │
                         │
                         ▼
                       RESULT
                         │
                         ▼
                    CONTROLLER
                         ↓
              HTTP MESSAGE CONVERTER
                         ↓
                       JSON
                         ↓
                      TOMCAT
                         ↓
                      CLIENT

The simplest responsibility-based mental model is:

Tomcat
→ Receives HTTP

Servlet Filters
→ Pre-process the request/response

Spring Security
→ Authentication + Authorization

DispatcherServlet
→ Coordinates Spring MVC

HandlerMapping
→ Finds the appropriate controller method

HandlerAdapter
→ Invokes the selected handler

Controller
→ Handles HTTP/API concerns

Service
→ Handles business logic

Repository
→ Handles persistence operations

JPA / Hibernate
→ Maps Java objects to database operations

Database
→ Stores and retrieves data

HttpMessageConverter
→ Converts HTTP body ↔ Java objects

Exception Handling
→ Converts failures into controlled HTTP responses

---

### One-Line Summary

«Client sends HTTP → Tomcat receives it → Filters/Security process it → DispatcherServlet coordinates it → HandlerMapping finds the controller → HandlerAdapter invokes it → Controller calls Service → Service calls Repository → Repository uses Hibernate → Hibernate queries Database → Result comes back through each layer → Jackson converts to JSON → Response sent to client.»

---

## Recommended Learning Order

If you are learning Spring Boot backend development, understand these concepts in this order:

1. HTTP fundamentals
        ↓
2. Tomcat / Servlet basics
        ↓
3. Filters
        ↓
4. DispatcherServlet
        ↓
5. Controllers
        ↓
6. HandlerMapping
        ↓
7. HandlerAdapter
        ↓
8. Request/Response conversion
        ↓
9. Service layer
        ↓
10. Repository layer
        ↓
11. JPA / Hibernate
        ↓
12. Transactions
        ↓
13. Exception handling
        ↓
14. Spring Security
        ↓
15. JWT / OAuth2
        ↓
16. Production concerns

The goal is not to memorize Spring internals.

The goal is to be able to look at a request such as:

GET /api/invoices/123

and mentally trace:

HTTP
 ↓
Security
 ↓
DispatcherServlet
 ↓
Controller
 ↓
Service
 ↓
Repository
 ↓
Hibernate
 ↓
Database
 ↓
Hibernate
 ↓
Service
 ↓
Controller
 ↓
JSON
 ↓
HTTP Response

Once this flow becomes intuitive, many Spring Boot concepts become much easier to understand.
