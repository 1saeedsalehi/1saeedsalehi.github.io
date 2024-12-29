---
title:   Functional Programming - part 2 
author: saeed
date: 2019-11-30T00:00 +0800
categories: [Software Engineering]
tags: [Functional Programming,C#.NET,Functional,Programming Paradigms,immutable,thread-safe,collection]
---

In [the previous part of this article](/posts/functional-programming/), we explored the theoretical concepts of functional programming. In this post, I will focus more on coding and examine the patterns and ideas of functional programming in #C.

### Immutable Types
When creating a new type, we should strive to make its internal data as immutable as possible. Even if we need to return an object, it's better to return a new instance rather than modifying the existing one. This will ultimately lead to more clarity and thread safety.

*Example:*

```csharp
public class Rectangle
{
    public int Length { get; set; }
    public int Height { get; set; }

    public void Grow(int length, int height)
    {
        Length += length;
        Height += height;
    }
}

Rectangle r = new Rectangle();
r.Length = 5;
r.Height = 10;
r.Grow(10, 10); // r.Length is 15, r.Height is 20, same instance of r

```


In this example, the class properties can be set from the outside, and anyone calling this class has no idea about the acceptable values. After modification, the responsibility of creating the output object should lie with the function itself to avoid unintended conditions:

```csharp
// After
public class ImmutableRectangle
{
    public int Length { get; }
    public int Height { get; }

    public ImmutableRectangle(int length, int height)
    {
        Length = length;
        Height = height;
    }

    public ImmutableRectangle Grow(int length, int height) =>
          new ImmutableRectangle(Length + length, Height + height);
}

ImmutableRectangle r = new ImmutableRectangle(5, 10);
r = r.Grow(10, 10); // r.Length is 15, r.Height is 20, new instance of r

```

With this change, anyone creating an object of the `ImmutableRectangle` class must provide values, and the properties are accessible as *read-only* from outside the class. Also, in the `Grow` method, a new object of the class is returned, which has no connection to the current instance.


### Using Expressions Instead of Statements

One key aspect of functional programming style can be seen in the following example:

```csharp
public static void Main()
{
    Console.WriteLine(GetSalutation(DateTime.Now.Hour));
}

// imperative, mutates state to produce a result
/*public static string GetSalutation(int hour)
{
    string salutation;

    if (hour < 12)
        salutation = "Good Morning";
    else
        salutation = "Good Afternoon";

    return salutation; 
}*/

public static string GetSalutation(int hour) => hour < 12 ? "Good Morning" : "Good Afternoon";
```

Notice the commented-out lines; a variable is defined to hold the output value, which is mutated. This is unnecessary. We can convert this into an expression that is shorter and more readable.


### Using Higher-Order Functions for Efficiency
in the previous part, we discussed higher-order functions (HOF). In short, these are functions that take another function as an input and return a function as output. Consider the following example:

```csharp
public static int Count<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate)
{
    int count = 0;

    foreach (TSource element in source)
    {
        checked
        {
            if (predicate(element))
            {
                count++;
            }
        }
    }

    return count;
}
```
This code corresponds to the `Count` method in the LINQ library. This method counts items based on a specific condition. We can either implement counting for each specific condition or write a function that takes the condition as input and returns the count.


### Function Composition
Function composition refers to the process of combining multiple simple functions to create more complex functions, just like in mathematics. The output of one function becomes the input for the next function, and in the end, we receive the output of the final function. In C#, we can compose functions in a functional programming style. 

Here's an example:

```csharp
public static class Extensions
{
    public static Func<T, TReturn2> Compose<T, TReturn1, TReturn2>(this Func<TReturn1, TReturn2> func1, Func<T, TReturn1> func2)
    {
        return x => func1(func2(x));
    }
}

public class Program
{
    public static void Main(string[] args)
    {
        Func<int, int> square = (x) => x * x;
        Func<int, int> negate = x => x * -1;
        Func<int, string> toString = s => s.ToString();
        Func<int, string> squareNegateThenToString = toString.Compose(negate).Compose(square);
        Console.WriteLine(squareNegateThenToString(2));
    }
}
```
In the above example, we have three separate functions that we want to apply one after another. We could have written them nested, but that would significantly reduce readability. Instead, we used an extension method for function composition.

### Chaining / Pipe-Lining and Extensions
One important approach in functional programming is method chaining, where the output of one method is passed as input to the next method. A good example of this is the `StringBuilder` class. `StringBuilder` uses the **Fluent Builder pattern**, allowing method chaining without mutating the underlying string. We can achieve the same result with extension methods. Here’s how:

```csharp
string str = new StringBuilder()
  .Append("Hello ")
  .Append("World ")
  .ToString()
  .TrimEnd()
  .ToUpper();

```

In this example, we extended the StringBuilder class with an extension method:

```csharp
public static class Extensions
{
    public static StringBuilder AppendWhen(this StringBuilder sb, string value, bool predicate) => predicate ? sb.Append(value) : sb;
}

public class Program
{
    public static void Main(string[] args)
    {
        string htmlButton = new StringBuilder().Append("<button").AppendWhen(" disabled", false).Append(">Click me</button>").ToString();
    }
}
```

### Avoid Creating Extra Types, Use yield Instead!
Sometimes we need to return a list of items from a method. The first choice is often to create an object of type List or a more general collection, then return it as the output type:

```csharp
public static void Main()
{
    int[] a = { 1, 2, 3, 4, 5 };

    foreach (int n in GreaterThan(a, 3))
    {
        Console.WriteLine(n);
    }
}

/*public static IEnumerable<int> GreaterThan(int[] arr, int gt)
{
    List<int> temp = new List<int>();
    foreach (int n in arr)
    {
        if (n > gt) temp.Add(n);
    }
    return temp;
}*/

public static IEnumerable<int> GreaterThan(int[] arr, int gt)
{
    foreach (int n in arr)
    {
        if (n > gt) yield return n;
    }
}
```
In the first example, we used a temporary list to hold items, but we can avoid this by using the `yield` keyword. This pattern of iteration is common in functional programming.

### Declarative Programming Instead of Imperative Using LINQ
In the previous part, we discussed imperative programming. Here’s an example of converting an imperative method to a declarative one. You can see how much shorter and more readable it becomes:

```csharp
List<int> collection = new List<int> { 1, 2, 3, 4, 5 };

// Imperative style of programming is verbose
List<int> results = new List<int>();

foreach(var num in collection)
{
  if (num % 2 != 0) results.Add(num);
}

// Declarative is terse and beautiful
var results = collection.Where(num => num % 2 != 0);
```


### Immutable Collections
We've discussed the importance of immutability before. Immutable collections are collections where their members cannot be modified after creation. When an item is added or removed, a new collection is returned. You can find different types of these collections in [this link](https://docs.microsoft.com/en-us/dotnet/api/system.collections.immutable?view=netcore-2.2) While creating a new collection may seem like it adds overhead to memory usage, this is not always the case. For example, if you have `f(x) = y`, x and y are likely to share memory. They don’t need to be mutable. For more details,[ this article](https://msdn.microsoft.com/en-us/magazine/mt795189.aspx) goes into a deeper explanation of how these collections are implemented. Eric Lepert has a series of articles on immutability in #C which you can find [here](https://stackoverflow.com/a/927219/1650277).


### Thread-Safe Collections

If we are writing a concurrent or async program, one of the problems we might encounter is a race condition. This occurs when two threads simultaneously attempt to access or modify the same resource. To solve this issue, we can define the objects we work with as immutable. 

Starting from .NET Framework version 4, [Concurrent Collections](https://docs.microsoft.com/en-us/dotnet/standard/collections/thread-safe/) were introduced. Some of the commonly used types are listed below:

| Type | Description |
|---|---|
| [ConcurrentDictionary](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentdictionary-2) | Thread-safe implementation of a key-value dictionary |
| [ConcurrentQueue](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentqueue-1) | Thread-safe implementation of a queue (first in, first out) |
| [ConcurrentStack](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentstack-1) | Thread-safe implementation of a stack (last in, first out) |
| [ConcurrentBag](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentbag-1) | Thread-safe implementation of an unordered list |

These classes won't solve all our problems, but it’s good to be aware of them so we can use them appropriately and in the right context.
