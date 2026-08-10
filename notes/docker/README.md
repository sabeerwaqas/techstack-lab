<img width="1024" height="1024" alt="ChatGPT Image Jul 27, 2025, 05_03_59 PM" src="https://github.com/user-attachments/assets/6022b2d8-a0fd-4846-9113-78ea3207a394" />

# Docker Learning Guide

> My personal learning notes for understanding Docker, containers, images, and containerized application deployment.

---

# 1. The Problem Docker Helps Solve

Before learning Docker, it is important to understand **what problem we are trying to solve**.

## 1.1 Sharing an Application

As software developers, our job is not only to write code but also to build applications that can eventually be used by many users.

Let's say we have developed a web application and now we want to:

* Share it with another developer.
* Give it to the QA/testing team.
* Move it from development to staging.
* Eventually deploy it to production.

At first, this sounds simple:

> "I'll just give them my source code."

But the source code alone is often **not enough**.

Our application usually depends on a particular environment.

For example, while developing a Java application, we may have configured:

* Java version
* Application server
* Database
* Operating-system configuration
* Environment variables
* Required libraries and dependencies
* Network configuration
* Other supporting services

The other developer or testing team may have to configure all of these things before they can even run our application.

---

# 2. The "Works on My Machine" Problem

Imagine that I developed an application using **Java 17**.

My colleague receives the source code and has **Java 21** installed.

They try to run the application and encounter an unexpected problem.

They contact me:

> "The application isn't working."

And I respond:

> "But it works on my machine."

This is a common problem in software development.

The application works in my environment because my machine has a particular combination of:

```text
Application Code
       +
Java Version
       +
Dependencies
       +
Database
       +
Configuration
       +
Operating System
       +
Other Requirements
```

But my colleague's environment may be different:

```text
Application Code
       +
Different Java Version
       +
Different Dependencies
       +
Different Database Configuration
       +
Different OS
       +
Different Environment Variables
```

Even though we are using the **same source code**, the application may behave differently.

---

# 3. Environment Setup Takes Time

Another problem is that setting up an application from scratch can take a significant amount of time.

For example, imagine a new developer joins the team.

To run the project, they may need to:

1. Install Java.
2. Install the correct Java version.
3. Install a database.
4. Configure the database.
5. Configure environment variables.
6. Install other required software.
7. Configure networking.
8. Download project dependencies.
9. Configure the application.
10. Finally, run the application.

The developer may spend hours configuring the environment before writing a single line of code.

This becomes even more problematic when an application consists of multiple services.

For example:

```text
                    Web Application
                          |
              +-----------+-----------+
              |                       |
          Backend API              Database
              |
        +-----+-----+
        |           |
     Redis       Message Queue
```

Now the developer has to configure several different services.

---

# 4. Operating System Differences

There can also be differences between operating systems.

For example:

```text
Developer A
Windows
Java 17
PostgreSQL
```

while another developer has:

```text
Developer B
Linux
Java 17
PostgreSQL
```

Even if both developers install the same software, differences in the operating system and its configuration can sometimes cause unexpected behavior.

So we have a bigger question:

> **What if we could package our application together with the environment it needs to run?**

Instead of telling another developer:

> "Install Java 17, install PostgreSQL, configure this environment variable, change this configuration, and then run the application."

What if we could provide them with something closer to:

> **"Here is the environment in which the application is already configured to run."**

That would make sharing and running applications much easier.

---

# 5. Virtualization

One solution to the environment isolation and compatibility problem is **virtualization**.

Before understanding how virtualization helps us, let's first understand what virtualization actually means.

---

## 5.1 How a Computer System Works

At the most basic level, a computer has **physical hardware**:

```text
CPU
RAM
Motherboard
Storage
Network Card
...
```

But as users and developers, we normally don't interact directly with the hardware.

We use an **Operating System (OS)**.

For example:

```text
Physical Hardware
       ↓
Operating System
       ↓
Applications
```

The operating system acts as an abstraction layer between applications and the physical hardware.

For example:

```text
Calculator Application
        ↓
Operating System
        ↓
Physical Hardware
```

When we use an application, the application communicates with the OS, and the OS manages access to the underlying hardware.

