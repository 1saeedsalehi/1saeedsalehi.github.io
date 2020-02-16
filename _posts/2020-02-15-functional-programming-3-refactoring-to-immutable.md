---
layout: post
title:   Functional Programming - قسمت سوم
categories:
  - Programming
tags:
  - functional-programming 
  - برنامه نویسی تابعی
  - تابع
  - immutable
  - thread-safe
  - collection

last_modified_at: 2020-02-16T12:10:00-05:00
---

در ادامه پست های مربوط به برنامه نویسی تابعی ، قصد دارم بیشتر وارد کد شویم و مباحث عنوان شده را در دنیای کد پیاده سازی کنیم. هدف این قسمت refactor کردن کد موجود به یک معماری immutable هست
پیشتر درباره immutable ها صحبت کردیم.
ابتدا برای یکسان سازی ادبیات مورد استفاده چند کلمه را مجددا تعریف خواهیم کرد.

Immutability :  عدم توانایی تغییر داده

State :  داده هایی که در طول زمان تغییر میکنند

Side Effect :  تغییری که روی داده ها اتفاق می افتد


در قطعه کد زیر سعی شده تفاوت یک کلاس Stateless و stateful را به سادگی نشان دهم.


```
//Stateful
public class UserProfile
{
    private User _user;
    private string _address;

    public void UpdateUser(int userId, string name)
    {
        _user = new User(userId, name);
    }
}

//Stateless
public class User
{
    public User(int id, string name)
    {
        Id = id;
        Name = name;
    }

    public int Id { get; }
    public string Name { get; }
}
```

### چرا Immutable بودن مهم است؟

هر عمل mutable معادل کد غیر شفاف است. در واقع وابستگی هر عملی که انجام می دهیم به state باعث میشود که شرایط ناپایدار در کد داشته باشیم. به طور مثال در یک عملیات  چند نخی تصور کنید که چندین نخ به طور همزمان می توانند state رو تغییر دهند و مدیریت این قضیه باعث به وجود آمدن کد های ناخوانا و تحمیل پیچیدگی بیشتر به کد خواهد شد.

![method honesty](/assets/images/method-honesty.png)

در واقع انتظار داریم که به ازای یک ورودی بر اساس بدنه متد یک خروجی داشته باشیم ، ولی در واقعیت تاثیری که اجرای متد بر روی state کل کلاس خواهد گذاشت از دید ما پنهان است و باعث به وجود آمدن مشکلات بعدی خواهد شد.


برای مثال قطعه کد بالا را به صورت Honest بازنویسی میکنیم

```
public class UserProfile
    {
    private readonly User _user;
    private readonly string _address;

    public UserProfile(User user,string address)
    {
        _user = user;
        _address = address;
    }
    public UserProfile UpdateUser(int userId, string name)
    {
        var newUser = new User(userId, name);
        return  new UserProfile(newUser,_address);
    }
}

public class User
{
    public User(int id, string name)
    {
        Id = id;
        Name = name;
    }

    public int Id { get; }
    public string Name { get; }
}

```

در این مثال  متد UpdateUser به جای Void یک شی از جنس کلاس UserProfile را بر میگرداند . کلاس UserProfile هم برای وهله سازی نیاز به یک شی از جنس User و Address دارد بنابراین مطمئن هستیم که مقدار دهی شده اند.
نکته دیگر در قطعه کد بالا این است که به ازای هر بار فراخوانی متد یک شی جدید بدون وابستگی به وهله سازی اشیا دیگر برگردانده میشود.


Immutable بودن باعث می شود:

1.	خوانایی کد افزایش پیدا کند
2.	یک جای واحد برای Validate کردن داشته باشیم
3.	به صورت ذاتی Thread Safe باشیم

محدودیت هایی که در کار با اشیا Immutable باید در نظر داشته باشیم میتوان به مصرف بالای رم و سی پی یو بیشتر اشاره کرد
در واقع به نسبت حالت mutate تعداد آبجکت های بیشتری ساخته خواهند شد.
در فریمورک دات نت برای کار با اشیا immutable امکاناتی در نظر گرفته شده که این هزینه را کاهش دهد
به طور مثال ما می توانیم از کلاس ImmutableList استفاده کنیم و از ایجاد اشیا اضافه تر و تحمیل بار اضافی به GC جلوگیری کنیم.


مثال

```
//Create Immutable List
ImmutableList<string> list = ImmutableList.Create<string>();
ImmutableList<string> list2 = list.Add("Salam");


//Builder
ImmutableList<string>.Builder builder = ImmutableList.CreateBuilder<string>();
builder.Add("avali");
builder.Add("dovomi");
builder.Add("sevomi");

ImmutableList<string> immutableList = builder.ToImmutable();

```

### چطور با side effect کنار بیایم؟
یکی از پترن های رایج برای این کار مفهوم جدا سازی  Command/Query است. به طور ساده  تمامی عملیاتی که تاثیر گذار هستند را به صورت Command در نظر میگیریم. Command ها معمولا هیچ نوعی را بازگشت نمیدهند و همینطور بر عکس این قضیه برای Query ها صادق است.
اشتباه رایج درباره این الگو ، محدود کردن این الگو به معماری های خاص مانند Domain Driven می باشد در صورتی که الزامی برای رعایت این الگو در سایر معماری ها وجود ندارد.


![command query separation](/assets/images/command-query-separation.png)

به مثال زیر دقت کنید. سعی کردم قسمت های Command و Query را از هم جدا کنم


![command query code sample](/assets/images/command-query-code.png)

در واقع هر اپلیکیشن شامل دو قسمت می تواند باشد 


قسمتی که منطق بیزینسی برنامه پیاده سازی می شود که باید به صورت Immutable باشد و خروجی را تولید میکند و قسمت دیگر برنامه که خروجی تولید شده برای ذخیره سازی وضعیت سیستم استفاده می کند.

![application architecutre](/assets/images/)

در واقع یک هسته Immutable ورودی را دریافت میکند و خروجی های مورد نیاز را تولید میکند و همه این ها در دل یک پوسته Mutable پیاده سازی می شوند که ما در اینجا به آن اصطلاحا Mutable Shell میگوییم.

![mutable-shell-immutable-core](/assets/images/application.png)

برای مسائلی که در بالا صحبت شد نمونه آماده کردم. این نمونه  به طور ساده یک سیستم مدیریت نوبت است که نوبت ها را در فایل ذخیره / بازیابی میکند (mutate) و منطق مربوط به نوبت ها و زمان ویزیت آن میتواند به صورت immutable پیاده شود.
این کد در دو حالت functional و غیر functional پیاده سازی شده تا به خوبی تفاوت آن در حالت قبل و بعد از برنامه نویسی تابعی بتوانیم درک کنیم.
به جهت خوانایی بیشتر و دسترسی به کد ها ، آن را روی گیتهاب قرار داده شما میتوانید از [اینجا](https://github.com/1saeedsalehi/Immutability) سورس کد مورد نظر را بررسی کنید.
سعی شده در این مثال تمامی مواردی که در این قسمت ذکر شده را پیاده سازی کنیم

امیدوارم که مطالب مربوط به برنامه نویسی تابعی یا functional programming توانسته باشد دیدگاه جدیدی به کد هایی که مینویسیم بدهد.
در  قسمت های بعدی به مواردی مانند مدیریت exception ها و کار با null ها و ... خواهیم پرداخت. 