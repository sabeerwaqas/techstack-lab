# Java Strings — Revision Notes

## 1. String Creation & Memory

### String Literal

```java
String s = "Hello";
```

* Stored in the **String Pool**.
* Special syntax; no explicit `new` is required.
* Same literal can be reused from the pool.

### `new String()`

```java
String s = new String("Hello");
```

* Creates a separate `String` object in normal **heap memory**.
* `"Hello"` as a literal also exists in the String Pool.
* Therefore, this can involve both a pool object and a separate heap object.

### Reference Example

```java
String s1 = "Aditya";
String s2 = new String(s1);
```

```text
String Pool:  "Aditya" ← s1
Heap:         "Aditya" ← s2
```

---

# 2. String Constructors

Common constructor forms:

```java
new String()
new String(String)
new String(char[])
new String(char[], offset, count)
new String(byte[])
new String(byte[], offset, count)
new String(StringBuilder)
new String(StringBuffer)
```

### Important

* `new String()` → empty String (`""`), not `null`.
* `new String("")` → also an empty String.
* `char[]` → converted into a String.
* `byte[]` → converted into a String using byte values.
* `offset + count` → creates a String from part of an array.
* `char[]` data is copied, so later changes to the original array do **not** change the String.
* Array/range operations follow the concept of **inclusive start + exclusive end**.

---

# 3. String Immutability

`String` is **immutable**.

```java
String s = "hello";
s = s.toUpperCase();
```

The original `"hello"` is not modified. A new String is produced.

### Why immutability matters

* A String object cannot be changed after creation.
* Multiple references can safely share the same String.
* Mutable operations on Strings may create additional String objects.

For frequent modifications, use:

```text
StringBuilder
StringBuffer
```

---

# 4. String Methods — Grouped

Don't memorize every overload. Understand the operation categories and use IDE autocomplete/Javadocs when needed.

```text
Length / Empty
Character Access
Comparison
Searching
Extraction
Transformation
Conversion
Advanced / Formatting
```

---

## A. Length / Empty

### `length()`

```java
s.length();
```

Returns number of characters.

> For String → `length()` is a method.
> For arrays → `length` is a field.

```java
String s = "Aditya";
s.length();

int[] arr = {1, 2, 3};
arr.length;
```

### `isEmpty()`

```java
s.isEmpty();
```

Returns `true` only when:

```text
length == 0
```

Example:

```java
"".isEmpty();       // true
"   ".isEmpty();    // false
```

### `isBlank()`

```java
s.isBlank();
```

Available since **Java 11**.

Returns `true` when the String is empty or contains only whitespace.

```java
"".isBlank();       // true
"   ".isBlank();    // true
"Aditya".isBlank(); // false
```

### Key Difference

```text
isEmpty() → checks whether length is 0
isBlank() → checks whether String is empty/whitespace only
```

---

# 5. Character Access

### `charAt()`

```java
s.charAt(index);
```

Returns the character at the specified index.

```java
String s = "Aditya";

s.charAt(2); // 'i'
```

Indexing starts from `0`.

### `toCharArray()`

```java
char[] arr = s.toCharArray();
```

Converts the String into a character array.

---

# 6. String Comparison

## `==`

```java
s1 == s2
```

Compares **references**, not String content.

```java
String s1 = new String("Java");
String s2 = new String("Java");

s1 == s2; // false
```

---

## `equals()`

`String` overrides `equals()` from `Object`.

```java
s1.equals(s2);
```

Compares the **actual String contents**.

```java
String s1 = new String("Java");
String s2 = new String("Java");

s1.equals(s2); // true
```

### Remember

```text
==      → reference comparison
equals  → content comparison for String
```

---

## `equalsIgnoreCase()`

```java
s1.equalsIgnoreCase(s2);
```

Compares String contents while ignoring case.

```java
"java".equalsIgnoreCase("JAVA"); // true
```

---

## `compareTo()`

```java
s1.compareTo(s2);
```

Performs **lexicographical comparison**.

Think of dictionary ordering.

### Return value

```text
negative → s1 comes before s2
0        → both are equal
positive → s1 comes after s2
```

Example:

```java
"ABC".compareTo("ABD"); // negative
"ABC".compareTo("ABC"); // 0
"ABD".compareTo("ABC"); // positive
```

Comparison is based on character/Unicode values.

---

# 7. Searching

## `contains()`

```java
s.contains("ity");
```

Checks whether a character sequence exists.

```java
"Aditya".contains("ity"); // true
```

---

## `indexOf()`

```java
s.indexOf('i');
s.indexOf("ity");
```

Returns the index of the **first occurrence**.

```java
"Aditya".indexOf('i'); // 2
```

If not found:

```text
-1
```

`indexOf()` has multiple overloads, including character, String, and starting-index variants.

---

## `lastIndexOf()`

```java
s.lastIndexOf('i');
s.lastIndexOf("ity");
```

Returns the index of the **last occurrence**.

If not found:

```text
-1
```

---

## `startsWith()`

