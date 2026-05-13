# Maven Beginner Notes

# What is Maven?

Maven is a build automation and dependency management tool for Java projects.

It helps developers:

- compile code
- run tests
- manage dependencies
- package applications
- build projects consistently

Instead of doing everything manually, Maven automates the process.

---

# Why Maven Exists

Without Maven, developers would manually need to:

- download libraries
- manage library versions
- compile Java files
- configure classpaths
- run tests
- package applications

This becomes difficult in large projects.

Maven solves this problem.

---

# Main Responsibilities of Maven

| Responsibility | Description |
|---|---|
| Build Automation | Compile, test, and build applications |
| Dependency Management | Download and manage external libraries |
| Project Standardization | Maintain a common project structure |

---

# What is a Build?

A build means:

> Converting source code into a runnable application.

Example flow:

```text
Source Code → Compilation → Testing → Build Output
```

---

# What is Dependency Management?

Dependencies are external libraries required by your project.

Instead of manually downloading libraries, Maven automatically:

- downloads them
- manages versions
- handles dependency relationships
- stores them locally

Example dependency in `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

---

# What is `pom.xml`?

`pom.xml` is the heart of a Maven project.

POM means:

```text
Project Object Model
```

This file contains:

- project information
- dependencies
- plugins
- Java version
- build configuration

Example:

```xml
<project>
    <dependencies>
    </dependencies>
</project>
```

---

# Standard Maven Project Structure

```text
project/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   │
│   └── test/
│       └── java/
│
├── target/
├── pom.xml
```

---

# Important Maven Folders

| Folder | Purpose |
|---|---|
| `src/main/java` | Application source code |
| `src/test/java` | Test code |
| `src/main/resources` | Configuration files |
| `target/` | Generated build files |

---

# What is the `target/` Folder?

The `target/` folder contains generated build files.

Examples:

- compiled files
- temporary build files
- packaged application output

This folder is generated automatically by Maven.

---

# What is Maven Lifecycle?

A lifecycle is:

> A predefined sequence of build phases.

Maven performs tasks step-by-step in a fixed order.

---

# Main Maven Lifecycles

| Lifecycle | Purpose |
|---|---|
| clean | Remove old build files |
| default | Build the application |
| site | Generate project documentation |

---

# Important Maven Phases

| Phase | Purpose |
|---|---|
| validate | Check project structure |
| compile | Compile source code |
| test | Run tests |
| package | Package the application |
| verify | Run additional checks |
| install | Store build output locally |
| deploy | Upload build output remotely |

---

# Most Important Maven Concept

When you run a phase, Maven automatically runs all previous phases.

Example:

```bash
mvn install
```

Maven runs:

```text
validate → compile → test → package → verify → install
```

---

# What Does `mvn clean install` Do?

```bash
mvn clean install
```

This command runs two lifecycles:

1. `clean`
2. `default` lifecycle up to `install`

---

# Step 1: clean

```bash
mvn clean
```

Deletes old generated build files.

Mainly removes:

```text
target/
```

This ensures a fresh build.

---

# Step 2: install

```bash
mvn install
```

Runs these phases:

```text
validate
↓
compile
↓
test
↓
package
↓
verify
↓
install
```

---

# Visual Flow of `mvn clean install`

```text
mvn clean install

clean
 ↓
validate
 ↓
compile
 ↓
test
 ↓
package
 ↓
verify
 ↓
install
```

---

# What Happens Internally?

When running:

```bash
mvn clean install
```

Maven may:

- delete old build files
- download dependencies
- compile code
- run tests
- package the application
- store build output locally

---

# What are Plugins?

Plugins are tools Maven uses to perform actual work.

Examples:

| Plugin | Purpose |
|---|---|
| Compiler Plugin | Compile Java code |
| Surefire Plugin | Run tests |
| Clean Plugin | Remove build files |

---

# Important Understanding

```text
Maven = Manager
Plugins = Workers
```

Maven controls the build process.
Plugins perform actual tasks.

---

# Common Maven Commands

## Clean build files

```bash
mvn clean
```

---

## Compile project

```bash
mvn compile
```

---

## Run tests

```bash
mvn test
```

---

## Package application

```bash
mvn package
```

---

## Build and install locally

```bash
mvn install
```

---

## Full clean build

```bash
mvn clean install
```

---

# Why Teams Use Maven

Maven helps teams because:

- everyone follows same project structure
- builds become reproducible
- dependencies are managed automatically
- project setup becomes easier

A developer can simply run:

```bash
mvn clean install
```

and build the project successfully.

---

# Simple Mental Model

Think of Maven as:

```text
A factory manager for Java projects.
```

It automates:

- dependency management
- build steps
- testing
- packaging
- project structure

---

# Final Summary

## Maven

A tool that automates building and managing Java projects.

---

## Lifecycle

A sequence of build phases.

---

## Phase

A specific build step like compile or test.

---

## Dependency

An external library required by the project.

---

## `mvn clean install`

Deletes old build files, rebuilds the project, runs tests, and stores the build output locally.
