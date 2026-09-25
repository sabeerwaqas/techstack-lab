# Java Strings — Core Concepts

## 1. What is a String?

A `String` is a **sequence of characters**.

```java
String name = "Sabeer";
```

Conceptually:

```text
'S'  'a'  'b'  'e'  'e'  'r'
 ↓    ↓    ↓    ↓    ↓    ↓
Character sequence
```

In Java:

* `char` is a **primitive** type representing a UTF-16 code unit.
* `String` is a **class** (`java.lang.String`).
* `String` is a **reference type** and is automatically available because `java.lang` is implicitly imported.

---

## 2. Why use String instead of `char[]`?

A `char[]` can store characters, but `String` provides many built-in operations and guarantees immutability.

For example:

```java
String s = "Hello World";

s.length();
s.substring(0, 5);
s.equals("Hello World");
s.concat("!");
```

With a raw `char[]`, many such operations would have to be implemented manually.

---

## 3. String is Immutable

A Java `String` **cannot be modified after it is created**.

```java
String s = "Hello";

s.concat(" World");

System.out.println(s);
```

Output:

```text
Hello
```

`concat()` does not modify the original object. It creates/returns another `String`.

Correct usage:

```java
s = s.concat(" World");
```

Conceptually:

```text
Before:

s ───────► "Hello"


After s = s.concat(" World"):

s ───────► "Hello World"

"Hello" remains unchanged
```

### Why immutable?

Immutability is useful because Strings are commonly used for:

* Hash keys
* URLs
* Configuration values
* File paths
* Identifiers
* Security-sensitive values

A `String` used as a key in `HashMap`/`HashSet` must not change while the collection relies on its hash/equality behavior.

It also enables safe sharing of String objects through the **String pool**.

---

# 4. Two Ways to Create a String

## String Literal

```java
String s1 = "Hello";
```

## `new` Operator

```java
String s2 = new String("Hello");
```

These are **not equivalent in object identity**.

---

# 5. String Pool

Java maintains a special pool called the **String Pool** (intern pool).

In modern Java, the String pool is located in the **heap**.

Its main purpose is to **reuse identical interned String objects**.

### String literal

```java
String s1 = "Hello";
String s2 = "Hello";
String s3 = "Hello";
```

Conceptually:

```text
                Heap
        ┌───────────────────┐
        │   String Pool     │
        │                   │
        │   "Hello"         │
        │      ▲ ▲ ▲        │
        └──────┼─┼─┼────────┘
               │ │ │
              s1 s2 s3
```

Only one pooled `"Hello"` object is reused.

---

# 6. `new String()` Does Not Reuse the Pool Object

```java
String s1 = new String("Hello");
String s2 = new String("Hello");
```

Conceptually:

```text
Heap
┌─────────────────────────────┐
│ String Pool                 │
│   "Hello"                   │
└─────────────────────────────┘

┌─────────────────────────────┐
│ Normal Heap                 │
│   "Hello"  ◄── s1           │
│   "Hello"  ◄── s2           │
└─────────────────────────────┘
```

`s1` and `s2` reference different objects.

The `"Hello"` argument itself is a String literal, so the pooled `"Hello"` may also exist.

---

# 7. `==` vs `equals()`

This is one of the most important String interview concepts.

## `==`

Checks **reference identity**.

```java
String s1 = "Hello";
String s2 = "Hello";

System.out.println(s1 == s2);
```

Output:

```text
true
```

Both point to the same pooled object.

---

## `equals()`

Checks **String content**.

```java
String s1 = new String("Hello");
String s2 = new String("Hello");

System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
```

Output:

```text
false
true
```

Because:

```text
s1 ─────► "Hello"
s2 ─────► "Hello"

Different references
Same content
```

### Rule

```text
==       → same object/reference?
equals() → same content?
```

For comparing String values, normally use:

```java
s1.equals(s2)
```

---

# 8. Compile-Time Constant Concatenation

Consider:

```java
String s1 = "Java" + "World";
String s2 = "JavaWorld";

System.out.println(s1 == s2);
```

Output:

```text
true
```

Because both operands are compile-time constants.

The compiler can effectively resolve:

