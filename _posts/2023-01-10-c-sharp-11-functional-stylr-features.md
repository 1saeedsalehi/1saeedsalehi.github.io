---
title: C# 11 functional style features
author: Saeed Salehi
date: 2023-01-10T00:00 +0800
categories: [Software Engineering]
tags: [Functional Programming,C#.NET,Functional,Programming Paradigms,null, nullable reference type]
---

Functional programming has been gaining popularity in recent years, and many programming languages have been adding features to support it. C# 11 is no exception, and it comes with several features that make it more functional. In this blog post, we'll take a look at some of these features and provide examples of how to use them.

### Record Types
Record types are a new type of class introduced in C# 9, and C# 11 has added some additional features to make them more functional. Records provide a way to define a type that is immutable by default and provides built-in value equality semantics. You can use records to represent data without worrying about accidental mutation or the need to implement value equality manually.

Here's an example of a record type:

```csharp
public record Person(string FirstName, string LastName, int Age);
```

This record type defines a person's first name, last name, and age. The record type is immutable by default, and you can create a new instance of this record type using the following code:


```csharp
var person = new Person("John", "Doe", 30);
```

### Init-Only Properties

Init-only properties are another new feature introduced in C# 9 that allows you to initialize a property's value when it is created and then make it read-only. C# 11 extends this feature to allow for init-only setters in interfaces and base classes.

Here's an example of an interface with an init-only property:

```csharp
public interface IShape
{
    public int Sides { get; init; }
}
```

In this example, the Sides property is initialized when the object is created, and it can be read, but it cannot be modified after initialization.

### Lambda Discards

Lambda discards are a new feature in C# 11 that allows you to use a discard (underscore) as a parameter in a lambda expression. This feature is useful when you want to ignore one or more parameters in a lambda expression.

Here's an example of a lambda expression with a discard parameter:

```csharp
var list = new List<int>() { 1, 2, 3, 4, 5 };
var evens = list.Where(_ => _ % 2 == 0);
```

In this example, the lambda expression ignores the parameter passed to it and only evaluates the even numbers in the list.

### Global Using Directives

Global using directives are another new feature in C# 11 that allows you to add using directives at the project level instead of having to add them to each file. This feature is useful when you have multiple files in a project that require the same using directives.

Here's an example of a global using directive:

```csharp
global using System.Linq;
```

In this example, the global using directive allows you to use LINQ methods in all files within the project without having to add a using directive to each file.

### Nullable Reference Types
Nullable reference types were introduced in C# 8, but C# 11 has added some additional features to make them more functional. Nullable reference types allow you to declare whether a reference type can be null or not.

Here's an example of a nullable reference type:

```csharp
string? nullableString = null;
```

In this example, the nullableString variable can be assigned a value of null because it has been declared as nullable.

### Conclusion

C# 11 has added several new features that make it more functional. These features include record types, init-only properties, lambda discards, global using directives, and nullable reference types. These features make it easier to write functional code in C#