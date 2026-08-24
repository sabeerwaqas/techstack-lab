# Spring Boot HTTP Request Lifecycle

> A concise reference for understanding how HTTP requests flow through a Spring Boot application.

## The Complete Flow

```
HTTP Request
    ↓
Tomcat (Servlet Container)
    ↓
Servlet Filters
    ↓
Spring Security Filter Chain
    ↓
DispatcherServlet (Front Controller)
    ↓
HandlerMapping (Locate handler)
    ↓
HandlerAdapter (Invoke handler)
    ↓
Controller
    ↓
Service
    ↓
Repository
    ↓
JPA/Hibernate
    ↓
Database
    ↓
[Response Path]
    ↓
HttpMessageConverter (Java → JSON)
    ↓
HTTP Response
```

## Key Components

### Tomcat / Servlet Container
- Embedded web server handling HTTP protocol and Servlet lifecycle
- Listens for connections, parses HTTP, manages request threads
- Spring Boot runs inside it, not replacing it

### Servlet Filters
- Implement `jakarta.servlet.Filter`
- Execute before `DispatcherServlet`
- Can perform pre/post request processing
- Common uses: logging, CORS, request modification

```java
@Component
public class LoggingFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        // Before
        chain.doFilter(req, res);
        // After
    }
}
```

**Filter Order:** Filter → DispatcherServlet → Interceptor → Controller

### Spring Security Filter Chain
- Handles authentication and authorization
- **Authentication:** "Who are you?" (401 Unauthorized if fails)
- **Authorization:** "Are you allowed?" (403 Forbidden if fails)
- Stores identity in `SecurityContext`

```
Request → Extract credentials → Validate → Create Authentication → 
SecurityContext → Authorization check → Continue OR Reject
```

**Common JWT Flow:**
```
Extract Bearer token → Validate → Extract claims → 
Create Authentication → SecurityContext → Continue
```

**Key Distinction:**
- 401 Unauthorized → Authentication failed
- 403 Forbidden → Authorization failed

### DispatcherServlet
- Spring MVC's Front Controller
- Central coordinator for request processing
- Finds handler, invokes it, handles exceptions

### HandlerMapping
- Maps HTTP requests to controller methods
- Considers: HTTP method, path, path variables, parameters, headers
- Does NOT execute—only finds the handler

```java
GET /api/customers/10  →  CustomerController.getCustomer(10)
POST /api/customers    →  CustomerController.createCustomer()
```

### HandlerAdapter
- Invokes the selected handler
- Resolves method arguments from HTTP request
- Handles argument resolution and return value processing

**Argument Sources:**
- `@PathVariable`: `/customers/{id}` → `id = 10`
- `@RequestParam`: `?status=ACTIVE` → `status = ACTIVE`
- `@RequestBody`: JSON body → deserialized object
- `@RequestHeader`: HTTP headers

### Controller
- HTTP/API layer
- Should be thin—avoid large business logic
- Receive request → extract data → call service → return response

```java
@RestController
@RequestMapping("/api/customers")
public class CustomerController {
    
    @GetMapping("/{id}")
    public CustomerDTO getCustomer(@PathVariable Long id) {
        return customerService.getCustomer(id);
    }
}
```

### Service Layer
- Business logic and orchestration
- Handles validation, calculations, multiple repository calls
- Transaction boundaries
- DTO conversion

```java
@Service
public class CustomerService {
    
    @Transactional
    public CustomerDTO getCustomer(Long id) {
        Customer customer = customerRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Not found"));
        return convertToDTO(customer);
    }
}
```

### Repository Layer
- Data access abstraction
- Spring Data JPA provides CRUD operations
- No need to write SQL manually

```java
public interface CustomerRepository extends JpaRepository<Customer, Long> {
}
```

### JPA / Hibernate
**JPA:** Specification/Standard API  
**Hibernate:** Implementation

Entity mapping:
```java
@Entity
@Table(name = "customer")
public class Customer {
    @Id
    private Long id;
    private String name;
}
```

Hibernate translates `customerRepository.findById(10L)` into:
```sql
SELECT * FROM customer WHERE id = 10;
```

### HttpMessageConverter
- Serializes/deserializes HTTP bodies
- Commonly Jackson for JSON
- Automatically converts: JSON ↔ Java objects

**Request:** JSON → Jackson → Java object  
**Response:** Java object → Jackson → JSON

## Interceptors vs Filters

| Aspect | Filter | Interceptor |
|--------|--------|-------------|
| Layer | Servlet | Spring MVC |
| Runs before DispatcherServlet | Yes | No |
| API | `Filter` | `HandlerInterceptor` |
| Common use | Security, CORS, logging | MVC-specific processing |
| Methods | `doFilter()` | `preHandle()`, `postHandle()`, `afterCompletion()` |

## Transactions & AOP

`@Transactional` uses Spring proxies/AOP:

```
Service Proxy
    ↓
BEGIN TRANSACTION
    ↓
Service method
    ↓
COMMIT / ROLLBACK
```

Other proxy-based features: `@Cacheable`, `@PreAuthorize`, custom aspects

## Exception Handling

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(ex.getMessage()));
    }
}
```

**Exception Flow:** Controller → Service → Repository → Exception → 
Exception Handler → Error Response

## Request Validation

```java
@PostMapping
public CustomerDTO create(
        @Valid @RequestBody CreateCustomerRequest request) {
    return customerService.create(request);
}