```java
"Java" + "World"
```

to:

```java
"JavaWorld"
```

So the pooled object can be reused.

Conceptually:

```text
Compile time:

"Java" + "World"
       ↓
"JavaWorld"
       ↓
String Pool
```

---

# 9. Runtime Concatenation

Now consider:

```java
String s1 = "Java";
String s2 = s1 + "World";

String s3 = "JavaWorld";

System.out.println(s2 == s3);
```

` s1 + "World"` depends on a variable, so the concatenation is performed at runtime.

Therefore `s2` and `s3` need not reference the same object.

```text
String Pool                 Heap
┌───────────────┐           ┌─────────────────┐
│ "Java"        │           │ "JavaWorld"     │ ◄── s2
│ "World"       │           └─────────────────┘
│ "JavaWorld"   │ ◄── s3
└───────────────┘
```

So:

```java
s2 == s3
```

is typically:

```text
false
```

### Important correction

Do not memorize:

> "Every runtime-created String always goes to normal heap and every compile-time String always goes to the pool."

That is an oversimplification.

A runtime-created String can be explicitly interned:

```java
String s = new String("Hello").intern();
```

After `intern()`, the reference points to the canonical pooled String.

---

# 10. Assignment Does Not Create a New String

```java
String s1 = "Java";
String s2 = s1;
```

No new String object is required.

Both references point to the same object:

```text
String Pool

"Java"
  ▲  ▲
  │  │
 s1  s2
```

Therefore:

```java
s1 == s2
```

is:

```text
true
```

---

# 11. Reassigning a String

```java
String s = "Hello";

s = "World";
```

The original `"Hello"` String is **not modified**.

Instead, the reference changes:

```text
Before:

s ─────► "Hello"


After:

s ─────► "World"

"Hello" remains unchanged.
```

If no live reference remains to `"Hello"`, it may eventually become eligible for garbage collection.

---

# 12. `new String("Hello")` — Important Case

```java
String s = new String("Hello");
```

There are two relevant objects/concepts:

```text
String Pool
┌──────────────┐
│ "Hello"      │
└──────────────┘
       ▲
       │ literal used by constructor

Normal Heap
┌──────────────┐
│ new String   │ ◄── s
│ "Hello"      │
└──────────────┘
```

So:

```java
String s1 = new String("Hello");
String s2 = "Hello";

System.out.println(s1 == s2);
```

Output:

```text
false
```

But:

```java
s1.equals(s2)
```

is:

```text
true
```

---

# 13. Immutability and the "New Object" Problem

Consider:

```java
String s = "";

for (int i = 0; i < 5; i++) {
    s += i;
    System.out.println(s);
}
```

Output:

```text
0
01
012
0123
01234
```

Because Strings are immutable, each concatenation creates a new String value rather than modifying the existing String.

Conceptually:

```text
"" 
 ↓
"0"
 ↓
"01"
 ↓
"012"
 ↓
"0123"
 ↓
"01234"
```

The intermediate Strings may become unreachable:

```text
""       ← eventually unused
"0"      ← eventually unused
"01"     ← eventually unused
"012"    ← eventually unused
"0123"   ← eventually unused
"01234"  ← final reference
```

This can produce unnecessary object creation, especially inside large loops.

---

# 14. StringBuilder / StringBuffer

For repeated String modifications, mutable classes are available:

```java
StringBuilder
StringBuffer
```

Unlike `String`, they are mutable.

Conceptually:

```text
String
Immutable
"Hello"
   ↓ concat
"Hello World"   → new object


StringBuilder
Mutable
"Hello"
   ↓ append()
"Hello World"   → same builder object modified
```

`StringBuilder` and `StringBuffer` are designed to reduce unnecessary intermediate String objects during repeated modifications.

---

# 15. Internal Structure of `String`

Conceptually, modern Java String implementations use something similar to:

```java
public final class String {
    private final byte[] value;
    private final byte coder;
    private int hash;
}
```

The exact implementation details can vary between JDK versions, but these fields illustrate the important concepts.

### Why `final`?

`String` is immutable.

### `value`

Stores the actual encoded String data.

### `coder`

Indicates how the bytes should be interpreted.

### `hash`