```java
s.startsWith("Ad");
```

Checks whether the String starts with a specified prefix.

---

## `endsWith()`

```java
s.endsWith("ty");
```

Checks whether the String ends with a specified suffix.

---

# 8. Extraction / Transformation

## `substring()`

Extracts part of a String.

```java
s.substring(beginIndex, endIndex);
```

### Rule

```text
beginIndex → inclusive
endIndex   → exclusive
```

Example:

```java
String s = "Aditya";

s.substring(1, 4);
```

Result:

```text
dit
```

Because indices `1, 2, 3` are included and `4` is excluded.

### Overloads

```java
s.substring(beginIndex);
s.substring(beginIndex, endIndex);
```

---

## `toUpperCase()`

```java
s.toUpperCase();
```

Returns the String in uppercase.

---

## `toLowerCase()`

```java
s.toLowerCase();
```

Returns the String in lowercase.

---

## `trim()`

```java
s.trim();
```

Removes leading and trailing whitespace covered by `trim()`.

```text
"   Java   "
     ↓
"Java"
```

Important:

```text
trim() does NOT remove spaces inside the String.
```

```text
"Java     Spring"
       ↓
"Java     Spring"
```

---

## `strip()`

```java
s.strip();
```

Similar purpose to `trim()` but Unicode-aware.

Use `strip()` when Unicode whitespace handling matters.

---

## `repeat()`

```java
s.repeat(3);
```

Repeats the String the specified number of times.

```java
"Java".repeat(3);
```

Result:

```text
JavaJavaJava
```

---

# 9. Replacement

## `replace(char, char)`

```java
s.replace('i', 'o');
```

Replaces occurrences of one character with another.

---

## `replace(CharSequence, CharSequence)`

```java
s.replace("ity", "ABC");
```

Replaces a matching character sequence.

---

## `replaceAll()`

```java
s.replaceAll("pattern", "replacement");
```

Replaces **all matching occurrences** using a **regular expression**.

Example:

```java
"a1a2a3".replaceAll("a", "X");
```

Result:

```text
X1X2X3
```

> `replaceAll()` is regex-based, so don't confuse it with simple literal replacement.

---

# 10. `split()`

Used to split one String into a `String[]`.

```java
String s = "Aditya,Rohit,Rohan";

String[] arr = s.split(",");
```

Result:

```text
["Aditya", "Rohit", "Rohan"]
```

### Concept

```text
String
   ↓ split(delimiter)
String[]
```

The delimiter can be different from comma:

```java
s.split("-");
s.split("\\.");
```

`split()` is especially useful when processing delimiter-separated input.

---

# 11. `String.join()`

A **static method** used to join multiple values.

```java
String.join("-", "A", "B", "C");
```

Result:

```text
A-B-C
```

### Concept

```text
split() → one String → many Strings
join()  → many Strings → one String
```

Useful for constructing strings such as keys or combined values.

---

# 12. Conversion

## `String.valueOf()`

Static method used to convert values into Strings.

```java
String s = String.valueOf(10);
```

Result:

```text
"10"
```

It has overloads for different data types.

### Similar concept

```text
Integer.valueOf() → converts to Integer
String.valueOf()  → converts value to String
```

---

## `getBytes()`

```java
byte[] bytes = s.getBytes();
```

Converts the String into a byte array.

Useful when working with byte-level representations/encodings.

```text
String → byte[]
```

> The exact byte values depend on the character encoding being used; don't assume every non-ASCII character maps directly to one byte.

---

# 13. `intern()`

`intern()` is related to the **String Pool**.

Example:

```java
String s1 = new String("Hello");
String s2 = s1.intern();
```

`intern()` returns the canonical pooled String representation.

Conceptually:

```text
Heap String
    ↓
intern()
    ↓
String Pool reference
```

If an equivalent String already exists in the pool, that pooled String is reused.

Therefore:

```java
String s1 = new String("Hello");
String s2 = s1.intern();

s2 == "Hello"; // true
```

### Important

`intern()` does not make the original heap String object disappear. It gives you the pooled reference.

---

# 14. `String.format()`

Static method used to build formatted Strings.

```java
String name = "Aditya";
int age = 28;

String result = String.format(
    "Hello %s, your age is %d",
    name,
    age
);
```

Output:

```text
Hello Aditya, your age is 28
```

Common placeholders:

```text
%s → String
%d → integer
%f → floating-point
```

Useful when String concatenation becomes difficult to read.

---

# 15. StringBuilder

`StringBuilder` is used when **frequent String modification** is required.

```java
StringBuilder sb = new StringBuilder("Hello");
```

### Properties

```text
Mutable
Not thread-safe
Generally faster than StringBuffer
```

Unlike String:

```java
sb.append(" World");
```

modifies the same `StringBuilder` object instead of repeatedly creating immutable Strings.

---

# 16. Important StringBuilder Methods

### `append()`

```java
sb.append(" Java");
```

Adds data at the end.

### `insert()`

```java
sb.insert(5, " Java");
```

