## Acknowledgement

These notes are based on and adapted from the original Servlet course material available in the [Spring Framework Full Course repository](https://github.com/adityatandon15/Spring-Framework-Full-Course/blob/main/Lecture%2014/notes/Notes.pdf).

Credit for the original content belongs to its respective author. The material has been condensed and reorganized for personal learning, reference, and revision purposes.

Please refer to the original source for the complete material and context.

---

# Java Servlets — Complete Notes

## Table of Contents

1. [What Is a Servlet?](#1-what-is-a-servlet)
2. [Why a Normal Java Program Is Not Enough](#2-why-a-normal-java-program-is-not-enough)
3. [Static Websites vs. Dynamic Websites](#3-static-websites-vs-dynamic-websites)
4. [What Is Apache Tomcat?](#4-what-is-apache-tomcat)
5. [What Is a Servlet Container?](#5-what-is-a-servlet-container)
6. [The Servlet Request–Response Flow](#6-the-servlet-requestresponse-flow)
7. [Creating and Mapping a Servlet](#7-creating-and-mapping-a-servlet)
8. [Servlet Lifecycle](#8-servlet-lifecycle)
9. [When Is a Servlet Created?](#9-when-is-a-servlet-created)
10. [Query Parameters vs. JSON Request Bodies](#10-query-parameters-vs-json-request-bodies)
11. [External Tomcat](#11-external-tomcat)
12. [What Is a WAR File?](#12-what-is-a-war-file)
13. [Why Is the Servlet API Dependency `provided`?](#13-why-is-the-servlet-api-dependency-provided)
14. [Running Tomcat on macOS/Linux](#14-running-tomcat-on-macoslinux)
15. [Running Tomcat on Windows](#15-running-tomcat-on-windows)
16. [Troubleshooting: Port 8080 Is Already in Use](#16-troubleshooting-port-8080-is-already-in-use)
17. [Embedded Tomcat in Spring Boot](#17-embedded-tomcat-in-spring-boot)
18. [Traditional Servlets vs. Spring MVC](#18-traditional-servlets-vs-spring-mvc)
19. [Servlet vs. Spring MVC Controller](#19-servlet-vs-spring-mvc-controller)
20. [Complete Mental Model](#20-complete-mental-model)

---

## 1. What Is a Servlet?

A **Servlet is a Java class used to handle web requests and generate web responses**. In a typical Servlet application, a Servlet container such as Apache Tomcat manages the Servlet and invokes it when a matching request arrives.

A Servlet connects HTTP requests with Java application logic.

For example, when a browser requests:

```http
GET /hello HTTP/1.1
Host: localhost:8080
```

The browser is not directly calling `HelloServlet.java` or its `doGet()` method. Instead:

1. The browser sends an HTTP request.
2. Tomcat receives the request.
3. Tomcat identifies the appropriate Servlet.
4. Tomcat invokes the Servlet's relevant method.
5. The Servlet prepares a response.
6. Tomcat sends the HTTP response back to the browser.

### Why Do We Need Servlets?

Java can handle network communication through classes such as `ServerSocket`, but building a complete web application using low-level networking requires manually handling:

- Reading and parsing HTTP requests.
- Extracting URLs, headers, query parameters, and request bodies.
- Identifying HTTP methods such as GET and POST.
- Creating HTTP responses, status codes, and headers.
- Managing connections, multiple requests, and threads.
- Keeping the server running continuously.

Servlets provide a standard way to handle web requests without requiring developers to implement all this low-level HTTP processing themselves.

**Core idea:** The container handles the web infrastructure, while the developer implements application logic.

---

## 2. Why a Normal Java Program Is Not Enough

A normal Java program commonly starts from:

```java
public static void main(String[] args)
```

It executes its instructions and may terminate after completing its work.

A web application, however, must remain available to receive requests from multiple users continuously.

For example:

```text
User 1 ──┐
User 2 ──┼──> Web Server ──> Application Logic
User 3 ──┘
```

A Servlet-based application runs within a server environment that handles incoming HTTP requests and invokes the appropriate application code.

## 3. Static Websites vs. Dynamic Websites

### Static Website

A static website returns existing files without requiring application-specific Java logic to generate their contents.

Example request:

```http
GET /index.html
```

Typical static resources include:

- HTML
- CSS
- JavaScript
- Images
- Videos

The server returns the requested resource.

### Dynamic Website

A dynamic website generates a response based on the request, application logic, or available data.

Example:

```http
GET /user?id=101
```

The application may:

1. Read the user ID.
2. Retrieve user information from a database.
3. Prepare the result.
4. Send the response.

For example:

```text
/user?id=101  → Name: Rahul
/user?id=102  → Name: Priya
```

Servlets allow Java application logic to process these requests and generate appropriate responses.

---

## 4. What Is Apache Tomcat?

**Apache Tomcat is a Java-based web server and Servlet container.**

A Servlet container provides the environment required to create, initialize, manage, and execute Servlets.

Tomcat acts as the bridge between the HTTP world and Java application code.

### Responsibilities of Tomcat

Tomcat manages:

1. Servlet creation and initialization.
2. Servlet lifecycle methods.
3. Request and response objects.
4. URL-to-Servlet mapping.
5. Invocation of Servlet methods.
6. Request-processing infrastructure and thread management.
7. Sending HTTP responses back to clients.
8. Hosting multiple web applications.

The developer normally does not instantiate or invoke the Servlet manually. Tomcat manages its lifecycle and invokes it when appropriate.

### Tomcat's Default Port

Tomcat commonly listens on port `8080`.

Consider:

```text
http://localhost:8080/hello
```

| Component | Meaning |
|---|---|
| `http://` | HTTP protocol |
| `localhost` | The current machine |
| `8080` | Port on which Tomcat listens |
| `/hello` | Requested path |

The browser sends a request to the server. Tomcat receives the request and identifies which application and Servlet should handle it.

**Important:** The Servlet does not directly listen on port `8080`. Tomcat listens on the port; the Servlet executes inside Tomcat.

---

## 5. What Is a Servlet Container?

A container manages application objects and provides the environment in which they execute.

In the Servlet ecosystem:

```text
Spring Core              Servlet World
------------             -----------------
Spring Container         Servlet Container
Manages Spring beans     Manages Servlets
```

Tomcat is a Servlet container because it manages Servlet objects and their execution.

A Servlet container typically controls:

- Servlet object creation.
- `init()` invocation.
- `service()` invocation.
- HTTP-specific request handling through `doGet()`, `doPost()`, and related methods.
- `destroy()` invocation.
- Request and response object creation.
- URL mapping.
- Request-processing threads.
- Response delivery.

### Inversion of Control (IoC)

In an ordinary Java program, our code usually creates an object and invokes its methods:

```java
HelloServlet servlet = new HelloServlet();
servlet.doGet(request, response);
```

In a Servlet application, Tomcat normally creates and manages the Servlet, then invokes its methods.

This is an example of **Inversion of Control (IoC)**: control over object creation and method invocation belongs to the container rather than the application code.

| Environment | Object managed by |
|---|---|
| Ordinary Java application | Application code |
| Servlet application | Servlet container, such as Tomcat |
| Spring application | Spring container, for Spring-managed beans |

---

## 6. The Servlet Request–Response Flow

The complete request flow is:

```text
Browser / Client
       |
       | HTTP Request
       v
     Tomcat
       |
       v
Identify Web Application
       |
       v
Find Matching Servlet
       |
       v
Create or provide request/response objects
       |
       v
Invoke Servlet Method
       |
       v
Servlet Processes Request
       |
       v
Prepare HTTP Response
       |
       v
     Tomcat
       |
       | HTTP Response
       v
Browser / Client
```

Tomcat converts the incoming HTTP request into Java-friendly request and response objects.

### `HttpServletRequest`

The request object provides information about the incoming request, such as:

- URL and requested path.
- HTTP method.
- Headers.
- Query parameters.
- Request body.
- Cookies.

Example request:

```http
GET /users?name=Rahul&email=rahul@test.com
```

The query parameters are:

```text
name  = Rahul
email = rahul@test.com
```

### `HttpServletResponse`

The response object is used by the Servlet to prepare the HTTP response, including the information returned to the client.

The Servlet processes the request and writes the required response. Tomcat handles sending the HTTP response to the client.

---

## 7. Creating and Mapping a Servlet

A common way to implement an HTTP Servlet is to extend `HttpServlet`.

```java
@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(
            HttpServletRequest request,
            HttpServletResponse response) {

        // Logic for handling GET requests
    }
}
```

### Important Components

- `HttpServlet`: Provides HTTP-specific Servlet behavior.
- `@WebServlet("/hello")`: Maps the Servlet to the `/hello` URL pattern.
- `HttpServletRequest`: Provides incoming request information.
- `HttpServletResponse`: Supports the outgoing response.
- `doGet()`: Handles HTTP GET requests through the `HttpServlet` request-processing mechanism.

When a matching GET request reaches Tomcat, Tomcat identifies this mapping and invokes the appropriate Servlet method.

The browser does not call `doGet()` directly. It sends an HTTP request to the server.

---

## 8. Servlet Lifecycle

The **Servlet lifecycle** describes the journey of a Servlet object, from initialization to removal from service.

```text
Servlet Object Creation
         |
         v
       init()
         |
         v
      service()
         |
         v
doGet() / doPost() / ...
         |
         v
      destroy()
```

Tomcat manages the Servlet lifecycle.

### 8.1 Constructor

The Servlet object's constructor executes when the object is created.

Under the usual default behavior, Tomcat creates the Servlet object when its first matching request arrives. Servlet startup configuration can change when the object is loaded.

### 8.2 `init()`

The container calls `init()` when the Servlet is initialized.

It normally runs **once per Servlet instance**, not once per request.

Typical uses include:

- Loading configuration.
- Initializing helper objects.
- Preparing caches.
- Initializing expensive resources.
- Setting up resource-related components.

Example lifecycle:

```text
First request
     |
     v
Constructor executes
     |
     v
init()
     |
     v
service()
     |
     v
doGet()
```

For a subsequent request to the same Servlet instance:

```text
Second request
     |
     v
service()
     |
     v
doGet()
```

The constructor and `init()` do not execute again for every request.

### 8.3 `service()`

The container invokes `service()` for incoming requests.

For an `HttpServlet`, the inherited `service()` method examines the HTTP method and dispatches the request to the corresponding method.

Conceptually:

```text
service()
   |
   +── GET     → doGet()
   |
   +── POST    → doPost()
   |
   +── PUT     → doPut()
   |
   +── DELETE  → doDelete()
```

Because `HttpServlet` already implements this dispatching behavior, application developers normally override the relevant `do...()` methods instead of overriding `service()`.

### 8.4 HTTP-Specific Methods

| Method | Typical purpose |
|---|---|
| `doGet()` | Retrieve or read data |
| `doPost()` | Submit data, commonly to create a resource |
| `doPut()` | Update or replace an existing resource |
| `doDelete()` | Delete a resource |

These are typical usage patterns, rather than restrictions on every possible implementation.

### 8.5 `destroy()`

The container calls `destroy()` when the Servlet is being removed from service.

Possible situations include:

- Tomcat is stopping.
- The web application is undeployed.
- The application is reloaded.
- A new deployment replaces the old application.

Typical uses include:

- Releasing resources.
- Closing resources owned by the Servlet.
- Flushing pending data.
- Stopping associated background tasks.
- Releasing memory-heavy objects.

`destroy()` is called before the Servlet instance is removed from service. Afterward, the object can become eligible for garbage collection when no longer referenced.

### 8.6 Lifecycle Summary

| Method or stage | Purpose | Typical frequency |
|---|---|---|
| Constructor | Creates the object | Once per instance |
| `init()` | Initializes the Servlet | Once per instance |
| `service()` | Processes incoming requests | Once per request |
| `doGet()` / `doPost()` / etc. | Handles the corresponding HTTP method | When dispatched for that request |
| `destroy()` | Performs cleanup before removal | Once before removal |

### Critical Lifecycle Concept: One Servlet, Many Requests

The usual model is:

```text
One Servlet instance
      |
      +── Request 1
      +── Request 2
      +── Request 3
      +── Request N
```

Tomcat normally reuses the same Servlet instance for multiple requests.

**Thread-safety implication:** Multiple requests may be processed concurrently. Avoid storing request-specific or user-specific mutable data in Servlet instance variables, because different requests could interfere with each other.

Keep request-specific data local to the request-processing method unless shared state is deliberately managed safely.

---

## 9. When Is a Servlet Created?

Under the usual default configuration, Tomcat initializes a Servlet when its first matching request arrives.

```text
First request to /hello
         |
         v
Create Servlet instance
         |
         v
Call init()
         |
         v
Handle request
```

A Servlet can also be configured to load during application startup rather than waiting for the first request.

| Configuration behavior | Result |
|---|---|
| Default lazy initialization | Servlet is typically initialized on its first matching request |
| Startup loading | Servlet is initialized when the application starts |

The key point is that the container controls creation and initialization timing.

---

## 10. Query Parameters vs. JSON Request Bodies

A query-parameter-based request is useful for learning how requests reach a Servlet:

```http
GET /users?name=Rahul&email=rahul@test.com
```

For a beginner's first Servlet example, this approach keeps attention on URL mapping, request handling, and the request–response cycle.

Handling a JSON request body involves additional steps, such as:

1. Reading the HTTP request body.
2. Parsing JSON.
3. Using a JSON library such as Jackson or Gson, if required.
4. Mapping the parsed data to a Java object.
5. Handling invalid JSON.

The original notes use query parameters to introduce the Servlet request flow before discussing JSON processing separately.

---

## 11. External Tomcat

In the traditional deployment model, Tomcat is installed separately from the Java web application.

The process is:

```text
Install Tomcat
     |
     v
Build the Java Web Application
     |
     v
Package as a WAR File
     |
     v
Deploy WAR to Tomcat
     |
     v
Start Tomcat
     |
     v
Application Runs
```

This is called **external Tomcat** because the server exists independently of the application package.

The application is deployed inside the server.

### Important Tomcat Directories

A typical Tomcat installation contains:

| Directory | General purpose |
|---|---|
| `bin` | Startup and shutdown scripts |
| `conf` | Server configuration |
| `lib` | Libraries supplied by Tomcat |
| `logs` | Server logs |
| `webapps` | Deployable web applications |

### Configuring the Port

Open:

```text
conf/server.xml
```

Find the HTTP connector configuration, for example:

```xml
<Connector port="8080"
           protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

The connector's `port` specifies the port on which the HTTP connector listens. The example uses port `8080`.

---

## 12. What Is a WAR File?

**WAR stands for Web Application Archive.**

A WAR file packages a Java web application for deployment to a compatible Servlet container.

Example:

```text
servlet-crud.war
```

A WAR file can contain:

- Compiled Java classes.
- Servlet classes.
- HTML, CSS, and JavaScript files.
- `web.xml`.
- Application libraries.
- Configuration files.

A simplified example structure is:

```text
servlet-crud.war
├── WEB-INF/
│   ├── classes/
│   │   └── Servlet classes
│   ├── lib/
│   │   └── Application libraries
│   └── web.xml
└── HTML, CSS, JavaScript and other resources
```

The exact contents depend on how the application is packaged.

### Deploying a WAR

Copy the WAR file into Tomcat's `webapps` directory.

Example on macOS/Linux:

```bash
cp target/servlet-crud.war ~/tools/apache-tomcat-10/webapps/
```

Example destination on Windows:

```text
C:\tools\apache-tomcat-10\webapps
```

After Tomcat starts, it can automatically extract the WAR into an application directory:

```text
webapps/
└── servlet-crud/
```

If the application is deployed under the `servlet-crud` context path and is configured correctly, its URL may be:

```text
http://localhost:8080/servlet-crud/
```

The context path is part of the URL used to access the deployed web application.

### Multiple Applications in One Tomcat

One external Tomcat instance can host multiple web applications:

```text
Tomcat
├── app1
│   └── HelloServlet
├── app2
│   └── LoginServlet
└── app3
    └── ProductServlet
```

For example:

```text
/app1/hello  → Application app1
/app2/login  → Application app2
```

Tomcat first identifies the deployed application and then resolves the matching Servlet within that application.

---

## 13. Why Is the Servlet API Dependency `provided`?

Servlet applications use API classes such as:

- `HttpServlet`
- `HttpServletRequest`
- `HttpServletResponse`

These classes are needed to compile the application. However, a compatible Servlet container such as Tomcat provides the Servlet API at runtime.

In Maven, this is the purpose of the `provided` dependency scope.

```xml
<scope>provided</scope>
```

### Compile Time vs. Runtime

```text
Compile time:
Maven provides the Servlet API to compile the application.

Runtime:
Tomcat provides the Servlet API when running the application.
```

With `provided` scope, the dependency is available for compilation but is not normally packaged as an application runtime library inside the WAR.

### What If `provided` Is Removed?

Without `provided`, Maven uses the default `compile` scope for a dependency unless another scope is specified.

The Servlet API may then be included in the application's WAR even though Tomcat already supplies the API.

This can introduce duplicate API classes and potentially cause classloading conflicts or confusing runtime issues.

**Rule:** When deploying to an external Servlet container, mark the container-provided Servlet API dependency as `provided`.

The Servlet API version must be compatible with the target container.

---

## 14. Running Tomcat on macOS/Linux

The following commands assume a Tomcat installation in the indicated directory.

### Enter the Tomcat Directory

```bash
cd ~/Downloads/apache-tomcat-10.1.x
```

### Make Shell Scripts Executable

```bash
chmod +x bin/*.sh
```

This step may be necessary on macOS/Linux. It is not required on Windows.

### Start in the Foreground

```bash
./bin/catalina.sh run
```

Foreground mode makes server logs visible in the terminal.

Open:

```text
http://localhost:8080
```

If Tomcat started successfully, its welcome page should be accessible.

To stop a foreground process, press `Ctrl + C`.

### Start in the Background

```bash
./bin/startup.sh
```

To stop Tomcat:

```bash
./bin/shutdown.sh
```

---

## 15. Running Tomcat on Windows

Open Command Prompt and enter the Tomcat directory:

```bat
cd C:\apache-tomcat-10.1.x
```

### Start in the Foreground

```bat
bin\catalina.bat run
```

Open:

```text
http://localhost:8080
```

Stop the foreground process with `Ctrl + C`.

### Start in the Background

```bat
bin\startup.bat
```

To stop Tomcat:

```bat
bin\shutdown.bat
```

Run these commands from the Tomcat installation directory unless you provide the full script path.

---

## 16. Troubleshooting: Port 8080 Is Already in Use

If Tomcat cannot start because port `8080` is occupied, identify the process using that port.

### macOS/Linux

Find the process:

```bash
lsof -i :8080
```

The output may identify a Java process or another application.

To terminate a process, use its actual process ID:

```bash
kill -9 PROCESS_ID
```

Use forceful termination only when appropriate; it does not allow the process to shut down gracefully.

### Windows

Find the process:

```bat
netstat -ano | findstr :8080
```

The output includes a PID. To terminate the identified process:

```bat
taskkill /PID PROCESS_ID /F
```

Replace `PROCESS_ID` with the actual PID, after verifying that the process can safely be stopped.

An alternative is to configure Tomcat to use another available port in `conf/server.xml`.

---

## 17. Embedded Tomcat in Spring Boot

Traditional Servlet applications commonly deploy a WAR into an independently installed Tomcat server.

Spring Boot commonly uses **embedded Tomcat**, which allows the server to start as part of the application.

For a typical Spring Boot web application, `spring-boot-starter-web` brings in the necessary web dependencies, including embedded Tomcat by default.

A typical entry point is:

```java
public static void main(String[] args) {
    SpringApplication.run(MyApp.class, args);
}
```

The startup flow is:

```text
Run main()
    |
    v
Spring Boot Starts
    |
    v
Embedded Tomcat Starts
    |
    v
Application Becomes Available
```

### External Tomcat vs. Embedded Tomcat

| External Tomcat | Embedded Tomcat |
|---|---|
| Tomcat is installed separately | Tomcat runs within the application process |
| A WAR is commonly deployed to the server | The application commonly starts through `main()` |
| Server is started independently | Spring Boot starts the embedded server |
| Deployment requires server setup and application deployment | Running the application can start both application and server |

Spring Boot does not eliminate the Servlet container concept. It simplifies how the application is packaged, started, and run.

Tomcat still receives HTTP requests and provides the Servlet execution environment.

---

## 18. Traditional Servlets vs. Spring MVC

### Traditional Servlet Style

A traditional Servlet application can map different URLs to separate Servlet classes:

```text
/users     → UserServlet
/products  → ProductServlet
/orders    → OrderServlet
```

A Servlet may be responsible for:

- Reading request parameters.
- Performing validation.
- Calling business logic.
- Preparing responses.
- Handling errors.
- Managing the request flow.

As an application grows, maintaining these responsibilities across many Servlets can become difficult.

### Spring MVC's Approach

Spring MVC introduces a central Servlet called **`DispatcherServlet`**.

Instead of Tomcat directly invoking a different custom Servlet for every application URL, requests first reach `DispatcherServlet`, which identifies the appropriate controller method.

The flow becomes:

```text
Browser / Postman
       |
       v
     Tomcat
       |
       v
DispatcherServlet
       |
       v
   Controller
       |
       v
     Service
       |
       v
   Response
```

Example controller:

```java
@RestController
public class UserController {

    @GetMapping("/users")
    public String getUsers() {
        return "All users";
    }
}
```

When the client sends:

```http
GET /users
```

the sequence is:

1. Tomcat receives the HTTP request.
2. Tomcat invokes Spring MVC's `DispatcherServlet`.
3. `DispatcherServlet` checks the registered request mappings.
4. Spring MVC invokes the appropriate controller method.
5. The response is returned to the client.

The browser does not know about `UserController`, `getUsers()`, or `@GetMapping`. It sends an HTTP request to `/users`.

### What Spring MVC Simplifies

Spring MVC provides abstractions for:

- `@Controller` and `@RestController`
- `@GetMapping` and `@PostMapping`
- `@RequestParam`
- `@RequestBody`
- `@ResponseBody`
- Automatic JSON conversion when appropriately configured
- Controller–service–repository separation

**Key relationship:** Spring MVC is built on the Servlet foundation. It does not replace Servlets; it uses `DispatcherServlet` to provide a more structured request-processing framework.

---

## 19. Servlet vs. Spring MVC Controller

| Aspect | Traditional Servlet | Spring MVC Controller |
|---|---|---|
| Main component | Servlet class | Controller class |
| Request entry point | A Servlet mapped to a URL | Usually Spring MVC's `DispatcherServlet` |
| Request routing | Servlet URL mappings | Spring MVC request mappings |
| Typical handler | `doGet()`, `doPost()`, etc. | Methods annotated with `@GetMapping`, `@PostMapping`, etc. |
| Request handling | More manual handling in Servlet code | Framework-assisted binding and handling |
| JSON conversion | Requires appropriate parsing and response handling | Can use configured message converters automatically |
| Underlying technology | Servlet API | Servlet API through Spring MVC |

A controller method is not directly called by the browser. Tomcat invokes `DispatcherServlet`, which coordinates the request and invokes the matching controller method.

---

## 20. Complete Mental Model

Remember the responsibilities of each component:

| Component | Responsibility |
|---|---|
| Browser or client | Sends HTTP requests and receives HTTP responses |
| HTTP | Defines the communication protocol |
| Tomcat | Listens for HTTP requests and runs web applications |
| Servlet container | Manages Servlet objects and their lifecycle |
| `HttpServletRequest` | Represents incoming request information |
| `HttpServletResponse` | Supports preparing the outgoing response |
| Servlet | Processes a request and generates application-level output |
| `DispatcherServlet` | Central Servlet used by Spring MVC to coordinate request routing |
| Controller | Handles the matched application endpoint |
| Service | Typically contains application or business logic |

### Final Request Flow

```text
Browser sends HTTP request
         |
         v
Tomcat receives the request
         |
         v
Tomcat identifies the web application
         |
         v
Tomcat resolves the matching Servlet
         |
         v
Tomcat provides request/response objects
         |
         v
Servlet processes the request
         |
         v
Servlet prepares the response
         |
         v
Tomcat sends the HTTP response
         |
         v
Browser receives the response
```

For a traditional Servlet application, the request usually reaches the mapped custom Servlet directly.

For a Spring MVC application, the mapped entry point is usually `DispatcherServlet`, which then routes the request to the appropriate controller method.

**The essential principle:**

- The browser communicates using HTTP.
- Java implements application logic.
- Tomcat connects incoming HTTP requests to Java code.
- The Servlet container manages Servlet objects and their lifecycle.
- Spring MVC builds a higher-level request-handling framework on top of the Servlet API.
