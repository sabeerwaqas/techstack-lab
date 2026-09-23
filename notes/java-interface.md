# Java Interfaces

## 1. What is an Interface?

An interface defines a **contract** for a class.

It tells:

> **What a class can do, not how it does it.**

```java
interface Car {
    void drive();
}
```

A class implements the interface:

```java
class Thar implements Car {

    @Override
    public void drive() {
        System.out.println("Thar is driving");
    }
}
```

### Remember

* Interface = **contract / behavior**
* Cannot create an object directly from an interface
* A class uses `implements`
* An interface uses `extends`

---

## 2. Interface Methods

Traditional interface methods are implicitly:

```java
public abstract
```

So:

```java
interface Car {
    void drive();
}
```

is effectively:

```java
interface Car {
    public abstract void drive();
}
```

When implementing the method, visibility cannot be reduced:

```java
public void drive() { } // ✅
void drive() { }        // ❌
```

---

## 3. If a Class Doesn't Implement All Methods

A concrete class must implement all abstract interface methods.

Otherwise, the class must be `abstract`.

```java
abstract class Thar implements Car {
}
```

Then another concrete class can implement the method.

---

## 4. Interface Polymorphism

```java
interface Payment {
    void pay();
}

class CreditCard implements Payment {
    public void pay() {
        System.out.println("Credit Card");
    }
}

class DebitCard implements Payment {
    public void pay() {
        System.out.println("Debit Card");
    }
}
```

```java
Payment p = new CreditCard();
p.pay();
```

Output:

```text
Credit Card
```

```java
p = new DebitCard();
p.pay();
```

Output:

```text
Debit Card
```

### Key Rule

* **Reference type** → determines what methods can be accessed
* **Actual object type** → determines which overridden implementation runs

This is **runtime polymorphism / dynamic dispatch**.

---

## 5. Variables in Interfaces

Interface variables are automatically:

```java
public static final
```

Example:

```java
interface MathConstants {
    double PI = 3.14;
    int VALUE = 10;
}
```

Equivalent to:

```java
public static final double PI = 3.14;
```

So they are **constants**.

Access directly:

```java
MathConstants.PI;
```

---

## 6. Multiple Inheritance

Java does **not** support multiple inheritance through classes:

```java
class C extends A, B { } // ❌
```

But a class can implement multiple interfaces:

```java
class C implements A, B {
}
```

A class can:

* Extend **one class**
* Implement **multiple interfaces**

---

## 7. Interface Inheritance

An interface can extend another interface:

```java
interface Animal {
    void eat();
}

interface Dog extends Animal {
    void bark();
}
```

A class implementing `Dog` must implement both:

```java
class StreetDog implements Dog {
    
    public void eat() {
    }

    public void bark() {
    }
}
```

---

# Java 8 Changes

## 8. Default Methods

Before Java 8, interface methods were generally abstract.

Java 8 introduced `default` methods:

```java
interface Vehicle {
    default void drive() {
        System.out.println("Vehicle is driving");
    }
}
```

A class can use the default implementation without overriding it.

It can also override it:

```java
class Car implements Vehicle {

    @Override
    public void drive() {
        System.out.println("Car is driving");
    }
}
```

### Why were default methods introduced?

Main reason:

> **Backward compatibility when evolving existing interfaces.**

Example: If Java added a new abstract method to `List`, existing implementations could break.

A `default` method gives existing implementations a ready-made implementation.

---

## 9. Static Methods in Interfaces

Java 8 also introduced static interface methods:

```java
interface Vehicle {

    static void brake() {
        System.out.println("Applying brake");
    }
}
```

Call it using the interface name:

```java
Vehicle.brake();
```

Not through an object/reference.

---

# Java 9 Changes

## 10. Private Methods in Interfaces

Java 9 introduced private interface methods.

```java
interface Vehicle {

    default void drive() {
        accelerate();
    }

    private void accelerate() {
        System.out.println("Accelerating");
    }
}
```

Purpose:

> Used internally by other methods inside the interface.

They cannot be called from outside the interface.

---

# Diamond Problem

## 11. What is the Diamond Problem?

Imagine:

```text
      A
     / \
    B   C
     \ /
      D
```

If both `B` and `C` provide different implementations of the same method, `D` would not know which one to inherit.

This ambiguity is called the **Diamond Problem**.

Java avoids this by not allowing multiple inheritance of classes.

---

## 12. Default Method Conflict

With Java 8, interfaces can have implementations.

