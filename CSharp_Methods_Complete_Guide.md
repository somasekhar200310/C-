# C# Methods — Complete Guide

> Companion notes for Vishnu's C++ → .NET transition. Each topic includes a C++ comparison where relevant, explanation, code examples, and key takeaways.

---

## Table of Contents
1. [Introduction to Methods](#1-introduction-to-methods)
2. [Method Parameters](#2-method-parameters)
3. [Return Types and void Methods](#3-return-types-and-void-methods)
4. [Method Overloading](#4-method-overloading)
5. [Optional Parameters and Named Arguments](#5-optional-parameters-and-named-arguments)
6. [Static vs Instance Methods](#6-static-vs-instance-methods)
7. [Local Functions (Nested Methods)](#7-local-functions-nested-methods)
8. [Extension Methods](#8-extension-methods)
9. [Async and Await in Methods](#9-async-and-await-in-methods)
10. [Lambda Expressions and Anonymous Methods](#10-lambda-expressions-and-anonymous-methods)
11. [Method Signatures and Overriding](#11-method-signatures-and-overriding)
12. [Access Modifiers in Methods](#12-access-modifiers-in-methods)
13. [Partial Methods](#13-partial-methods)

---

## 1. Introduction to Methods

A **method** is a block of code that performs a specific task, declared inside a class or struct. Unlike C++, there are **no free-floating/global functions** — every method belongs to some type.

### Basic Anatomy

```csharp
[access modifier] [static] [return type] MethodName([parameters])
{
    // method body
    return value; // if return type is not void
}
```

### Example

```csharp
public int Add(int a, int b)
{
    return a + b;
}
```

### C++ vs C#

| Aspect | C++ | C# |
|---|---|---|
| Global functions | Allowed | Not allowed — must be in a class |
| Declaration/definition split | Common (`.h` / `.cpp`) | Not needed — single location |
| Default access | depends on context | `private` if unspecified (inside class) |

### Key Takeaways
- Every method lives inside a class, struct, or interface.
- No header files — declaration and implementation are combined.
- `Main` is the entry point of a console application and must be `static`.

---

## 2. Method Parameters

Parameters allow you to pass data into a method. C# supports several parameter-passing mechanisms beyond plain pass-by-value.

### 2.1 Value Parameters (default)

```csharp
void Increment(int x)
{
    x++; // only modifies the local copy
}

int num = 5;
Increment(num);
Console.WriteLine(num); // 5 — unchanged
```

### 2.2 `ref` Parameters

Passes the variable **by reference** — changes inside the method affect the caller's variable. The variable **must be initialized** before being passed.

```csharp
void Double(ref int x)
{
    x *= 2;
}

int num = 5;
Double(ref num);
Console.WriteLine(num); // 10
```

### 2.3 `out` Parameters

Also passed by reference, but the variable does **not need to be initialized** beforehand — the method is required to assign it before returning.

```csharp
void GetMinMax(int[] arr, out int min, out int max)
{
    min = arr[0];
    max = arr[0];
    foreach (int n in arr)
    {
        if (n < min) min = n;
        if (n > max) max = n;
    }
}

GetMinMax(new[] { 4, 1, 9, 2 }, out int min, out int max);
Console.WriteLine($"Min: {min}, Max: {max}");
```

### 2.4 `in` Parameters

Passed by reference but **read-only** inside the method — useful for large structs to avoid copy overhead while preventing modification.

```csharp
struct BigData { public int A, B, C, D, E; }

void PrintData(in BigData data)
{
    Console.WriteLine(data.A); // OK — read
    // data.A = 10;            // ERROR — cannot modify 'in' parameter
}
```

### 2.5 `params` (Variable-Length Arguments)

Allows passing any number of arguments of a given type, collected into an array.

```csharp
int SumAll(params int[] numbers)
{
    int total = 0;
    foreach (int n in numbers)
        total += n;
    return total;
}

SumAll(1, 2, 3);       // 6
SumAll(1, 2, 3, 4, 5); // 15
SumAll();              // 0
```

> `params` must be the **last** parameter in the method signature.

### C++ Comparison

| C# | C++ Equivalent |
|---|---|
| `ref` | `&` reference parameter |
| `out` | `&` reference (or pointer), used as output-only |
| `in` | `const T&` |
| `params` | variadic templates / `...` (much more verbose in C++) |

### Key Takeaways
- Default is **pass-by-value**.
- `ref` = must be initialized, two-way.
- `out` = need not be initialized, method must assign it.
- `in` = read-only reference, good for performance with large structs.
- `params` = variable argument count, must be last parameter.

---

## 3. Return Types and void Methods

### 3.1 Returning a Value

```csharp
int Square(int x)
{
    return x * x;
}
```

The return type must match (or be implicitly convertible to) the declared type.

### 3.2 `void` Methods

A method that performs an action but returns nothing.

```csharp
void LogMessage(string message)
{
    Console.WriteLine($"[LOG]: {message}");
    // no return statement needed
}
```

You *can* use a bare `return;` inside a void method to exit early:

```csharp
void Process(int value)
{
    if (value < 0)
    {
        Console.WriteLine("Invalid value");
        return; // exits early
    }
    Console.WriteLine($"Processing {value}");
}
```

### 3.3 Returning Multiple Values

C# doesn't allow multiple return values directly, but you can use:

**a) `out` parameters** (seen above)

**b) Tuples** (modern, preferred)

```csharp
(int min, int max) GetMinMax(int[] arr)
{
    return (arr.Min(), arr.Max());
}

var result = GetMinMax(new[] { 3, 7, 1, 9 });
Console.WriteLine($"Min: {result.min}, Max: {result.max}");
```

**c) Custom classes/records**

```csharp
record Stats(int Min, int Max, double Average);

Stats GetStats(int[] arr)
{
    return new Stats(arr.Min(), arr.Max(), arr.Average());
}
```

### Key Takeaways
- `void` = no return value, but `return;` can exit early.
- Tuples are the modern, lightweight way to return multiple values.
- Records/classes are better when the returned data has clear meaning/identity.

---

## 4. Method Overloading

Multiple methods can share the same name as long as their **parameter lists differ** (by number, type, or order of parameters).

```csharp
int Add(int a, int b) => a + b;
double Add(double a, double b) => a + b;
int Add(int a, int b, int c) => a + b + c;
string Add(string a, string b) => a + b;
```

### Rules
- Return type **alone** cannot differentiate overloads — parameters must differ.
- `ref`/`out`/`in` modifiers count toward the signature, but two overloads differing **only** by `ref` vs `out` are not allowed (ambiguous to callers).

```csharp
// NOT allowed — differ only in out vs ref
void Process(ref int x) { }
void Process(out int x) { x = 0; } // ERROR
```

### Overload Resolution
The compiler picks the **best match** based on argument types, applying implicit conversions if needed:

```csharp
Add(2, 3);        // int overload
Add(2.5, 3.5);    // double overload
Add(2, 3, 4);     // 3-arg overload
Add("Hello, ", "World"); // string overload
```

### C++ Comparison
Function overloading works almost identically in C++. The main difference: C# has no operator overloading at the global/free-function level (overloaded operators must be defined inside the class using `operator` keyword, similar to C++ member operator overloads).

### Key Takeaways
- Overloads must differ in parameter type, count, or order — not just return type.
- The compiler resolves overloads at compile time based on argument types.

---

## 5. Optional Parameters and Named Arguments

### 5.1 Optional Parameters

Provide a default value — caller may omit the argument.

```csharp
void CreateUser(string name, int age = 18, string city = "Unknown")
{
    Console.WriteLine($"{name}, {age}, {city}");
}

CreateUser("Vishnu");                       // Vishnu, 18, Unknown
CreateUser("Vishnu", 28);                   // Vishnu, 28, Unknown
CreateUser("Vishnu", 28, "Visakhapatnam");  // Vishnu, 28, Visakhapatnam
```

**Rule:** Optional parameters must come **after** all required parameters.

```csharp
// ERROR — required parameter after optional
void Foo(int a = 1, int b) { }
```

### 5.2 Named Arguments

Allows specifying arguments by parameter name, in any order, and skipping optional ones.

```csharp
CreateUser("Vishnu", city: "Visakhapatnam"); // skips 'age', uses default
CreateUser(age: 30, name: "Asha");           // order doesn't matter
```

### Combining Both

```csharp
void Configure(string name, bool verbose = false, int retries = 3, string logLevel = "INFO")
{
    Console.WriteLine($"{name}, verbose={verbose}, retries={retries}, log={logLevel}");
}

Configure("Service A", logLevel: "DEBUG");
```

### C++ Comparison

| Feature | C++ | C# |
|---|---|---|
| Default arguments | ✅ (declared in header) | ✅ |
| Named arguments | ❌ | ✅ |
| Skipping middle defaults | ❌ — must provide all preceding args | ✅ — via named arguments |

### Key Takeaways
- Optional parameters reduce the need for many overloads.
- Named arguments improve readability, especially with multiple optional/boolean parameters.
- Optional parameters must be declared last.

---

## 6. Static vs Instance Methods

This is one of the **most important distinctions** in C# OOP.

### 6.1 Instance Methods

Belong to an **object** (instance) of the class. They can access instance fields/properties (`this`).

```csharp
class Calculator
{
    int memory;

    public void Store(int value)
    {
        memory = value; // accesses instance field
    }

    public int Recall()
    {
        return memory;
    }
}

Calculator calc = new Calculator();
calc.Store(10);
Console.WriteLine(calc.Recall()); // 10
```

### 6.2 Static Methods

Belong to the **class itself**, not any instance. Cannot directly access instance members. Called using the class name.

```csharp
class MathHelper
{
    public static int Square(int x) => x * x;
}

int result = MathHelper.Square(5); // no instance needed
```

### 6.3 The Classic Pitfall

```csharp
class Program
{
    void Sum(int x, int y) // instance method
    {
        Console.WriteLine(x + y);
    }

    static void Main(string[] args) // static method
    {
        Sum(8, 9); // ERROR: An object reference is required
    }
}
```

**Fix 1 — Make `Sum` static:**
```csharp
static void Sum(int x, int y) => Console.WriteLine(x + y);
```

**Fix 2 — Create an instance:**
```csharp
Program p = new Program();
p.Sum(8, 9);
```

### 6.4 Static Fields and Constructors

```csharp
class Counter
{
    public static int Count = 0; // shared across all instances

    public Counter()
    {
        Count++;
    }
}

new Counter();
new Counter();
Console.WriteLine(Counter.Count); // 2
```

### Quick Reference Table

| | Static | Instance |
|---|---|---|
| Belongs to | the class | an object |
| Access via | `ClassName.Member` | `objectName.Member` |
| Can access instance fields? | No | Yes |
| `this` keyword available? | No | Yes |
| Memory | One copy shared by all | One copy per object |
| `Main` method | Always static | — |

### C++ Comparison

| C# | C++ |
|---|---|
| `static` method | `static` member function |
| `static` field | `static` member variable |
| Same semantics — one shared copy, accessed via class name | Same |

### Key Takeaways
- `static` = belongs to the type; `instance` = belongs to an object.
- A static method cannot call instance methods without an object reference.
- Utility/helper methods that don't need object state are typically `static` (e.g., `Math.Sqrt`, `Convert.ToInt32`).

---

## 7. Local Functions (Nested Methods)

A **local function** is a method defined **inside** another method — useful for helper logic only relevant to that method.

```csharp
int CalculateTotal(int[] prices)
{
    int total = 0;
    foreach (int p in prices)
        total += ApplyDiscount(p);
    return total;

    // Local function — only visible inside CalculateTotal
    int ApplyDiscount(int price) => price > 100 ? price - 10 : price;
}
```

### Characteristics
- Can access variables of the enclosing method (closure-like behavior).
- Not visible outside the containing method.
- Can be recursive.

```csharp
int Factorial(int n)
{
    return Fact(n);

    int Fact(int x) => x <= 1 ? 1 : x * Fact(x - 1);
}
```

### Local Functions vs Lambdas

| | Local Function | Lambda Expression |
|---|---|---|
| Syntax | method-like | `(params) => expr` |
| Can be recursive easily | Yes | Awkward |
| Performance | Slightly better (no delegate allocation in simple cases) | Allocates a delegate |
| Named | Yes | Anonymous (unless assigned to a variable) |

### C++ Comparison
C++ doesn't support true nested named functions (only lambdas, introduced in C++11). Local functions in C# give you named, reusable helper logic scoped to a method — closer to nested lambdas but more readable.

### Key Takeaways
- Use local functions for helper logic that's only relevant within one method.
- They can capture variables from the enclosing scope.
- Prefer them over private helper methods when the logic has no use outside the containing method.

---

## 8. Extension Methods

Extension methods let you **add new methods to existing types** (including types you don't own, like `string` or `int[]`) **without modifying their source code** or using inheritance.

### Syntax Requirements
- Must be defined in a `static` class.
- Must be a `static` method.
- The **first parameter** uses the `this` keyword, specifying the type being extended.

```csharp
public static class StringExtensions
{
    public static bool IsPalindrome(this string str)
    {
        string reversed = new string(str.Reverse().ToArray());
        return str.Equals(reversed, StringComparison.OrdinalIgnoreCase);
    }
}

// Usage — looks like a built-in method!
string word = "Madam";
Console.WriteLine(word.IsPalindrome()); // True
```

### Another Example — extending `int`

```csharp
public static class IntExtensions
{
    public static bool IsEven(this int number) => number % 2 == 0;
}

int x = 10;
Console.WriteLine(x.IsEven()); // True
```

### How It Works
The compiler translates `word.IsPalindrome()` into `StringExtensions.IsPalindrome(word)` — it's syntactic sugar over a static method call.

### Where You'll See This
LINQ itself is built almost entirely using extension methods on `IEnumerable<T>` (e.g., `.Where()`, `.Select()`, `.OrderBy()`).

```csharp
using System.Linq;

int[] numbers = { 1, 2, 3, 4, 5, 6 };
var evens = numbers.Where(n => n % 2 == 0); // Where is an extension method!
```

### C++ Comparison
No direct equivalent — closest analogy is free functions taking the object as a parameter, but extension methods provide the **calling syntax** of an instance method (`obj.Method()`), improving readability and discoverability via IntelliSense.

### Key Takeaways
- Extension methods "add" methods to types you can't modify.
- Defined as `static` methods in `static` classes, first parameter prefixed with `this`.
- Heavily used throughout .NET (especially LINQ) — you'll use them constantly even if you never write your own.

---

## 9. Async and Await in Methods

Asynchronous programming lets a method **start a long-running operation** (like a file read, database call, or HTTP request) **without blocking the calling thread**.

### 9.1 Basic Syntax

```csharp
async Task<string> GetDataAsync()
{
    await Task.Delay(2000); // simulates a 2-second operation (e.g., network call)
    return "Data loaded";
}
```

- `async` keyword marks the method as asynchronous.
- `await` pauses execution **without blocking the thread** until the awaited task completes.
- Return types: `Task` (no return value), `Task<T>` (returns `T`), or `void` (only for event handlers — avoid otherwise).

### 9.2 Calling an Async Method

```csharp
async Task Main() // top-level async Main (C# 7.1+)
{
    Console.WriteLine("Fetching data...");
    string result = await GetDataAsync();
    Console.WriteLine(result);
}
```

### 9.3 Why Not Just Use Threads (like C++ `std::thread`)?

| | C++ `std::thread` | C# `async`/`await` |
|---|---|---|
| Model | OS thread per task | Single thread, cooperative yielding |
| Overhead | Higher (thread creation) | Lower (no new thread for I/O-bound work) |
| Best for | CPU-bound parallel work | I/O-bound work (DB, files, network, web APIs) |

`async`/`await` doesn't necessarily create new threads — for I/O-bound operations, the thread is **freed up** while waiting, then resumes when the operation completes.

### 9.4 Real Example — Calling a Web API

```csharp
using System.Net.Http;

async Task<string> FetchWebsiteAsync(string url)
{
    using HttpClient client = new HttpClient();
    string content = await client.GetStringAsync(url);
    return content;
}
```

### 9.5 Multiple Async Calls

```csharp
async Task ProcessAllAsync()
{
    Task<string> task1 = GetDataAsync();
    Task<string> task2 = GetDataAsync();

    // Run concurrently, wait for both
    string[] results = await Task.WhenAll(task1, task2);
}
```

### 9.6 Exception Handling

```csharp
try
{
    string data = await GetDataAsync();
}
catch (Exception ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
```

Standard `try/catch` works seamlessly with `await` — no special async-specific syntax needed.

### Key Takeaways
- `async` marks a method that contains asynchronous operations.
- `await` suspends execution until the awaited `Task` completes, without blocking the thread.
- Return `Task` / `Task<T>` (not `void`, except for event handlers).
- Critical for ASP.NET Core (Web API calls, database queries via Entity Framework, etc.) — you'll use this constantly.

---

## 10. Lambda Expressions and Anonymous Methods

### 10.1 Anonymous Methods (older syntax)

A method without a name, assigned to a delegate.

```csharp
delegate void Greet(string name);

Greet greet = delegate (string name)
{
    Console.WriteLine($"Hello, {name}!");
};

greet("Vishnu");
```

### 10.2 Lambda Expressions (modern, preferred)

A more concise syntax for anonymous methods, using `=>` ("goes to" / "fat arrow").

```csharp
Greet greet = (name) => Console.WriteLine($"Hello, {name}!");
greet("Vishnu");
```

### 10.3 Common Built-in Delegate Types

Instead of declaring custom delegates, use built-in generic delegate types:

```csharp
Func<int, int, int> add = (a, b) => a + b;       // takes params, returns value
Action<string> log = (msg) => Console.WriteLine(msg); // takes params, returns void
Predicate<int> isEven = (n) => n % 2 == 0;       // takes param, returns bool
```

| Delegate Type | Signature |
|---|---|
| `Action` | no params, returns void |
| `Action<T>` | takes `T`, returns void |
| `Func<T, TResult>` | takes `T`, returns `TResult` |
| `Predicate<T>` | takes `T`, returns `bool` |

### 10.4 Lambdas with LINQ

This is where lambdas shine — heavily used with LINQ:

```csharp
int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8 };

var evens = numbers.Where(n => n % 2 == 0);          // filter
var squares = numbers.Select(n => n * n);            // transform
var sum = numbers.Sum(n => n);                        // aggregate
var anyNegative = numbers.Any(n => n < 0);            // check
```

### 10.5 Capturing Variables (Closures)

Lambdas can "capture" variables from their enclosing scope:

```csharp
int factor = 10;
Func<int, int> multiply = x => x * factor; // captures 'factor'

Console.WriteLine(multiply(5)); // 50

factor = 20;
Console.WriteLine(multiply(5)); // 100 — captured by reference!
```

### 10.6 Expression Lambdas vs Statement Lambdas

```csharp
// Expression lambda (single expression)
Func<int, int> square = x => x * x;

// Statement lambda (multiple statements, needs braces and return)
Func<int, int> squarePlusOne = x =>
{
    int result = x * x;
    return result + 1;
};
```

### C++ Comparison

| C# | C++ |
|---|---|
| `Func<int,int,int> add = (a,b) => a+b;` | `auto add = [](int a, int b) { return a+b; };` |
| Captures variables by reference automatically | Must specify `[&]` or `[=]` capture mode |
| `Action`, `Func`, `Predicate` | `std::function` |

### Key Takeaways
- Lambdas are the modern way to write small, inline functions — especially with LINQ.
- `Func`, `Action`, `Predicate` cover most common delegate needs.
- Lambdas capture outer variables by reference (changes to the variable affect the lambda).

---

## 11. Method Signatures and Overriding

### 11.1 What Makes Up a Method Signature?

A method's **signature** = method name + parameter types (count, type, order). It does **not** include the return type or parameter names.

```csharp
void Process(int x)      // signature: Process(int)
void Process(int x, int y) // signature: Process(int, int) — different signature, OK (overload)
int Process(int x)        // ERROR — same signature as first, differs only by return type
```

### 11.2 Overriding — `virtual`, `override`, `base`

To override a method in a derived class, the base method must be marked `virtual` (or `abstract`), and the derived method uses `override`.

```csharp
class Animal
{
    public virtual void Speak()
    {
        Console.WriteLine("Some generic animal sound");
    }
}

class Dog : Animal
{
    public override void Speak()
    {
        Console.WriteLine("Woof!");
    }
}

class Cat : Animal
{
    public override void Speak()
    {
        base.Speak();          // calls Animal's Speak() first
        Console.WriteLine("Meow!");
    }
}

Animal a = new Dog();
a.Speak(); // "Woof!" — runtime polymorphism
```

### 11.3 `new` Keyword — Method Hiding (different from overriding!)

```csharp
class Base
{
    public void Show() => Console.WriteLine("Base.Show");
}

class Derived : Base
{
    public new void Show() => Console.WriteLine("Derived.Show"); // hides, doesn't override
}

Base obj = new Derived();
obj.Show(); // "Base.Show" — because Show() isn't virtual, no polymorphism
```

### 11.4 `abstract` Methods

Declared without implementation; **must** be overridden by non-abstract derived classes.

```csharp
abstract class Shape
{
    public abstract double Area(); // no body
}

class Circle : Shape
{
    public double Radius;
    public override double Area() => Math.PI * Radius * Radius;
}
```

### 11.5 `sealed` — Preventing Further Overrides

```csharp
class Dog : Animal
{
    public sealed override void Speak() => Console.WriteLine("Woof!");
}

class Puppy : Dog
{
    // public override void Speak() // ERROR — Speak is sealed
}
```

### C++ Comparison

| C# | C++ |
|---|---|
| `virtual` | `virtual` |
| `override` | `override` (C++11+) — optional but recommended |
| `abstract` method | pure virtual function (`= 0`) |
| `abstract` class | class with at least one pure virtual function |
| `sealed` | `final` |
| `base.Method()` | `BaseClass::Method()` |
| `new` (method hiding) | shadowing (rare, usually unintentional in C++) |

### Key Takeaways
- A method signature = name + parameter types (not return type).
- `virtual` + `override` enable runtime polymorphism (like C++ `virtual`).
- `abstract` methods have no body and force derived classes to implement them.
- `sealed` prevents further overriding — like C++'s `final`.
- Avoid `new` (method hiding) unless you have a very specific reason — it's a common source of bugs.

---

## 12. Access Modifiers in Methods

Access modifiers control the **visibility/accessibility** of a method from other code.

### 12.1 The Modifiers

| Modifier | Accessible From |
|---|---|
| `public` | Anywhere |
| `private` | Only within the same class |
| `protected` | Same class + derived classes |
| `internal` | Same assembly (project) only |
| `protected internal` | Same assembly OR derived classes (even in other assemblies) |
| `private protected` | Same assembly AND derived classes only |

### 12.2 Examples

```csharp
public class BankAccount
{
    private decimal balance; // only accessible inside BankAccount

    public void Deposit(decimal amount) // accessible from anywhere
    {
        balance += amount;
    }

    protected void LogTransaction(string message) // accessible to derived classes
    {
        Console.WriteLine($"LOG: {message}");
    }

    internal void AuditCheck() // accessible only within the same project/assembly
    {
        Console.WriteLine("Audit performed");
    }
}

public class SavingsAccount : BankAccount
{
    public void ApplyInterest()
    {
        LogTransaction("Interest applied"); // OK — protected, accessible in derived class
    }
}
```

### 12.3 Default Access Levels

- Class members (methods, fields) default to `private` if no modifier is specified.
- Top-level classes default to `internal` if no modifier is specified.

```csharp
class Example
{
    void Foo() { } // implicitly private
}
```

### C++ Comparison

| C# | C++ |
|---|---|
| `public` | `public` |
| `private` | `private` |
| `protected` | `protected` |
| `internal` | No direct equivalent (closest: anonymous namespaces / translation-unit visibility, but C# `internal` is assembly-wide) |
| Default for class members | `private` (same as C++ `class`, unlike `struct` which defaults `public`) |

### Key Takeaways
- `private` = strictest, `public` = least restrictive.
- `internal` is unique to .NET — scoped to the containing assembly (project/DLL), useful for hiding implementation details from other projects.
- Choosing the right access modifier is core to encapsulation — expose only what's necessary.

---

## 13. Partial Methods

Partial methods allow splitting a method's **declaration** and **implementation** across different parts of a **partial class** — often used with code generators (e.g., auto-generated UI code, Entity Framework scaffolding).

### 13.1 Basic Syntax

**File 1 (auto-generated, e.g., `Customer.Generated.cs`):**
```csharp
public partial class Customer
{
    public string Name { get; set; }

    // Declaration only — no body
    partial void OnNameChanged();

    public void SetName(string name)
    {
        Name = name;
        OnNameChanged(); // calls the implementation, if one exists
    }
}
```

**File 2 (your hand-written code, e.g., `Customer.cs`):**
```csharp
public partial class Customer
{
    // Implementation
    partial void OnNameChanged()
    {
        Console.WriteLine($"Name changed to {Name}");
    }
}
```

### 13.2 Rules
- Partial method declarations must return `void` (in classic partial methods; C# 9+ relaxed this for non-`private` partial methods used in source generators).
- If **no implementation** is provided, the compiler **removes the call entirely** — zero runtime overhead.
- Cannot have access modifiers (implicitly `private`) in classic partial methods.
- Cannot have `out` parameters.

### 13.3 Why This Matters

Common in **auto-generated code** scenarios:
- WinForms / WPF designer-generated code
- Entity Framework scaffolded entity classes
- Source generators (modern C#)

This lets tooling generate a class while you add custom logic in a separate file — without editing the generated file (which might get overwritten/regenerated).

### C++ Comparison

No direct equivalent. Closest concept: splitting a class across multiple `.cpp` files isn't really a thing in C++ (a class's methods can be defined in any `.cpp` file that includes the header, but there's no "partial class" concept). C#'s `partial` keyword is specifically designed for code-generation workflows.

### Key Takeaways
- `partial` classes/methods let generated code and hand-written code coexist cleanly.
- If a partial method has no implementation, the call is compiled away — no cost.
- You'll encounter this mostly when working with designer tools or Entity Framework scaffolding — rarely write `partial` methods yourself from scratch.

---

## Summary Cheat Sheet

| Topic | One-Line Takeaway |
|---|---|
| Introduction to Methods | All methods live inside a type — no free functions like C++ |
| Method Parameters | `ref`, `out`, `in`, `params` extend pass-by-value/reference options |
| Return Types | Use tuples or records for multiple return values |
| Overloading | Same name, different parameter list — resolved at compile time |
| Optional/Named Args | Reduce overload bloat; improve call-site readability |
| Static vs Instance | Static = class-level; Instance = object-level (needs `this`) |
| Local Functions | Helper methods scoped inside another method |
| Extension Methods | Add methods to existing types via `static` class + `this` param |
| Async/Await | Non-blocking I/O — critical for ASP.NET Core |
| Lambdas | Concise inline functions, heavily used with LINQ |
| Overriding | `virtual`/`override`/`abstract`/`sealed` for polymorphism |
| Access Modifiers | Control visibility: `private`, `public`, `protected`, `internal`, etc. |
| Partial Methods | Used in generated-code scenarios; zero-cost if unimplemented |

---

*Next recommended topic: **Classes & Objects (OOP in C#)** — this builds directly on Static vs Instance Methods and Overriding/Access Modifiers covered above.*
