---
layout: post
title: '.NET Core前后端分离(5) - IOC(控制反转)与DI(依赖注入)'
date: 2020-06-20
author: HANABI
color: rgb(91,47,107)
tags: [.NET Core]
hidden: true
---

> 准备好了所有的层，如何在各个*Service*层中调用*Repository*层的接口，并控制调用的*Repository*层的生命周期呢，之前写在各构造函数中的参数是如何获得的呢，让我们来了解IoC(Inversion of Control)以及DI(Dependence Injection)这两个概念

## IoC(控制反转)思想



