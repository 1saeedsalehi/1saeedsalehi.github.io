---
layout: post
title:  مساله null reference exception و راه حل فانکشنال طور!
categories:
  - lifestyle
tags:
  - functional-programming 
  - برنامه نویسی تابعی
  - تابع
  - exception
  - c#
  - validation

last_modified_at: 2021-11-08T12:00:00-05:00
---
### مساله null reference exception و راه حل فانکشنال طور!

تکه کد زیر را ببینید

```
Customer customer = _repository.GetById(id);
Console.WriteLine(customer.Name);
```

این کد برای شما آشناست؟ چه مشکلی رو میبینید؟

مشکل اینجاست که ما مطمئن نیستیم که متد getById میتونه null برگردونه یا نه؟
اگه احتمال داشته باشه که این کد null برگردونه ، زمانی که کد رو اجرا میکنیم خطای NullReferenceException دریافت خواهیم کرد.

فاجعه جایی اتفاق میوفته که تا زمان تست نرم افزار با دیتای واقعی / محیط مشتری / پروداکشن احتمالا متوجه این مساله نخواهیم شد و  به راحتی نمیتونیم بفهمیم که این کجا تبدیل به null شده!

احتمالا هر چقد سریع تر بفهمیم ، زمان کمتری میگیره حل کردنش.
احتمالا بهترین راه حل اینه که کامپایلر بهمون خطا بده!

```
Customer! customer = _repository.GetById(id);
Console.WriteLine(customer.Name);
```

 معنی `Customer!` اینه که این یک  نوغ غیر نال هست.
 یعنی به هیچ شکلی امکان تبدیل این آبجکت به نال وجود نداره.

 میتونید دنیایی رو تصور کنید که خطای null reference exception وجود نداشته باشه؟ من نمیتونم!

این امکان در سی شارپ از نسخه 8 به بعد فراهم شده
برای این که ببینید چه جوری میتونید از این فیچر استفاده کنید [اینجا](https://docs.microsoft.com/en-us/dotnet/csharp/nullable-references) رو ببینید

اما از اونجایی که ممکنه شما به هر دلیلی امکان ارتقا به نسخه سی شارپ بالاتر تو پروژه نداشته باشید
یا نخواین که این فیچر رو تو کل پروژه فعال کنید ما یه راه حل جانبی رو با هم اینجا بررسی میکنیم

البته میتونید از راهکاری که اینجا ازائه میدیم در کنار nullable reference type های c# استفاده کنید و ترکیب قوی تری رو درست کنید!

### چه جوری مقدار نال داشته باشیم؟

برای این کار ما یک موناد (monad) نیاز داریم 
تو این مثال اسمشو میزاریم MayBe

کد اون تو ساده ترین حالت میتونه به این شکل باشه

```
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

همونجور که میبینید ما توی ورودی از اتربیوت AllowNull استفاده کردیم که اجازه میده از ورودی هایی که جنس اونا null هستن بتونیم استفاده کنیم

اگه بخوایم تکه کد بالا رو بازنویسی کنیم احتمالا به شکل زیر میشه

```
Maybe<Customer> customer = _repository.GetById(id);
```

از الان به بعد میتونیم با نگاه کردن به این کد میتونیم بفهمیم که ممکنه این متد null برگردونه
این بر میگرده به اصل fucntion honesty توی functional progamming

در تکمیل این مطلب میتونید پیاده سازی های مختلفی که از این monad هست رو ببینید
احتمالا یکی از بهترین پیاده سازی ها رو میتونید اینجا پیدا کنید

[Language.Ext Github](https://github.com/louthy/language-ext#null-reference-problem)