So this can create a conflict:

```java
interface B {
    default void fun() {
        System.out.println("B");
    }
}

interface C {
    default void fun() {
        System.out.println("C");
    }
}
```

```java
class D implements B, C {
}
```

❌ Conflict: Java doesn't know which `fun()` to use.

`D` must resolve it:

```java
class D implements B, C {

    @Override
    public void fun() {
        System.out.println("D");
    }
}
```

Or explicitly choose one:

```java
@Override
public void fun() {
    B.super.fun();
}
```

or:

```java
C.super.fun();
```

---

# 13. Class Method vs Interface Default Method

If a class already has a method and an interface provides a default method with the same signature:

```java
class B {
    public void fun() {
        System.out.println("B");
    }
}

interface A {
    default void fun() {
        System.out.println("A");
    }
}

class C extends B implements A {
}
```

The **class method wins**.

```java
new C().fun();
```

Output:

```text
B
```

### Rule

> **Class implementation has priority over interface default implementation.**

---

# Interface vs Abstract Class

## 14. Main Conceptual Difference

### Interface

Represents:

> **What can this class do?**

Examples:

```text
Runnable
Comparable
Payable
```

Think:

> **Capability / Contract**

---

### Abstract Class

Represents:

> **What family does this class belong to?**

Example:

```java
abstract class Animal {
    void eat() {
    }
}

class Dog extends Animal {
}

class Cat extends Animal {
}
```

Think:

> **Shared base / family**

---

## 15. Interface vs Abstract Class

| Feature                     | Interface                                    | Abstract Class                                    |
| --------------------------- | -------------------------------------------- | ------------------------------------------------- |
| Multiple inheritance        | ✅ Multiple interfaces                        | ❌ Only one superclass                             |
| Instance variables          | ❌                                            | ✅                                                 |
| Constructors                | ❌                                            | ✅                                                 |
| Abstract methods            | ✅                                            | ✅                                                 |
| Concrete methods            | ✅ `default`                                  | ✅                                                 |
| Static methods              | ✅                                            | ✅                                                 |
| Private methods             | ✅ Java 9+                                    | ✅                                                 |
| Access modifiers on methods | Mostly `public`; private methods are special | `public`, `protected`, package-private, `private` |
| Main purpose                | Contract / capability                        | Shared base / family                              |

---

# Functional Interface

## 16. What is a Functional Interface?

A functional interface has:

> **Exactly one abstract method**

Example:

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}
```

It may still have:

* `default` methods
* `static` methods
* `private` methods

Only **one abstract method** is allowed.

Functional interfaces are used heavily with **lambda expressions**.

Common examples:

```text
Runnable
Comparable
Predicate
Function
Consumer
Supplier
```

---

# Marker Interface

## 17. What is a Marker Interface?

A marker interface has:

> **No methods**

Example:

```java
interface MyMarker {
}
```

It is used to **mark or indicate a capability/property**.

Common Java examples:

```text
Cloneable
Serializable
RandomAccess
```

### Example

```java
class Student implements Cloneable {
}
```

`Cloneable` itself does not contain `clone()`.

It is a marker used by Java's cloning mechanism.

---

# Important Syntax Rules

## 18. Quick Revision

```text
Class → implements → Interface

Interface → extends → Interface

Class → extends → One Class

Class → implements → Multiple Interfaces
```

### Interface fields

```text
public static final
```

### Traditional interface methods

```text
public abstract
```

### Java 8

```text
default methods
static methods
```

### Java 9

```text
private methods
```

---

# Final Mental Model

```text
INTERFACE
│
├── Contract / Capability
├── Abstract methods
├── default methods
├── static methods
├── private methods
├── public static final fields
├── Multiple interfaces can be implemented
├── Functional Interface → 1 abstract method
└── Marker Interface → 0 methods
```

```text
ABSTRACT CLASS
│
├── Shared family/base class
├── Instance state
├── Constructors
├── Concrete methods
├── Abstract methods
└── Only one superclass
```

## Most Important Takeaways

1. **Interface = contract / capability**
2. **Abstract class = shared base/family**
3. **A class can implement multiple interfaces**
4. **Interface fields are `public static final`**
5. **Java 8 → default + static methods**
6. **Java 9 → private interface methods**
7. **Functional interface → exactly one abstract method**
8. **Marker interface → no methods**
9. **Class method takes priority over interface default method**
10. **Conflicting default methods must be resolved by the implementing class**
