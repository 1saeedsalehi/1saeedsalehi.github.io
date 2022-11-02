---
layout: post
title: command line for unhide files

categories:
  - Programming
tags:
  - virus
  - ویروس

---
a simple command line script to hide / unhide files 

```
@ECHO 1saeed.dev
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