Inserts data at a specific index.

### `delete()`

```java
sb.delete(0, 5);
```

Start index inclusive, end index exclusive.

### `deleteCharAt()`

```java
sb.deleteCharAt(2);
```

Removes one character.

### `replace()`

```java
sb.replace(0, 5, "Hi");
```

Replaces a range.

### `reverse()`

```java
sb.reverse();
```

Reverses the contents.

### `charAt()`

```java
sb.charAt(2);
```

Reads a character.

### `setCharAt()`

```java
sb.setCharAt(2, 'X');
```

Changes a character.

### `length()`

```java
sb.length();
```

Returns current number of characters.

### `capacity()`

```java
sb.capacity();
```

Returns current internal buffer capacity.

### `ensureCapacity()`

```java
sb.ensureCapacity(100);
```

Ensures minimum capacity.

### `trimToSize()`

```java
sb.trimToSize();
```

Reduces capacity to approximately the current required size.

### `toString()`

```java
String s = sb.toString();
```

Converts the mutable builder into an immutable String.

---

# 17. `length()` vs `capacity()`

For `StringBuilder`:

```text
length   → actual characters currently stored
capacity → available internal buffer size
```

Example:

```text
length   = 5
capacity = 16
```

They are not the same thing.

When more capacity is required, the internal buffer can grow.

---

# 18. StringBuilder vs StringBuffer

Both are **mutable** String-manipulation classes.

## StringBuilder

```text
Mutable
Not thread-safe
Generally faster
```

## StringBuffer

```text
Mutable
Thread-safe
Generally slower than StringBuilder
```

### Main reason

`StringBuffer` synchronizes its methods.

This adds synchronization overhead but provides thread safety when the same object is accessed by multiple threads.

---

# 19. Race Condition Concept

Suppose multiple threads modify the same mutable object:

```text
Thread 1 → append("A")
Thread 2 → append("B")
```

Without proper synchronization, operations can interfere with each other.

This can lead to a **race condition**.

`StringBuffer` synchronizes its mutation methods to provide thread-safe access.

`StringBuilder` does not provide that built-in synchronization.

---

# 20. Practical Choice

```text
Need immutable text
        → String

Need frequent String modifications
        → StringBuilder

Need frequent modifications + built-in thread safety
        → StringBuffer
```

In typical modern application code, `StringBuilder` is preferred when synchronization is not required because of its lower overhead.

---

# 21. String vs StringBuilder vs StringBuffer

| Feature                            | String          | StringBuilder    | StringBuffer     |
| ---------------------------------- | --------------- | ---------------- | ---------------- |
| Mutable                            | ❌               | ✅                | ✅                |
| Thread-safe due to synchronization | N/A (immutable) | ❌                | ✅                |
| Frequent modification              | Poor choice     | ✅                | ✅                |
| General mutation performance       | N/A             | Generally faster | Generally slower |
| Content-based `equals()`           | ✅               | ❌                | ❌                |

### Important `equals()` trap

```java
StringBuilder a = new StringBuilder("Java");
StringBuilder b = new StringBuilder("Java");

a.equals(b); // false
```

`StringBuilder` does not override `equals()` for content comparison, so reference-based behavior from `Object` applies.

The same applies to `StringBuffer`.

---

# 22. Core Revision Cheat Sheet

```text
String
→ Immutable
→ String Pool for literals
→ equals() compares content
→ == compares references

length()
→ String length

isEmpty()
→ length == 0

isBlank()
→ empty or whitespace only

charAt()
→ character at index

toCharArray()
→ String → char[]

contains()
→ checks for sequence

indexOf()
→ first occurrence

lastIndexOf()
→ last occurrence

startsWith()
→ prefix check

endsWith()
→ suffix check

substring()
→ extracts part of String
→ start inclusive, end exclusive

toUpperCase()
→ uppercase

toLowerCase()
→ lowercase

trim()
→ removes leading/trailing whitespace

strip()
→ Unicode-aware stripping

repeat()
→ repeats String

replace()
→ simple replacement

replaceAll()
→ regex-based replacement

split()
→ String → String[]

String.join()
→ multiple values → one String

String.valueOf()
→ value → String

getBytes()
→ String → byte[]

intern()
→ pooled/canonical String reference

String.format()
→ formatted String

StringBuilder
→ mutable + not thread-safe

StringBuffer
→ mutable + thread-safe
→ synchronized
```

## Most Important Mental Model

```text
String
   ↓
Immutable text

StringBuilder
   ↓
Mutable text
   ↓
Fast
   ↓
Not thread-safe

StringBuffer
   ↓
Mutable text
   ↓
Thread-safe
   ↓
Synchronization overhead
```

## Final Rule

Don't try to memorize every String method or every overload.

Know:

1. **What operation you need**
2. **Which class provides it**
3. **What it returns**
4. **Whether it changes the existing object**
5. **Whether indexes are inclusive/exclusive**

Use IDE autocomplete and documentation to discover exact overloads when coding.
