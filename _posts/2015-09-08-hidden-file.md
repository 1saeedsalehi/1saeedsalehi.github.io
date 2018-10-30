---
layout: post
title: فلش ویروس و فایل های مخفی

categories:
  - Programming
tags:
  - virus
  - ویروس

---
این دیگه برای اکثر ماها پیش اومده که فلشمون ویروسی بشه و همه فایل های روش مخفی بشن
 راه ساده و جمع و جور برای این که فایل های مخفی رو از این حالت در بیاریم

```
@ECHO 1saeedsalehi.ir
@ECHO esme drive ro inja bezan: 
set /p letter= @ECHO %letter%: selected 
taskkill /im explorer.exe /f 
@ECHO. 
@ECHO "please wait..." 
@ECHO. attrib -s -h -a /s /d %letter%:\*.* 
@ECHO "completed." 

start explorer %letter%: 
taskkill /im cmd.exe /f


```