---

# 6. What Is a Virtual Machine?

Now suppose we already have:

```text
Physical Hardware
       ↓
Host Operating System
```

For example:

```text
Physical Hardware
       ↓
Windows
```

What if we want to run another operating system, such as Ubuntu, **at the same time**?

Normally, installing two operating systems using dual boot means we can choose between them, but we generally run only one OS at a time.

Virtualization provides another approach.

We can create a **Virtual Machine (VM)**.

A VM is a software-defined computer that provides virtualized hardware to a guest operating system.

The structure becomes:

```text
Physical Hardware
       ↓
Host Operating System
       ↓
Hypervisor
       ↓
Virtual Machine
       ↓
Guest Operating System
       ↓
Application
```

For example:

```text
Physical Hardware
       ↓
Windows
       ↓
Hypervisor
       ↓
Virtual Machine
       ↓
Ubuntu
       ↓
Java Application
```

The Ubuntu operating system believes it is running on a computer with hardware available to it, but that hardware is **virtualized by the virtualization software/hypervisor**.

---

# 7. Hypervisor

A **hypervisor** is software that creates and manages Virtual Machines.

It provides virtualized hardware resources such as:

* Virtual CPU
* Virtual RAM
* Virtual storage
* Virtual network interfaces

The guest operating system runs on this virtualized hardware.

Conceptually:

```text
Physical Hardware
       ↓
Host OS
       ↓
Hypervisor
       ↓
+-------------------+
| Virtual Machine   |
|                   |
| Guest OS          |
| Application       |
+-------------------+
```

There are different virtualization technologies and hypervisors, including tools such as VMware and VirtualBox.

---

# 8. How Virtualization Solves the "Works on My Machine" Problem

Let's return to our original problem.

Suppose I have built an application on my machine.

My environment looks like:

```text
Ubuntu
Java 17
Web Server
Database
Application
Configuration
```

Instead of giving my colleague only the application source code and asking them to recreate this environment, I can create a **Virtual Machine containing the required environment**.

For example:

```text
Virtual Machine
│
├── Ubuntu
├── Java 17
├── Web Server
├── Database
├── Application
└── Configuration
```

I can then provide this VM to another team.

They can run the VM using a compatible hypervisor.

The important idea is:

> **Instead of asking another machine to recreate my environment, I can package the environment into a virtual machine.**

---

# 9. VM Images

A Virtual Machine can be represented by files that contain the VM's configuration and virtual disk state.

These files can be used to create or reproduce the VM on another machine.

Conceptually:

```text
Running Virtual Machine
        ↓
Save VM state / virtual disk
        ↓
VM Image / Files
        ↓
Transfer to another machine
        ↓
Run using a compatible hypervisor
        ↓
Same Guest OS + Environment
```

This means I don't necessarily have to send the project and instructions such as:

```text
Install Java 17
Install PostgreSQL
Configure PostgreSQL
Install Web Server
Set Environment Variables
...
```

Instead, I can provide the VM containing the configured environment.

The receiving machine still needs suitable virtualization software and sufficient hardware resources, but the environment itself is already packaged.

---

# 10. Application Flow Inside a VM

An important thing to understand is that the virtual machine still ultimately depends on the physical hardware.

For example:

```text
Application
    ↓
Guest OS (Ubuntu)
    ↓
Virtual Hardware
    ↓
Hypervisor
    ↓
Host OS (Windows)
    ↓
Physical Hardware
```

So if an application inside Ubuntu performs some operation, that operation eventually reaches the physical hardware through these layers.

The virtual hardware is an abstraction provided by the virtualization layer.

This is why the guest OS does not need to know the exact physical hardware configuration of the host machine.

---

# 11. Virtualization in Production

Virtualization is not only useful for developers.

It became particularly valuable in server environments.

Imagine a physical server with a large amount of:

* CPU
* RAM
* Storage

If we run only one application on that server, a significant amount of the available resources may remain unused.

For example:

```text
Physical Server
│
├── Huge CPU capacity
├── Huge RAM capacity
└── Storage
        ↓
   One Application
```

This can result in inefficient resource utilization.

What if we could run multiple isolated environments on the same physical server?

