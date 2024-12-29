---
title: Handling exceptions in a functional way
author: saeed
date: 2021-11-08T00:00 +0800
categories: [Software Engineering]
tags: [Functional Programming,C#.NET,Functional,Programming Paradigms,exception,c#,null reference,exception]

---

Please see the code below:

```csharp
Customer customer = _repository.GetById(id);
Console.WriteLine(customer.Name);
```
Are you familiar with this code? What problem do you see?

The problem here is that we are not sure if the `getById` method can return null or not. If there is a possibility that this code returns null, we will receive a `NullReferenceException` error when we run the code.

The disaster happens when we may not notice this issue until we test the software with real data/customer environment/production, and we cannot easily determine where it has been turned into null!

The faster we understand, the less time it takes to solve it. Probably the best solution is for the compiler to give us an error!

```csharp
Customer! customer = _repository.GetById(id);
Console.WriteLine(customer.Name);
```
The meaning of `Customer!` is that this is a non-null object. This means that there is no way to convert this object to null in any way.

Can you imagine a world without a null reference exception error? I can't!

This possibility has been provided in C# from version 8 onwards. To see how you can use this feature, see [here](https://docs.microsoft.com/en-us/dotnet/csharp/nullable-references).

But since you may not be able to upgrade to a higher version of C# in your project for any reason, or you may not want to enable this feature in the entire project, we will discuss an alternative solution here.

How can we have a null value?
For this, we need a monad in this example we call it MayBe.

The code in its simplest form can be like this:


```csharp
public struct Maybe<T>
{
    private readonlyT _value;
 
    public T Value
    {
        get
        {
            Contracts.Require(HasValue);
 
            return _value;
        }
    }
 
    public bool HasValue
    {
        get { return _value != null; }
    }
 
    public bool HasNoValue
    {
        get { return !HasValue; }
    }
 
    private Maybe([AllowNull] T value)
    {
        _value = value;
    }
 
    public static implicitoperatorMaybe<T>([AllowNull] T value)
    {
        return new Maybe<T>(value);
    }
}
```
As you can see, we used the AllowNull attribute in the input, which allows us to use inputs that have a null type.

If we want to rewrite the code above, we can do it like this.

```csharp
Maybe<Customer> customer = _repository.GetById(id);
```
From now on, we can understand by looking at this code that this method may return null
This goes back to the principle of functional honesty in functional programming

In completing this article, you can see different implementations of this monad
You can probably find one of the best implementations here
[Language.Ext Github](https://github.com/louthy/language-ext#null-reference-problem)

