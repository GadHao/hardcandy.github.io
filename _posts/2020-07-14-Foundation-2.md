---
layout: post
title: '地基系列(1) - 数据库(1) 数据库系统实现'
date: 2020-07-14
author: HANABI
color: rgb(54,59,64)
tags: 地基系列 DBMS
---

> 整理数据库系统(主要是关系数据库系统[^1])学习过程中的相关知识点，数据库系统实现

## 辅助存储管理(辅存)

要了解辅存，我们先要了解一些概念

### 存储器层次

典型的计算机系统包括几个不同的可以储存数据的部件，这些部件中，最小容量的设备提供了最快的访问速度，也具有最高的每个字节的价格，这里用一张[知乎回答](https://www.zhihu.com/question/28445273/answer/143956523)上的图

<table border="1">
    <tr>
        <td rowspan="10">
            存储系统
        </td>
        <td rowspan="6">
            内存存储器(内存，interal memory)
        </td>
        <td>寄存器(register)</td>
        <td>在CPU内部</td>
    </tr>
    <tr>
        <td>高速缓冲处理器(cache)</td>
        <td>现在一般集成在CPU内部</td>
    </tr>
    <tr>
        <td rowspan="4">主存储器(主存，main memory)<br>
            (其最大可利用空间由地址总线宽度决定)
        </td>
        <td>内存条</td>
    </tr>
    <tr>
        <td>显卡中的RAM芯片</td>
    </tr>
    <tr>
        <td>接口卡中ROM芯片</td>
    </tr>
    <tr>
        <td>等等</td>
    </tr>
    <tr>
        <td rowspan="4" colspan="2">外部存储器，辅助存储器(外存，辅存，external memory，secondary memory)</td>
        <td>
            硬盘
        </td>
    </tr>
    <tr>
        <td>U盘</td>
    </tr>
    <tr>
        <td>光盘</td>
    </tr>
    <tr>
        <td>等等</td>
    </tr>
</table>

[^1]: 1970年*Ted Codd*发表了一篇著名的论文(*CODASYL Data Base Task Group April 1971 Report, ACM, New York*)，里面认为数据库系统应该呈现给用户一种被组织成叫做"关系"的表的数据。在幕后，应该有一个复杂的数据结构，允许对各式各样的查询进行快速相应。关系数据库的程序员不需要关心存储及结构。查询可以用很高级的语言来表达，而*SQL(结构化查询语言)*就是这一基于关系模型的最重要的查询语言。

