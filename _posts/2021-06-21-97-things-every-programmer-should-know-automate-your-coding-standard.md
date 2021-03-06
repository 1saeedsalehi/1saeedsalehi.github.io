---
layout: post
title:   چیزهایی که هر برنامه نویسی باید بداند - چیز چهارم
categories:
  - lifestyle
tags:
  - 97-چیز
  - software-architecture
  - software
  - developer
  - برنامه نویسی
  - چیزهایی که هر برنامه نویسی باید بداند

last_modified_at: 2021-06-18T12:00:00-05:00
---
### استاندارد های کد نویسی خود را خودکار کنید

در شروع پروژه ، همه کلی ایده های خوب دارند، میتونیم بهشون بگیم *قول و قرار های پروژه جدید*
خیلی از این قول و قرار ها نوشته میشن ، معمولا در شروع همه اونو با هم مرور میکنن و سعی میکنن که رعایت کنناما به مرور زمان این قول و قرار های خوب یکی یکی نار گذاشته میشن و در نهایت وقتی پروژه تحویل داده میشه کد به هم ریخته به نظر میرسه و هیچکس نمیدونه چرا اینجوری شده.


چه زمانی این مشکل پیش اومده؟احتمالا در شروغ  داستان. بعضی از اعضای تیم بهش توجهی نکردن ، بعضی ها موضوع رو نفهمیدن 
بد از همه بعضی ها مخالف بودن اما نظری ندادن و حتی بعضی ها حرف رو فهمیدن ، موافقت کردن اما وقتی فشار پروژه زیاد شد مجبور شدن این قواعد رو زیر پا بزارن.

خوندن یک کد که استاندارد ها توش رعابت نشده واقعا کار خسته کننده ای میتونه باشه ، معمولا شما مجبور میشید که یک کلاس نامرتب رو با دست  indent گذاری کنید که بتونید بخونیدش!


اما اگر این واقعا یه مشکله ، پس چرا ما میخوایم که یه سری استاندارد کد نویسی داشته باشیم؟ یکی از دلایل فرمت کردن کد به شکل واحد اینه که کسی نتونه با فرمت انحصاری خودش مالک اون تکه کد بشه که کس دیگه ای نتونه اونو بفهمه.
ممکنه ما بخواهیم که جلوی دولوپر ها رو بگیریم که مانع یه سری anti-pattern ها بشیم
و در نهایت نزاریم یه سری باگ های معمول اتفاق بیوفته.
در کل یه استاندارد کد نویسی باید کار کردن تو یه پروژه و نگه داری کد ها از اول تا آخر پروژه رو آسون تر کنه.
میشه نتیجه گرفت که همه اعضای تیم باید از این استاندارد پیروی بکنن.

تعداد خیلی زیادی ابزار وجود دارد که میتواند به تهیه گزارشات کیفیت کد و مستند سازی و حفظ استاندارد های کد نویسی کمک کند ، این تمامی راه حل نیستو این ابزار ها تا حد ممکن باید به صورت خودکار اجرا شوند، در اینجا چند نمونه از این ابزار ها را معرفی میکنیم

  - مطمئن شوید که ابزار های فرمت کد قسمتی از پروسه بیلد هستن و هر کس هر بار که کد را کامپایل میکند به طور خودکار اجرا می شود.
 - از ابزار های اسکن کد استفاده کنید که anti-pattern های ناخواسته را شناسایی کنید. در صورتی که موردی پیدا شد بیلد نباید انجام شود.
 - یاد بگیرید که این ابزار ها را بسته به نیاز پروژه و بسته به الگو های پروژه خود تنظیم کنید
 - پوشش تست های کد را اندازه گیری کنید. نتایج تست ها را نیز بررسی  کنید و اگر پوشش تست خیلی کم است پروسه بیلد را متوقف کنید.


 سعی کنید این کار را برای همه موارد مهم پروژه انجام دهید. شما نمیتوانید همه آن چیزی را که واقعا برایتان اهمیت دارد را به صورت خودکار انجام بدین
 
درم ورد مواردی که نمیتوانید به صورت خودکار انجام بدهید مستندات تهیه کنید و دستور العمل های استاندارد برایشنان بنویسید ، اما باید بپذیرید که شما و همکارای شما ممکنه که اونا رو با دقت دنبال نکنن.

در نهایت ، استاندارد کد نویسی باید پویا باشد تا ایستا. با تکمیل شدن پروژه نیاز های پروژه تغییر میکند و اون چیزایی که اول پروژه  ممکنه هوشمندانه به نظر برسه ، لزوما چند ماه بعد هوشمندانه نیست.
