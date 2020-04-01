---
layout: post
title:   Functional Programming - Primitive Obsession
categories:
  - Programming
tags:
  - functional-programming 
  - برنامه نویسی تابعی
  - تابع
  - exception
  - استثنا
  - خطا

last_modified_at: 2020-04-01T18:00:00-05:00
---
### وسواس استفاده از نوع های اولیه
در ادامه سری مقالات مرتبط با برنامه نویسی تابعی ، قصد دارم به استفاده کردن یا نکردن از نوع های داده اولیه (Primitive Types) را بررسی کنیم.
پیشنهاد میکنم در صورتی که قسمت های قبلی را مطالعه نکرده اید ابتدا قسمت های قبل را بخوانید:

- [قسمت اول : آشنایی با مفاهیم](http://1saeedsalehi.ir/programming/2019/11/25/functional-programming.html)
- [قسمت دوم : مثال های عملی](http://1saeedsalehi.ir/programming/2019/11/30/functional-programming-2-examples.html)
- [قسمت سوم: Immutability عدم توانایی تغییر](http://1saeedsalehi.ir/programming/2020/02/15/functional-programming-3-refactoring-to-immutable.html)
- [قسمت چهارم :  برخورد با Exception ها](http://1saeedsalehi.ir/programming/2020/03/18/functional-programming-4-stay-away-from-exception.html)



### Exception و خوانایی کد
در طراحی مدل دامین ، بیشتر مواقع از نوع های اولیه مانند int , string,… استفاده میکنیم و به عبارتی میتوانیم بگوییم در استفاده از این نوع داده [وسواس](http://wiki.c2.com/?PrimitiveObsession) داریم. 

قطعه کد زیر را در نظر بگیرید:

```
public class UserFactory
{
    public User CreateUser(string email) { 
        return new User(email);
    }
}
```

کلاس UserFactory یک متد به نام CreateUser دارد که یک رشته را به عنوان ورودی میگیرد و یک شی از کلاس User را بر میگرداند. خوب مشکل این متد کجاست؟

اگر به خاطر داشته باشید در قسمت های قبلی در مورد مفهومی به نام Honesty صحبت کردیم. به طور ساده باید بتوانیم از روی امضای تابع ، کاری که تابع انجام میدهد و خروجی آن را ببینیم.

این تابع Honest نیست ، شرایطی که string می تواند درست نباشد ، خالی باشد ، طول غیر مجاز داشته باشد و .. را نمیتوانیم از امضای تابع حدس بزنیم.

![dishonest method signature](/assets/images/functional-programming/primitive-obsession/dishonest-method-signature.png)

برای روشن تر شدن بحث ، مثال بالا رو همیشه در ذهن خود داشته باشید. در مثال تابع Divide که عمل تقسیم را انجام می دهد ، پارامتر y که یک عدد از نوع int است ، میتواند مقدار صفر داشته باشد و باعث یک exception  شود. و از آنجایی که نوع خروجی این متد هم int است انتظار دریافت یک exception را نداریم. در مورد exception ها به طول مفصل در قسمت قبلی صحبت کردیم.    

در مثال بالا تصور کنید ، که به جای یک ایمیل از چند ایمیل به عنوان ورودی میخاهید استفاده کنید ، آیا منطق Validation را به ازای هر پارامتر ورودی باید تکرار کنید؟ 


![dishonest method signature](/assets/images/functional-programming/primitive-obsession/dishonest-method-signature.png)
به طور کلی استفاده نا به جا و بیش از جد از نوع های داده اولیه باعث میشود Honesty متد ها را از دست بدهیم و قاعده DRY را نقض کنیم.


صحبت در مورد استفاده کردن یا نکردن جنبه های زیادی دارد و یکی از مواردی هست که در معماری DDD تحت عنوان Value Object به آن پرداخته شده ، هدف ما در این قسمت از مقاله صرفا پرداختن به گوشه ای از این مورد هست ، ولی شما میتوانید برای مطالعه بیشتر و اطلاعات تکمیلی کتاب Domain-Driven Design: Tackling Complexity in the Heart of Software نوشته Eric Evans را مطالعه کنید.

### به جای نوع های اولیه از چی استفاده کنیم؟

جواب خیلی ساده اس ، شما نیاز دارید که یک Type اختصاصی درست کنید. به طور مثال به جای استفاده از نوع string برای ایمیل  ،  می توانید یک کلاس به عنوان Email ایجاد کنید که مشخصه ای به نام Value دارد.


این کار به روش های مختلفی قابل انجام است ، اما پیشنهاد من استفاده از این روش هست:

<script src="https://gist.github.com/1saeedsalehi/e2b454a3be06fb81a5e9f2782f316991.js"></script>

در این روش ، یک کلاس به عنوان Value Object  ایجاد کرده ایم ، این کلاس نوع اولیه ای که با آن سر و کار داریم را در بر خواهد گرفت. و منطق مربوط به مقایسه ، همچنین عملگر های `==` و `!=` را هم از طریق `.Equals`  و `.GetHashCode()` پیاده سازی کرده .



برای طور مثال برای کلاس ایمیل می توانیم به صورت زیر عمل کنیم: 

```
public class EmailAddress : ValueOf<string, EmailAddress> { }
```

همچنین برای مقدار دهی این کلاس میتوانید به صورت زیر عمل کنید

```
EmailAddress emailAddress = EmailAddress.From("foo@bar.com");
```

برای مثال های پیچیده تر ، مانند آدرس که شامل ادرس و کد پستی و ... می باشد میتوانید با استفاده از امکان Tuple ها که از سی شارپ 7 به بعد معرفی شده مانند مثال زیر عمل کنید

```
public class Address : ValueOf<(string firstLine, string secondLine, Postcode postcode), Address> { }
```

و در نهایت برای نوشتن منطق مربوط به validation می توانید متد Validate را Override کنید
و قاعده DRY را هم نقض نکنید.


روش معرفی شده در این مقاله صرفا جهت آشنایی بیشتر شما و داشتن کد تمیز تر از طریق مفاهیم برنامه نویسی تابعی خواهد بود. در دنیای واقعی ، احتمالا مسائلی برای ذخیره سازی این آبجکت ها و یا کار با لایبرری هایی مانند Entity Framework خواهید داشت که به سادگی قابل حل است.

در صورتی که مشکلی در پیاده سازی داشتید ، می توانید مشکل خود را زیر همین پست  و یا بر روی [gist](https://gist.github.com/1saeedsalehi/e2b454a3be06fb81a5e9f2782f316991) کامنت کنید.