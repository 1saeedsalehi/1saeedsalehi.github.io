---

title: git for absent-mindeds
author: saeed
date: 2019-11-25T14:09:00 +0800
categories: [Software Engineering]
tags: [Functional Programming,C#.NET,Functional,Programming Paradigms]
title:   Functional Programming - Part 1 
image:
  path: /assets/img/paradigms.png
  alt: The Evolution of Programming Paradigms

---


This article introduces the concept of functional programming and why it's useful. One of the biggest challenges in software development is complexity. Changes are inevitable, especially when adding new features. These changes can make the code harder to understand, increase development time, and introduce bugs. Moreover, it's almost impossible to change something in a program without causing side effects or unwanted behavior. All of this can slow down development and even cause project failure. 

Imperative programming styles, like object-oriented programming, can help manage complexity—if done right. Abstraction in OOP hides complexity, but there are still challenges when dealing with threads and performance.

### The Problem

C# developers are familiar with object-oriented programming (OOP). We often focus on how to use inheritance, encapsulation, and polymorphism to design our code. However, in OOP, multiple threads might access the same memory area, causing race conditions. While we can write thread-safe code, improving performance can be challenging, especially when refactoring existing code for parallel execution.

### The Solution: Functional Programming

Functional programming (FP) is a paradigm that dates back to before computers, originating from a mathematical theory called *lambda calculus*. It allows functions to calculate results without changing the program’s state, which makes the code simpler to understand, reduces side effects, and makes testing easier.

### Functional Programming Languages

Languages like Lisp, Clojure, Erlang, and Haskell support functional programming. F# is a language in the .NET ecosystem that also embraces FP principles. Interestingly, general-purpose languages like C# are flexible enough to support various paradigms, including functional programming. Since many of us use C# for development, integrating functional programming concepts can help solve some of our problems.

### Key Concepts

In functional programming, a function is similar to a mathematical function: it takes specific inputs and returns the expected output without changing the state. This concept is called *referential transparency* and *function honesty*. 

Note that in C#, a function is not just a method; `Func`, `Action`, and `Delegate` are also types of functions.

### Referential Transparency

Referential transparency means that by looking at a function’s inputs and its name, you should be able to predict what it does. A function should only depend on its inputs and should not be affected by global state. For example:

```csharp
public int CalculateElapsedDays(DateTime from)
{
    DateTime now = DateTime.Now;
    return (now - from).Days;
}
```

This function is **not referentially transparent** because it depends on the global `DateTime.Now`, which changes over time. To fix it, pass `DateTime.Now` as an argument:

```csharp
public static int CalculateElapsedDays(DateTime from, DateTime now) => (now - from).Days;
```

Now the function is referentially transparent because it no longer depends on a global variable.


### Function Honesty
A function should handle all possible inputs and outputs correctly. For example:

```csharp

public int Divide(int numerator, int denominator)
{
    return numerator / denominator;
}
```

Is this function transparent? Yes, but it doesn't cover all cases. If the denominator is zero, it will throw an error:

```csharp
var result = Divide(1, 0);  // Divide by zero error
To solve this, we can prevent division by zero by using a custom type:
```

```csharp
public static int Divide(int numerator, NonZeroInt denominator)
{
    return numerator / denominator.Value;
}
```
Here, `NonZeroInt` is a custom type that only accepts values other than zero.


### Functions as First-Class Values

In functional programming, functions are considered *first-class values*. This means functions can be assigned to variables and passed as arguments to other functions. For example:

```csharp
Func<int, bool> isEven = x => x % 2 == 0;
var list = Enumerable.Range(1, 10);
var evenNumbers = list.Where(isEven);
```

In this case, `isEven` is a function assigned to a variable and passed to Where. This ability is crucial for using higher-order functions.

### Higher-Order Functions (HOF)

A higher-order function is a function that accepts other functions as arguments or returns a function as a result. The Where method in LINQ is a good example:

```csharp
public static IEnumerable<T> Where<T>(this IEnumerable<T> ts, Func<T, bool> predicate)
{
    foreach (T t in ts)
        if (predicate(t))
            yield return t;
}
```
In this example, `Where` takes a function (`predicate`) and applies it to each item in the list. The function passed to `Where` is a **higher-order function**.


### Pure Functions

Pure functions are essentially *mathematical functions* that follow the two concepts we’ve already discussed: 
**referential transparency ** and **function honesty**  

A pure function should never have **side effects**. This means it shouldn't modify __global state__ or use __global variables__ as inputs. Pure functions are easy to test because for a given input, they always return the same output. The order of execution doesn’t matter! These are the core components of a functional program and can be used for parallel execution, lazy evaluation, and memoization.

In the next articles of this series, we will explore functional programming implementations and common patterns in C#.