Virtualization allows us to do this.

---

# 12. Multiple Virtual Machines on One Server

A simplified architecture looks like this:

```text
Physical Server
│
├── Host OS
│
└── Hypervisor
      │
      ├── Virtual Machine 1
      │     ├── Guest OS
      │     └── Application A
      │
      ├── Virtual Machine 2
      │     ├── Guest OS
      │     └── Application B
      │
      └── Virtual Machine 3
            ├── Guest OS
            └── Application C
```

Each application can run inside its own VM.

This provides a significant degree of **isolation**.

For example:

```text
VM 1
Banking Application
Ubuntu
```

and:

```text
VM 2
Crypto Exchange Application
Ubuntu
```

The applications are separated from each other by the VM boundaries.

This is much safer and more manageable than simply installing unrelated applications directly into the same operating-system environment.

---

# 13. Why Virtualization Became Popular

Virtualization helps solve several important problems:

### 13.1 Better Resource Utilization

Instead of dedicating one physical server to one application, multiple VMs can share the same physical server.

### 13.2 Isolation

Applications can run inside separate virtual machines.

### 13.3 Portability

A VM can potentially be moved or reproduced on another machine that supports the required virtualization technology.

### 13.4 Consistent Environment

The guest OS and its configured software can travel together with the application.

This helps reduce environment-related problems.

---

# 14. Problems With Virtualization

Virtualization solves many problems, but it introduces some of its own.

## 14.1 VMs Are Heavy

A VM typically contains a **complete guest operating system**.

For example:

```text
VM
│
├── Guest OS
├── System Libraries
├── Runtime
├── Dependencies
└── Application
```

If we run multiple VMs, each VM needs resources for its own operating system.

This consumes:

* RAM
* CPU
* Storage
* Startup time

For example:

```text
Physical Machine
│
├── Host OS
│
├── VM 1 → Guest OS + Application
├── VM 2 → Guest OS + Application
└── VM 3 → Guest OS + Application
```

A large portion of the resources may be used simply to run multiple operating systems.

---

## 14.2 Startup Time

Because a VM contains a complete operating system, starting a VM generally takes more time than starting a lightweight isolated process.

---

## 14.3 Resource Overhead

Every guest operating system requires memory, CPU, storage, and other resources.

This becomes expensive when we need to run many applications.

---

# 15. Virtualization vs Our Original Problem

Virtualization can solve our original problem.

Instead of:

```text
Developer
    ↓
Application
    ↓
"Please configure your machine exactly like mine."
```

we can have:

```text
Developer
    ↓
Virtual Machine
    ↓
Guest OS
    ↓
Configured Environment
    ↓
Application
```

The same VM can potentially be provided to:

```text
Developer
    ↓
Testing Team
    ↓
Operations Team
    ↓
Production
```

This greatly reduces the need for everyone to manually recreate the environment.

---

# 16. But We Have a New Problem

Virtualization solved one problem but introduced another.

We wanted to share:

```text
Application
+
Environment
```

But with virtualization, we are effectively sharing:

```text
Application
+
Environment
+
Entire Guest Operating System
```

The guest operating system can be much larger than what the application actually needs.

For a relatively small application, running an entire OS just to provide its environment can be inefficient.

This leads to an important question:

> **Can we get the isolation and portability of virtualization without having to include a complete operating system for every application?**

The answer is:

**Containerization.**

And this is where Docker becomes important.

---

# 17. Key Takeaways

Before moving to containers, remember these concepts:

### Problem

> The application works on my machine, but may not work on another machine because the environments are different.

### Virtualization

> Virtualization allows us to create virtual computers on top of physical hardware.

### Virtual Machine

> A VM is a software-defined computer with virtualized hardware on which a guest operating system can run.

### Hypervisor

> A hypervisor creates and manages virtual machines and provides them with virtualized hardware resources.

### Benefit

> We can package an application's environment inside a VM and reproduce that environment on another machine.

### Limitation

> A VM normally requires a complete guest operating system, which introduces additional resource usage and overhead.

This leads us to the next concept:

# **Containerization**

> **Can we isolate and package an application without carrying an entire guest operating system with it?**
