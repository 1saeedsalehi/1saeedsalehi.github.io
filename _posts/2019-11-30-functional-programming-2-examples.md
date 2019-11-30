---
layout: post
title:   Functional Programming - قسمت دوم
categories:
  - Programming
tags:
  - functional-programming 
  - برنامه نویسی تابعی
  - تابع
  - immutable
  - thread-safe
  - collection

last_modified_at: 2019-11-30T00:00:00-05:00
---
در [قسمت قبلی](http://localhost:4000/programming/2019/11/25/functional-programming.html) این مقاله، با مفاهیم تئوری برنامه نویسی تابعی آشنا شدیم. در این مطلب قصد دارم بیشتر وارد کد نویسی شویم و الگوها و ایده‌های پیاده سازی برنامه نویسی تابعی را در #C مورد بررسی قرار دهیم.

### Immutable Types
هنگام ایجاد یک Type جدید باید سعی کنیم دیتای داخلی Type را تا حد ممکن Immutable کنیم. حتی اگر نیاز داریم یک شیء را برگردانیم، بهتر است که یک instance جدید را برگردانیم، نه اینکه همان شیء موجود را تغییر دهیم. نتیحه این کار نهایتا به شفافیت بیشتر و Thread-Safe بودن منجر خواهد شد.

*مثال:*

```
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
r.Grow(10, 10);// r.Length is 15, r.Height is 20, same instance of r
```

در این مثال، Property های کلاس، از بیرون قابل Set شدن می‌باشند و کسی که این کلاس را فراخوانی میکند، هیچ ایده‌ای را درباره‌ی مقادیر قابل قبول آن‌ها ندارد. بعد از تغییر بهتر است وظیفه‌ی ایجاد آبجکت خروجی به عهده تابع باشد، تا از شرایط ناخواسته جلوگیری شود:

```
// After
public class ImmutableRectangle
{
    int Length { get; }
    int Height { get; }

    public ImmutableRectangle(int length, int height)
    {
        Length = length;
        Height = height;
    }

    public ImmutableRectangle Grow(int length, int height) =>
          new ImmutableRectangle(Length + length, Height + height);
}

ImmutableRectangle r = new ImmutableRectangle(5, 10);
r = r.Grow(10, 10);// r.Length is 15, r.Height is 20, is a new instance of r
```

با این تغییر در ساختار کد، کسی که یک شیء از کلاس ImmutableRectangle را ایجاد میکند، باید مقادیر را وارد کند و مقادیر Property ها به صورت فقط خواندنی از بیرون کلاس در دسترس هستند. همچنین در متد Grow، یک شیء جدید از کلاس برگردانده می‌شود که هیچ ارتباطی با کلاس فعلی ندارد.

### استفاده از Expression بجای Statement

یکی از موارد با اهمیت در سبک کد نویسی تابعی را در مثال زیر ببینید:

```
public static void Main()
{
    Console.WriteLine(GetSalutation(DateTime.Now.Hour));
}

// imparitive, mutates state to produce a result
/*public static string GetSalutation(int hour)
{
    string salutation; // placeholder value

    if (hour < 12)
        salutation = "Good Morning";
    else
        salutation = "Good Afternoon";

    return salutation; // return mutated variable
}*/

public static string GetSalutation(int hour) => hour < 12 ? "Good Morning" : "Good Afternoon";
```

به خط‌های کامنت شده دقت کنید؛ می‌بینیم که یک متغیر، تعریف شده که نگه دارنده‌ای برای خروجی خواهد بود. در واقع به اصطلاح آن را mutate می‌کند؛ در صورتیکه نیازی به آن نیست. ما می‌توانیم این کد را به صورت یک عبارت (Expression) در آوریم که خوانایی بیشتری دارد و کوتاه‌تر است.

### استفاده از High-Order Function ها برای ایجاد کارایی بیشتر

در قسمت قبلی درباره توابع HOF صحبت کردیم. به طور خلاصه توابعی که یک تابع را به عنوان ورودی میگیرند و یک تابع را به عنوان خروجی برمی‌گردانند. به مثال زیر توجه کنید:

```
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
این قطعه کد، مربوط به متد Count کتابخانه‌ی Linq می‌باشد. در واقع این متد تعدادی از چیز‌ها را تحت شرایط خاصی می‌شمارد. ما دو راهکار داریم، برای هر شرایط خاص، پیاده سازی نحوه‌ی شمردن را انجام دهیم و یا یک تابع بنویسیم که شرط شمردن را به عنوان ورودی دریافت کند و تعدادی را برگرداند.


### ترکیب توابع

ترکیب توابع به عمل پیوند دادن چند تابع ساده، برای ایجاد توابعی پیچیده گفته می‌شود. دقیقا مانند عملی که در ریاضیات انجام می‌شود. خروجی هر تابع به عنوان ورودی تابع بعدی مورد استفاده قرار میگیرد و در آخر ما خروجی آخرین فراخوانی را به عنوان نتیجه دریافت میکنیم. ما میتوانیم در #C به روش برنامه نویسی تابعی، توابع را با یکدیگر ترکیب کنیم. به مثال زیر توجه کنید:

```
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

در مثال بالا ما سه تابع جدا داریم که میخواهیم نتیجه‌ی آن‌ها را به صورت پشت سر هم داشته باشیم. ما میتوانستیم هر کدام از این توابع را به صورت تو در تو بنویسیم؛ ولی خوانایی آن به شدت کاهش خواهد یافت. بنابراین ما از یک Extension Method استفاده کردیم.

### Chaining / Pipe-Lining و اکستنشن‌ها

یکی از روش‌های مهم در سبک برنامه نویسی تابعی، فراخوانی متد‌ها به صورت زنجیره‌ای و پاس دادن خروجی یک متد به متد بعدی، به عنوان ورودی است. به عنوان مثال کلاس String Builder یک مثال خوب از این نوع پیاده سازی است. کلاس StringBuilder از پترن Fluent Builder استفاده می‌کند. ما می‌توانیم با اکستنشن متد هم به همین نتیجه برسیم. نکته مهم در مورد کلاس StringBuilder این است که این کلاس، شیء string را mutate نمیکند؛ به این معنا که هر متد، تغییری در object ورودی نمی‌دهد و یک خروجی جدید را بر می‌گرداند.

```
string str = new StringBuilder()
  .Append("Hello ")
  .Append("World ")
  .ToString()
  .TrimEnd()
  .ToUpper();
```

در این مثال  ما کلاس StringBuilder را توسط یک اکستنشن متد توسعه داده‌ایم:


```
public static class Extensions
{
    public static StringBuilder AppendWhen(this StringBuilder sb, string value, bool predicate) => predicate ? sb.Append(value) : sb;
}

public class Program
{
    public static void Main(string[] args)
    {
        // Extends the StringBuilder class to accept a predicate
        string htmlButton = new StringBuilder().Append("<button").AppendWhen(" disabled", false).Append(">Click me</button>").ToString();
    }
}
```

### نوع‌های اضافی درست نکنید ، به جای آن از کلمه‌ی کلیدی yield استفاده کنید!

گاهی ما نیاز داریم لیستی از آیتم‌ها را به عنوان خروجی یک متد برگردانیم. اولین انتخاب معمولا ایجاد یک شیء از جنس List یا به طور کلی‌تر Collection و سپس استفاده از آن به عنوان نوع خروجی است:

```
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

همانطور که مشاهده میکنید در مثال اول، ما از یک لیست موقت استفاده کرد‌ه‌ایم تا آیتم‌ها را نگه دارد. اما میتوانیم از این مورد با استفاده از کلمه کلیدی yield اجتناب کنیم. این الگوی iterate بر روی آبجکت‌ها در برنامه نویسی تابعی، خیلی به چشم میخورد.

### برنامه نویسی declarative به جای imperative با استفاده از Linq

در قسمت قبلی به طور کلی درباره برنامه نویسی Imperative صحبت کردیم. در مثال زیر یک نمونه از تبدیل یک متد که با استایل Imperative نوشته شده به declarative را می‌بینید. شما میتوانید ببینید که چقدر کوتاه‌تر و خواناتر شده:

```
List<int> collection = new List<int> { 1, 2, 3, 4, 5 };

// Imparative style of programming is verbose
List<int> results = new List<int>();

foreach(var num in collection)
{
  if (num % 2 != 0) results.Add(num);
}

// Declarative is terse and beautiful
var results = collection.Where(num => num % 2 != 0);
```

### Immutable Collection


در مورد اهمیت immutable قبلا صحبت کردیم؛ Immutable Collection ها، کالکشن‌هایی هستند که به جز زمانیکه ایجاد می‌شنود، اعضای آن‌ها نمی‌توانند تغییر کنند. زمانیکه یک آیتم به آن اضافه یا کم شود، یک لیست جدید، برگردانده خواهد شد. شما می‌توانید انواع این کالکشن‌ها را در [این لینک](https://docs.microsoft.com/en-us/dotnet/api/system.collections.immutable?view=netcore-2.2) ببینید.  
به نظر میرسد که ایجاد یک کالکشن جدید میتواند سربار اضافی بر روی استفاده از حافظه داشته باشد، اما همیشه الزاما به این صورت نیست. به طور مثال اگر شما f(x)=y را داشته باشید، مقادیر x و y به احتمال زیاد یکسان هستند. در این صورت متغیر x و y، حافظه را به صورت مشترک استفاده می‌کنند. به این دلیل که هیچ کدام از آن‌ها Mutable نیستند. اگر به دنبال جزییات بیشتری هستید [این مقاله](https://msdn.microsoft.com/en-us/magazine/mt795189.aspx) به صورت خیلی جزیی‌تر در مورد نحوه پیاده سازی این نوع کالکشن‌ها صحبت میکند. اریک لپرت یک سری مقاله در مورد Immutable ها در #C دارد که میتوانید آن هار [در اینجا](https://stackoverflow.com/a/927219/1650277) پیدا کنید.  



  
### Thread-Safe Collections 
  

اگر ما در حال نوشتن یک برنامه‌ی Concurrent / async باشیم، یکی از مشکلاتی که ممکن است گریبانگیر ما شود، race condition است. این حالت زمانی اتفاق می‌افتد که دو ترد به صورت همزمان تلاش میکنند از یک resource استفاده کنند و یا آن را تغییر دهند. برای حل این مشکل میتوانیم آبجکت‌هایی را که با آن‌ها سر و کار داریم، به صورت immutable تعریف کنیم. از دات نت فریمورک نسخه 4 به بعد  [Concurrent Collection](https://docs.microsoft.com/en-us/dotnet/standard/collections/thread-safe/)‌ها معرفی شدند. برخی از نوع‌های کاربردی آن‌ها را در لیست پایین می‌بینیم:
  
| نوع | توضیح |
|---|---|
| [ConcurrentDictionary](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentdictionary-2) | پیاده سازی thread safe از دیکشنری key-value |
|  [ConcurrentQueue](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentqueue-1)    |   پیاده سازی thread safe از صف (اولین ورودی ، اولین خروجی) |
|  [ConcurrentStack](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentstack-1)  |   پیاده سازی thread safe از پشته (آخرین ورودی ، اولین خروجی) |
|  [ConcurrentBag](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentbag-1)    |   پیاده سازی thread safe از لیست نامرتب |

  
  
این کلاس‌ها در واقع همه مشکلات ما را حل نخواهند کرد؛ اما بهتر است که در ذهن خود داشته باشیم که بتوانیم به موقع و در جای درست از آن‌ها استفاده کنیم.  
  
در این قسمت از مقاله سعی شد با روش‌های خیلی ساده، با مفاهیم اولیه برنامه نویسی تابعی درگیر شویم. در ادامه مثال‌های بیشتری از الگوهایی که میتوانند به ما کمک کنند، خواهیم داشت.