public class CreateCustomerRequest {
    @NotBlank
    private String name;
    
    @Email
    private String email;
}
```

**Flow:** JSON → Deserialize → Create DTO → Validate → 
Valid? → Yes: Controller | No: 400 Bad Request

## Requests Without Database Access

Not every request hits the database:

```java
@GetMapping("/health")
public String health() {
    return "OK";  // No service, repository, or database needed
}
```

**Flow:** Client → Tomcat → Security → DispatcherServlet → 
HandlerMapping → HandlerAdapter → Controller → Response

## Security Rejection Scenarios

### Authentication Fails
```
Request → Security Filter → Invalid JWT → 401 Unauthorized
```
Controller never executes.

### Authorization Fails
```
Authenticated user (ROLE_USER) → DELETE /api/admin/users 
(requires ROLE_ADMIN) → 403 Forbidden
```
Controller never executes.

### No Handler Mapping Found
```
GET /api/does-not-exist → No matching handler → 404 Not Found
```

## Five Zones Mental Model

1. **HTTP/Web Server Zone:** Tomcat receives HTTP
2. **Security Zone:** Filters + Spring Security (Authentication/Authorization)
3. **Spring MVC Zone:** DispatcherServlet → HandlerMapping → HandlerAdapter
4. **Application Zone:** Controller → Service → Repository
5. **Persistence Zone:** JPA/Hibernate → Database

**Key Questions to Ask:**
- How does the request enter? (Tomcat)
- Is it authenticated/authorized? (Security)
- Which controller handles it? (HandlerMapping)
- How is it invoked? (HandlerAdapter)
- What business logic runs? (Service)
- How is data persisted? (Repository + Hibernate)
- How is the result converted to HTTP? (HttpMessageConverter)

## Important Spring MVC Components

| Component | Responsibility |
|-----------|-----------------|
| `DispatcherServlet` | Central coordinator |
| `HandlerMapping` | Find the handler |
| `HandlerAdapter` | Invoke the handler |
| `HandlerMethodArgumentResolver` | Resolve method parameters |
| `HttpMessageConverter` | Convert body ↔ Java objects |
| `HandlerExceptionResolver` | Resolve exceptions |

## REST Controller Patterns

### Request/Response Lifecycle
```java
@PostMapping("/customers")
public ResponseEntity<CustomerDTO> create(
        @Valid @RequestBody CreateCustomerRequest req) {
    
    CustomerDTO created = customerService.create(req);
    return ResponseEntity.status(HttpStatus.CREATED)
        .body(created);
}
```

**Flow:**
- Request: JSON → Jackson → DTO validation → Controller
- Response: DTO → Jackson → JSON → HTTP response

### Path vs Query Parameters
```java
GET /customers/10           // Path variable:  @PathVariable Long id
GET /customers?status=ACTIVE // Query param:   @RequestParam String status
```

## Content Negotiation

```
Request:
Content-Type: application/json  → "I'm sending JSON"
Accept: application/json        → "I want JSON back"

Response:
Content-Type: application/json  → "I'm sending JSON"
```

## POST Request Lifecycle Example

```
JSON Input
    ↓
@RequestBody → Jackson deserializes → DTO
    ↓
@Valid → Bean Validation checks DTO
    ↓
@Transactional begins
    ↓
Controller calls Service
    ↓
Service calls Repository
    ↓
Repository uses Hibernate
    ↓
Hibernate executes SQL INSERT
    ↓
Database persists data
    ↓
Transaction commits
    ↓
Entity returned to Controller
    ↓
Jackson serializes → JSON
    ↓
HTTP 200/201 response
```

## Common Pitfalls for Senior Developers

1. **Business logic in controller** → Move to service
2. **N+1 queries** → Use eager loading or batch fetching
3. **Lazy loading outside transaction** → Use `@Transactional` or explicit fetching
4. **Forgetting to validate** → Use `@Valid` and `@RestControllerAdvice`
5. **Missing exception handling** → Implement global exception handlers
6. **Security too permissive** → Verify role-based access control
7. **Not handling timeouts** → Add timeout configs for external calls
8. **Loose transaction boundaries** → Keep `@Transactional` at service level
9. **Not considering concurrent requests** → Remember servlet container handles multiple threads
10. **Connection pool exhaustion** → Monitor connection usage and timeouts

## What Senior Developers Must Know

- ✓ Complete request-to-response flow
- ✓ When security checks execute and what happens on failure (401 vs 403)
- ✓ HandlerMapping and HandlerAdapter distinguish
- ✓ Controller vs Service responsibilities
- ✓ Transaction boundaries and rollback scenarios
- ✓ Exception handling integration with `@RestControllerAdvice`
- ✓ Request/response serialization with Jackson
- ✓ Lazy loading pitfalls with Hibernate (N+1 problem)
- ✓ Interceptor vs Filter use cases
- ✓ When requests bypass database (health checks, caching)
- ✓ Performance implications of each layer (connection pooling, N+1, eager/lazy loading)
- ✓ Authentication vs Authorization
- ✓ Thread safety in servlet container (concurrent requests)
- ✓ Spring MVC argument resolution mechanisms
- ✓ How AOP and proxies enable `@Transactional`, `@Cacheable`, security
- ✓ Validation frameworks and error response patterns
- ✓ REST API principles and HTTP status codes
