---
layout: post
title:  Getting started with Dapr 
categories:
  - Software-Engineering
tags:
  - dapr
  - microservices
  - miniservices
  - sidecar
  - software-architecture
  - software
  - developer
  - مایکروسرویس
  - میکروسرویس
  -

last_modified_at: 2021-05-05T18:00:00-05:00
---

اگه شما هم درگیر توسعه اپلیکیشن ها با معماری مایکروسرویس باشید این روز ها خیلی اسم Dapr رو بشنوید

اگه بخوایم خیلی ساده به این قضیه نگاه کنیم Dapr سعی کرده توسعه اپلیکیشن های مایکروسرویسی رو خیلی راحت تر کنه

و این کار رو با تمرکز روی منطق اصلی برنامه و ساده نگه داشتن کد و دور کردن کد از مسائل زیر ساختی هست
در واقع Dapr همونجور که خودش رو معرفی میکنه یه runtime هست برای اجرای برنامه ها



اولین ریلیز این محصول هیجان انگیز 17 فوریه 2021 بوده و اماده استفاده توی پروداکشن هست 

[معرفی](https://blog.dapr.io/posts/2021/02/17/announcing-dapr-v1.0/)

### پیشنیاز ها

 - [نصب Cli Dapr](https://docs.dapr.io/getting-started/install-dapr-cli/)
 
 - [نصب .Net 5 SDK](https://dotnet.microsoft.com/download/dotnet/5.0)
 
### شروع

برای یک پروژه ساده Asp.net Core ایجاد میکنیم
```
mkdir WeatherForecastService
cd WeatherForecastService
dotnet new webapi
dotnet run
``` 

همون جوری که خودتون بهتر میدونید این یه پروژه ساده asp .net core درست میکنه

معمولا به صورت پیشفرض توی تملیت این پروژه یه کنترلر پیشفرضی وجود داره که دیتای آب و هوا رو مثلا قراره برگردونه



### اجرای Api با استفاده از Dapr

Dapr از الگوی [sidecar](https://docs.microsoft.com/en-us/azure/architecture/patterns/sidecar) داره استفاده میکنه

dapr میتونه api ما رو اجرا کنه و Api ما رو به از طریق dapr به بیرون expose کنه

دستور زیر رو روی مسیر جاری پروژه تون اجرا کنید

```
dapr run --app-id weatherforecastservice --dapr-http-port 3500 --app-port 5001 --app-ssl -- dotnet run
```

بیاین بخش های مختلف این دستور رو با هم دیگه مرور کنیم

 - app-id یه شناسه برای اپلیکیشن/سرویس تون که بشه با این شناسه پیداش کرد
 - dapr-http-port پورتی که sidecar dapr به اون گوش میکنه
 - app-port پورتی که اپلیکیشن/سرویس ما روی اون گوش میکنه
 - app-ssl مشخص میکنه که درخواست هایی که سرویس میده ssl باشه یا نه
 - dotnet run روشی که باهاش پروژه رو اجرا میکنید
	
### مزایای استفاده از Dapr

قبل از این که پروژه رو روی Dapr اجرا کنید میتونید با ادرس
[https://localhost:5001/weatherforecast](https://localhost:5001/weatherforecast)
اما وقتی با استفاده از dapr پروژه رو اجرا میکنید علاوه بر روش بالا

میتونید متد مورد نظرتون رو با استفاده از فراخوانی اون از طریق sidecar اجرا کنید
[http://localhost:3500/v1.0/invoke/weatherforecastservice/method/weatherforecast](http://localhost:3500/v1.0/invoke/weatherforecastservice/method/weatherforecast)

![dapr](/assets/images/dapr/getting-started.jpg)

شاید تو نگاه اول این خیلی ساده به نظر برسه

اما نکته خیلی مهم درباره این روش اینه اهمیتی نداره سرویسی که پشت dapr داره اجرا میشه روی چه بستری هست
به طور مثال اگه روی gRPC هم باشه ما میتونیم اونو خیلی راحت با استفاده از Api هایی که Dapr به ما میده میتونیم فراخوانی کنیم

در واقع یه روش استانداردی برای فراخوانی متد ها بین سرویس های مختلف میشه داشته باشیم که به صورت زیره

```
http://localhost:<dapr-http-port>/v1.0/invoke/<app-id>/method/<method-name>
```

نکته مهم اینه که این روش فارغ از زبان و تکنولوژی هست و شما به همین روش میتونید به صورت مستقیم با استفاده از این Api یک سرویس که با زبان دیگه ای توسعه داده شده رو فراخوانی کنید 
ولی این همه ماجرا نیست

با استفاده از این روش استاندارد برای فراخوانی سرویس ها ما میتونیم به چیز های بزرگتری توی توسعه اپلیکیشن های مایکروسرویسی برسیم
خیلی از این ها به صورت توکار توی Dapr پیاده شدن 

 - Service discovery
 - Distributed tracing
 - Metrics
 - Error handling
 - Encryption 
 
 و نکته خیلی جالب درباره Dapr اینه که شما نیاز نیست وقتی دارین کد مینویسین درباره همه این مسائل فک کنید
 و به جاش میتونید تمرکزتون رو بزارین رو معماری درست و پیاده سازی درست و با کیفیت منطق برنامه
 
 لایبرری هایی برای راحت تر کردن استفاده از امکانات dapr وجود داره که با یه جستجوی ساده توی nuget میتونید اونا رو پیدا کنید
 
### خلاصه
 
 توی این متن به صورت خیلی خلاصه با Dapr آشنا شدیم میتونید روی لینک [dapr.io](https://dapr.io) اطلاعات بیشتری پیدا کنید
 
 احتمالا در آینده چیز های بیشتری از این محصول جذاب مینویسم
 
 ممنون میشم شما هم اگه نظری داشتین با من به اشتراک بزارید
 