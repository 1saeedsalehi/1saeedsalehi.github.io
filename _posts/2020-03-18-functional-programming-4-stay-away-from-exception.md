---
layout: post
title:   Functional Programming - part 4 
categories:
  - Programming
tags:
  - functional-programming 
  - exception

last_modified_at: 2020-03-18T20:45:00-05:00
---
### برخورد با Exception ها
چنان چه قسمت های قبلی سری آموزش برنامه نویسی تابعی Functional Programming را مطالعه نکردین پیشنهاد میکنم قبلا اون ها [+](http://1saeedsalehi.ir/programming/2020/02/15/functional-programming-3-refactoring-to-immutable.html) و [+](http://1saeedsalehi.ir/programming/2019/11/30/functional-programming-2-examples.html) و [+](http://1saeedsalehi.ir/programming/2019/11/25/functional-programming.html) رو قبل از شروع بخونید
تو این قسمت قراره که تاثیر استثناها (exception) رو بر روی کد بررسی کنیم و راهکاری از جنس functional براش ارائه کنیم.


### Exception و خوانایی کد
تکه کد زیر را در نظر بگیرید:
یک Action معمولی در Asp.Net MVC که یک نام را دریافت میکند  و یک کارمند ایجاد میکند.

```
public ActionResult CreateEmployee(string name) { 
    try { 
        ValidateName(name);
        // ادامه کد ها
        return View("با موفقیت ثبت شد");
        }
    catch (ValidationException ex) 
    { 
        return View("خطا", ex.Message);
    }
}

private void ValidateName(string name) { 
    if (string.IsNullOrWhiteSpace(name)) 
        throw new ValidationException("نام نمی تواند خالی باشد");

    if (name.Length > 100) 
        throw new ValidationException("نام نمی تواند طولانی باشد");
}

```

![Hidden Part of Method](/assets/images/functional-programming/exception/hidden-part.png)

در این قطع کد متد  ValidateName در صورت معتبر نبودن ورودی ، Exception رخ میدهد ، و بلاک کد try/catch این exception را دریافت کرده و خطای مناسب را به کاربر نشان خواهد داد. تا اینجا ظاهرا همه چیز مرتب است و مشکلی ندارد! احتمالا کد های مشابه این کد را زیاد دیده اید.
در اینجا متد ValidateName ، صادق نیست، در قسمت اول در مورد Honesty صبحت کردیم. به عبارت ساده تر شما از امضای این متد نمی توانید به نوع خروجی و کاری که قرار است انجام دهد پی ببرید. 


در واقع شما همیشه باید پیاده سازی متد را گوشه ای در ذهن خود داشته باشید. و برای اطمینان از کاری که متد انجام میدهد همیشه باید به بدنه متد برگردیم.
اگر به خاطر داشته باشید توابع برنامه نویسی را به توابع ریاضی تشبیه کردیم. پس میتوانیم بگوییم:


![mathematical function](/assets/images/functional-programming/exception/mathematical-function.png)

به عبارت دیگر وقتی از exception ها برای کنترل flow برنامه استفاده میکنید ، مشابه کاری را انجام می دهید که  دستور GOTO انجام می داد. این دستور در روش های قبل از برنامه نویسی ساخت یافته وجود داشت و توسط یک دانشمند هلندی به نام آقای دیکسترا حذف شد. 
وقتی از دستور GOTO یا JUMP استفاده میکنیم ، فهمیدن flow برنامه پیچیدگی های زیادی خواهد داشت. چرا که فراخوانی قطعه های کد و متد ها وابستگی شدیدی خواهند داشت و البته میتوان گفت استفاده از exception ها برای کنترل ، می توانند از GOTO هم بد تر باشند ، چرا که exception میتواند از لایه های مختلف کد عبور کند.


امیدوارم تا اینجا به یک عقیده مشترک رسیده باشیم ، خوب راهکار چیست؟
تصور کنید که تکه کد بالا را به صورت زیر تبدیل کنیم:

```
public ActionResult CreateEmployee(string name) { 
    string error = ValidateName(name);

 	if (error != string.Empty) 
        return View("خطا", error);
    // ادامه کد ها 

    return View("با موفقیت ثبت شد");
}

private string ValidateName(string name) { 

    if (string.IsNullOrWhiteSpace(name)) 
        return "نام نمی تواند خالی باشد";

    if (name.Length > 100) 
        return "طول نام نمی تواند بیشتر از 100 کاراکتر باشد";

    return string.Empty;
}
```

با refactor ای که انجام دادیم ، متد ValidateName را به یک تابع ریاضی تبدیل کردیم ، به این معنی که هر آن چه که از امضای متد مشخص است را انجام می دهد و چیزی مخفی نیست.
توجه داشته باشید که این راهکار نهایی ما نیست ، و لطفا مقاله را تا انتها بخوانید!


### موارد استفاده Exception

با همه بدی هایی که از Exception ها گفتیم ، با این حساب پس کی ازش استفاده کنیم؟

1. Exception ها واقعا برای موارد استثنائی هستند.
2. Exception ها برای شرایطی هستند که به معنای واقعی یک باگ باشند.
3.  منتظر رخ دادن Exception  نباشیم!


![mathematical function](/assets/images/functional-programming/exception/validation.png)

در توضیح مورد سوم ، در اعتبار سنجی داده های کاربر  (Validation)انتظار داده نادرست را می توان داشت پس نمی توانیم آن را یک حالت استثنایی یکی بدانیم.
معماری زیر را در نظر بگیرید


![mathematical function](/assets/images/functional-programming/exception/architecutre-validation-exception.png)

دیتایی که به API ما ارسال خواهد شد همیشه شامل عملیات Filter یا به عبارتی Validation خواهد بود ، و از آن جایی که می توان انتظار استفاده نادرست ، یا دیتای نادرست را داشت نمیتوانیم این را حالتی از استثنائات در نظر بگیریم ، ولی بر خلاف آن وقتی در دامین پروژه و ارتباط بین دامین های مختلف دیتایی رد و بدل می شود که معتبر نیست ، میتوانیم آن را جزء استثناء ها در نظر بگیریم.
به مثال زیر دقت کنید

```
public ActionResult UpdateEmployee(int employeeId, string name) { 
    string error = ValidateName(name);
    
    if (error != string.Empty) 
        return View("Error", error);
    
    Employee employee = GetEmployee(employeeId); 
    employee.UpdateName(name);
}

public class Employee { 

    public void UpdateName(string name){

        if (name == null) 
            throw new ArgumentNullException();
        
        // ادامه کد ها
    }
}

```
در قطعه کد بالا تصور این است که کلاس Employee و  متد UpdateName خارج از دامین می باشند.
همانطور که مشاهده میکنید ما در action controller از خالی نبودن نام اطمینان حاصل کردیم و سپس آن را به متد UpdateName ارجاع دادیم . ولی اگه به بدنه متد UpdateName دقت کنید ، میبینید که مجددا از خالی نبودن نام اطمینان حاصل کرده ایم و **در صورت خالی بودن یک Exception پرت میکنیم!**
به این مدل چک کردن ها توی دامین های مختلف معمولا میگیم guard clause و میشه بگیم یه جور قرارداد بین برنامه نویس هاست. 

اگه طبق تعریفی که بالاتر اراه کردیم هم چک کنیم میتونیم حدس بزنیم که **خالی بودن نام نشان یک باگ در نرم افزار است!**


### مفهوم fail fast

تا اینجا متوجه شدیم که از exception ها باید در شرایط استثنائی استفاده کنیم ،  خوب با توجه به این مساله چه طور میتوانیم آن را Handle کنیم؟  این سوال ما را به مفهومی به نام fail fast می رساند.
این مفهوم به ما میگوید:

- کار جاری را به محض یک اتفاق استثنائی باید متوقف کنیم.
- رعایت این نکته در نهایت ما را به یک نرم افزار پایدار خواهد رساند.

برای درک هر چه بهتر این موضوع بیایید به برعکس این حالت نگاه کنیم ، اصطلاحا Fail Silently

متد زیر را ببینید:

```
public void ProcessItems(List<Item> items) { 

    foreach (Item item in items) { 
        try { 
            Process(item);
 		} 
        catch (Exception ex) 
        {	 
            Logger.Log(ex);
 		}
 	}
 }

```
در قطعه کد بالا در نگاه اول احتمالا حس نرم افزار پایدار تر و بدون خطا را خواهیم داشت

اما در واقع اینطور نیست ، احتمال این که خطا از چشم برنامه نویس به دور باشد و بعد از اجرا باعث شود که یکپارچگی داده ها را به هم بریزد وحود دارد.در واقع هیچ راهی برای زمانی که این عملیات نباید انجام شود در نظر گرفته نشده.

طبق صحتب هایی که بالا تر داشتیم ، یک شرایط غیر منتظره در واقع یک باگ در نرم افزار است و هیچ مزیتی در جلوگیری از وقوع این باگ بدون حل مشکل نیست!


به صور خلاصه مهم ترین مزیت Fail Fast را میتوانیم به صورت زیر خلاصه کنیم

-	مسیر رسیدن به خطا ها سر راست تر می شود
-	نرم افزار به پایداری مناسبی خواهد رسید
-	از اعتبار دیتای ذخیره شده اطمینان خواهیم داشت

### کجا exception ها را به دام بندازیم؟

در یکی از حالت های زیر

-	لاگ کردن
-	متوقف کردن عملیات
-	هیچ گاه در بلاک catch هیچ منطقی را پیاده نکنید.

حالت دیگر در استفاه از لایبرری های سوم شخص (3rd parties) است.به طور مثال در استفاده از EF ممکن است به دلیل عدم برقراری ارتباط با دیتابیس خطا دریافت کنید با توجه به دو نکته با این استثنائات برخورد کنید:

-	جلوی این نوع استثنائات را در پایین ترین حد ممکن در کد خود بگیرید.
-	Exception هایی را catch کنید که میدانید در حالت استثنا چه کاری انجام دهید

این به این معنی میباشد که به صورت کلی  همه نوع Exception  ها را به صورت کلی نگیرید و نوع Exception اختصاصی را در بلاک catch قرار دهید.

الان که قرار شد تو بعضی حالت ها ، جلوی استثنائات رو بگیریم خوبه که ببینیم چه جوری باید اینکار رو انجام بدیم

قطعه کد زیر رو در نظر بگیرید 

```

public void CreateCustomer(string name) { 
    Customer customer = new Customer(name); 
    bool result = SaveCustomer(customer);
    if (!result) { 
        MessageBox.Show("Error connecting to the database. Please try again later.");
    }
}

private bool SaveCustomer(Customer customer) { 
    try { 
        using (MyContext context = new MyContext()) { 
            context.Customers.Add(customer);
 	        context.SaveChanges();
        } 
        return true;
    }
    catch (DbUpdateException ex) { 
        if (ex.Message == "Unable to open the DB connection") 
            return false; 
        else 
            throw;
    }
}
```


همانطور که مشاهده میکنید ، در حالتی که خطا از نوع DbUpdateException رخ میدهد ، مقدار بازگشتی متد را برابر با false میکنیم.


 اما مشکلی که وجود دارد این است که این به اندازه کافی خوانا نیست ، همچنین honest بودن متد را نقض کرده ایم همچنین مشکل بزرگتر این است که ما با بازگرداندن یک مقدار bool میتوانیم به متد بالاتر اطلاع بدهیم که کار مورد نظر انجام شده یا نه ، اما در مورد دلیل انجام نشدن آن هیچ کاری نمیتوانیم بکنیم.

 پیشنهاد من برای مقدار بازگشتی متد های احتمال انجام نشدن کاری در آن ها می رود استفاده از یک نوع اختصاصی می باشد.

در اینجا من این نوع را با نام کلاس Result معرفی میکنم.
انتظاری که از این نوع اختصاصی داریم:

-	Honest بودن متد را نگه دارد
-	خروجی متد را به همراه وضعیت اجرا شدن برگرداند
-	یک شکل یکسان برای خطا ها داشته باشد
-	فقط جلوی خطا های غیر منتظره را بگیرد

به طور مثال کد بالا به شکل زیر refactor کنیم

```
private Result SaveCustomer(Customer customer) { 
    try { 

        using (var context = new MyContext()) { 

            context.Customers.Add(customer); 
            context.SaveChanges();
 		} 

        return Result.Ok();
    } 
    catch (DbUpdateException ex) { 
        if (ex.Message == "Unable to open the DB connection") 
            Result.Fail(ErrorType.DatabaseIsOffline);

        if (ex.Message.Contains("IX_Customer_Name")) 
            return Result.Fail(ErrorType.CustomerAlreadyExists);

        throw;
    }
}

```

به عبارتی با این روش میتوانیم  از انجام شدن / نشدن عملیات اطیمان حاصل کنیم و خروجی / دلیل انجام نشدن را میتوانیم برگردانیم  

اگر به امضای متد های زیر نگاه کنیم می توانیم آن ها را طبق CQS دسته بندی کنیم


![mathematical function](/assets/images/functional-programming/exception/cqs.png)

به عنوان نمونه یک پیاده سازی از این کلاس را در اینجا قرار داده ام.

<script src="https://gist.github.com/1saeedsalehi/af1d65e9668220f44e80c6c28b9e3019.js"></script>

قطعا میتوانیم پیاده سازی های بهتری از این کلاس داشته باشیم ، خوشحال میشم که نظراتتون رو باهام به اشتراک بذارین.

امیدوارم که این قسمت و صحبت هایی که در مورد استثنائات داشتیم تونسته باشه دیدگاه جدیدی به کد هاتون بده. در ادامه این سری مطالب ، مفاهیم پارادایم برنامه نویسی تابعی را بیشتر مورد بررسی قرار خواهیم داد.