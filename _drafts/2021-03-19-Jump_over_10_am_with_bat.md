---
layout: post
title: 开源AR库ArUco的原理与位姿估计实战
categories:  AR
description: none
keywords: AR,  Marker, ARUco
---

近期在搭建SLAM-AR的过程中，需要用到Marker作为虚拟坐标系的参考，使用了开源库ArUco实现了该功能。

------

### 参考文献

[https://www.bilibili.com/video/av6458387/?p=2&spm_id_from=pageDriver](https://www.bilibili.com/video/av6458387/?p=2&spm_id_from=pageDriver)

### 上代码

```bash
@echo off
setlocal enabledelayedexpansion
mode con cols=30 lines=5 & color 0A
set /a hour=10
set /a min=0
set/a tt=30
echo 这个文件可以用记事本编辑
echo 10秒后继续
ping 10.252.252.252 -n 1 -w 10000 1>nul 2>nul
set /a min1=%min%-1
set /a min2=%min%+2

:timeloop
ping 10.252.252.252 -n 1 -w 800 1>nul 2>nul
set /a n+=1
cls
echo %time:~0,8%
echo ...
echo n=!n!

if %n% GEQ %tt% (goto timejump) else (goto timeloop)

:timejump
if %time:~0,2% EQU %hour% (
set /a tt=30
if %time:~3,2% EQU %min1% (
set /a tt=1)
if %time:~3,2% EQU %min% (
time 10:01
goto loop1) else(
cls
echo %time:~0,8%
echo ...
echo n=!n!
set /a n=0
goto timeloop)) else (
cls
set /a tt=600
echo %time:~0,8%
echo ...
echo n=!n!
set /a n=0
goto timeloop
)
pause

:loop1
ping 10.252.252.252 -n 1 -w 800 1>nul 2>nul
set /a m+=1
cls
set /a mm=60-%time:~6,2%
echo 时间%mm%后与网络同步
if %mm% LEQ 10 (goto update) else (
goto loop1)

:update
if %time:~3,2% EQU %min2% (
net time /setsntp:time-nw.nist.gov
net stop w32time & net start w32time
cls
echo 时间和网络同步完成
cls
echo 如果时间错误，没关系!
echo 30秒后系统会自动更正。
ping 10.252.252.252 -n 1 -w 30000 1>nul 2>nul
goto timeloop) else (
set /a m=0
goto loop1
)
```

