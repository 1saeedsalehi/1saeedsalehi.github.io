---
title: Functional Programming - part 5
author: saeed
date: 2020-04-01T00:00 +0800
categories: [Software Engineering]
tags: [Functional Programming,C#.NET,Functional,Programming Paradigms,primitive-obsession]
image: 
    path: /assets/img/functional-programming/primitive-obsession/dry.png
    alt: exceptions
---


## Obsession with Primitive Types

Continuing the series of articles related to functional programming, today I want to examine whether or not we should use [primitive types](https://wiki.c2.com/?PrimitiveObsession) in our code. 

If you haven't read the previous parts, I recommend starting with the following:

- [Part 1: Introduction to Concepts](/posts/functional-programming/)
- [Part 2: Practical Examples](/posts/functional-programming-2-examples/)
- [Part 3: Immutability](/posts/functional-programming-3-refactoring-to-immutable/)
- [Part 4: Dealing with Exceptions](/posts/functional-programming-4-stay-away-from-exception/)

In domain modeling, we often use primitive types like `int`, `string`, and so on. In fact, we can say that we have an obsession with these primitive data types. 

Consider the following code snippet:

```csharp
public class UserFactory
{
    public User CreateUser(string email) { 
        return new User(email);
    }
}
```

The `UserFactory` class has a method called CreateUser that takes a string (an email) as input and returns a `User` object. So, what's the issue with this method?

If you recall from previous parts, we talked about the concept of **"Honesty"**. Simply put, we should be able to understand the function's purpose and output just from its signature.

*This method is not honest*. The function signature does not indicate that the string could be invalid, empty, have an incorrect length, and so on. We can't infer these possible issues from just the method signature.

![dishonest method signature](/assets/img/functional-programming/primitive-obsession/dishonest-method-signature.png)

To make this clearer, keep the example above in mind. In the `Divide` function, which performs division, the parameter `y` (of type `int`) could be zero, causing an exception. Since the method returns an `int`, we don't expect it to throw an `exception`. We've already discussed exceptions in detail in the previous part.

Now, imagine you want to use multiple emails as inputs instead of just one. Should you repeat the validation logic for each input parameter?


![dishonest method signature](/assets/img/functional-programming/primitive-obsession/dry.png)

Using primitive types recklessly can lead to losing the honesty of our methods and violating the **DRY (Don't Repeat Yourself) ** principle.

The discussion about when to use or not use primitive types has many angles. One such approach is discussed in **Domain-Driven Design (DDD)** under the concept of **Value Objects**. The goal of this section is simply to touch on one aspect of this issue, but for further reading and more in-depth information, you can refer to the book **Domain-Driven Design: Tackling Complexity in the Heart of Software by Eric Evans**.


### What Should We Use Instead of Primitive Types?
The answer is quite simple: You need to create a dedicated type. For example, instead of using a string for email, you can create a class called `Email` that has a property named Value.

There are various ways to achieve this, but my suggestion is to use the following method:

<script src="https://gist.github.com/1saeedsalehi/e2b454a3be06fb81a5e9f2782f316991.js"></script>
In this approach, we create a class as a `Value Object`, which encapsulates the primitive type we are working with. The class implements logic for comparison and operators like `==` and `!=` through `.Equals` and `.GetHashCode()`.

For instance, for the `Email` class, we can implement it as follows:

```csharp
public class EmailAddress : ValueOf<string, EmailAddress> { }
```

You can instantiate this class like so:

```csharp
EmailAddress emailAddress = EmailAddress.From("foo@bar.com");
```

For more complex examples, such as an address that includes a street, postcode, etc., you can use C#'s `Tuple` feature (introduced in C# 7) like this:

```csharp
public class Address : ValueOf<(string firstLine, string secondLine, Postcode postcode), Address> { }
```
Finally, for implementing validation logic, you can override the Validate method and avoid violating the DRY principle.

The method outlined in this article is simply to familiarize you with cleaner code using functional programming concepts. In real-world applications, you'll likely face issues with storing these objects or working with libraries like Entity Framework, but these are straightforward challenges to resolve.

If you encounter any issues during implementation, feel free to leave a comment under this post or on the [gist](https://gist.github.com/1saeedsalehi/e2b454a3be06fb81a5e9f2782f316991).