Can cache the String's hash code.

---

# 16. Java 8 vs Java 9+: Compact Strings

Older Java implementations commonly represented String contents using:

```text
char[]
```

Since Java 9, modern implementations use **compact strings** with:

```text
byte[]
```

plus a `coder`.

The purpose is to reduce memory usage when characters can be represented in one byte.

---

# 17. Compact Strings: `LATIN1` vs `UTF16`

The important distinction is:

```text
coder = LATIN1
    → 1 byte per character/code unit

coder = UTF16
    → 2 bytes per UTF-16 code unit
```

Conceptually:

```text
String
  │
  ├── LATIN1
  │     └── byte[]
  │
  └── UTF16
        └── byte[]
```

### Important correction

This is not simply:

```text
ASCII → 1 byte
Unicode → 2 bytes
```

More accurately, Java's compact-string implementation uses:

* **LATIN1** when all characters fit in Latin-1 (`U+0000`–`U+00FF`)
* **UTF-16** otherwise

UTF-16 can require multiple code units for some Unicode characters, such as many emoji.

---

# 18. Why `byte[]` Saves Memory

Suppose a String contains only Latin-1 characters:

```java
String s = "Java";
```

Conceptually:

```text
Old representation:

char[]:
J → 2 bytes
a → 2 bytes
v → 2 bytes
a → 2 bytes

Total = 8 bytes
```

Compact representation:

```text
byte[]:
J → 1 byte
a → 1 byte
v → 1 byte
a → 1 byte

Total = 4 bytes
```

For such strings, this can roughly halve the storage required for the character data.

---

# 19. What Does `coder` Do?

`coder` tells the String implementation how the `byte[]` should be interpreted.

Conceptually:

```text
coder
 ├── LATIN1
 │      → interpret bytes as Latin-1
 │
 └── UTF16
        → interpret bytes as UTF-16
```

Example:

```text
"Java"
   ↓
LATIN1
   ↓
1 byte per character


"کگ"
  ↓
UTF16
  ↓
2 bytes per UTF-16 code unit
```

The exact internal constant names and implementation details are JDK-specific; the key concept is **compact encoding + coder flag**.

---

# 20. Cached Hash Code

Strings are frequently used as keys in hash-based collections:

```java
HashMap<String, Integer>
HashSet<String>
```

A String's hash code can be computed and cached.

Conceptually:

```text
String
 ├── value
 ├── coder
 └── hash
       ↑
       └── cached hash value
```

Example:

```java
String s = "Java";

int h1 = s.hashCode();
int h2 = s.hashCode();
```

Once computed, the implementation can reuse the cached result rather than recalculating it each time.

### Why is this safe?

Because:

```text
String is immutable
        ↓
contents never change
        ↓
hash value never changes
        ↓
cached hash is safe to reuse
```

---

# 21. Three Major String Optimizations

Modern Java gets important benefits from three mechanisms:

### 1. String Pool

Reuses identical interned Strings.

```text
"Hello"
   ▲ ▲ ▲
   │ │ │
  s1 s2 s3
```

### 2. Compact Strings

Uses `byte[]` instead of always using 2-byte `char` storage.

```text
LATIN1 → 1 byte
UTF16  → 2 bytes per UTF-16 code unit
```

### 3. Cached Hash Code

Avoids repeatedly calculating the same String hash.

```text
String
   ↓
hashCode()
   ↓
cached hash
```

---

# 22. Interview Rules to Remember

```text
String
│
├── Immutable
├── final class
├── java.lang.String
├── String literals use the String Pool
├── new String(...) creates a distinct String object
├── == checks reference identity
├── equals() checks content
├── compile-time constant concatenation can be pooled
├── runtime concatenation creates a new String result
├── modern Java uses compact strings
├── byte[] + coder is used internally
└── hash code can be cached
```

## Most Important Mental Model

```text
                STRING
                   │
        ┌──────────┴──────────┐
        │                     │
    Immutable             String Pool
        │                     │
        │                Reuse literals
        │
        ├───────────────┐
        │               │
    byte[]           coder
        │               │
   String data     LATIN1 / UTF16
        │
      hash
        │
   cached value
```
