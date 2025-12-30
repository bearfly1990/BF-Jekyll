---
layout: post
title: Xpath的使用
subtitle: 使用Xpath定位页面中的元素
date: 2022-04-28
author: BF
thumbnail: /img/bf/web.jpg
catalog: true
toc: true
categories: Diary
header-img: /img/bf/web.jpg
tags:
    - web
    - xpath
    - selenium
---

## Overview

最近一段时间有在做界面 UI 的自动化，所以就需要定位元素来点击和输入。

`xpath`是相对万能的一种方式，网上已经有比较多好的总结了，整理了一下并加了点自己的理解。

在`Selenium(java)`中，我们可以使用`By.ByXPath("xpath")` 来定位元素。

<!--more-->

## 参考

- [Selenium XPATH 定位最全总结](https://testerhome.com/topics/20296)
- [selenium---Xpath 定位方法](https://www.cnblogs.com/qican/p/13183791.html)
- [关于 Xpath 定位方法知道这些基本够用](https://blog.51cto.com/u_15009374/3182032)

## Xpath 常用属性

| 表达式          | 描述                             |
| --------------- | -------------------------------- |
| nodename        | 选取此节点的所有子节点           |
| /               | 从当前节点选取直接子节点         |
| //              | 从当前节点选取子孙节点           |
| .               | 选取当前节点                     |
| ..              | 选取当前节点的父节点             |
| @               | 选取属性                         |
| \*              | 通配符，选择所有元素节点与元素名 |
| @\*             | 选取所有属性                     |
| [@属性]         | 选取具有给定属性的所有元素       |
| [@属性='value'] | 选取给定属性具有给定值的所有元素 |

## 常用例子

在xpath中的字符串用单引号和双引号都是ok的。

1、通过 id 属性定位
`//button[@id="kw"]`

2、通过 name 属性定位
`//input[@name="wd"]`

3、通过 class 属性定位
`//button[@class="primary-button"]`

4、通过 text 属性定位
`//label[text()="new"]`

5、通过 contains 方法定位
`//a[contains(text(),"map")]`

`//input[contains(@id,"kw")]`

## xpath 函数

| Column1       | Column2                                                                                                                                              |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| contains()    | `//div[contains(@id,'in')]`，表示选择 id 中包含有’in’的 div 节点                                                                                      |
| text()        | `//a[text()='baidu']`，用 text()函数来匹配节点                                                                                                        |
| last()        | `book[last()]` ，取 xpath 最后一个 book 元素 \n book[last()-1] ，取 xpath 最后第二个 book 元素                                                         |
| starts-with() | `//div[starts-with(@id,'in')]` 表示选择以’in’开头的 id 属性的 div 节点                                                                                 |
| not()         | `not()`函数，表示否定. `//input[@name=‘identity’ and not(contains(@class,'a'))]` ，表示匹配出 name 为 identity 并且 class 的值中不包含 a 的 input 节点。 |

特别注意

not()函数通常与返回值为 `true or false` 的函数组合起来用 `contains()`,`starts-with()`等，但有一种特别情况请注意一下。

我们要匹配出 input 节点含有 id 属性的，写法如下：`//input[@id]`，如果我们要匹配出 input 节点不含用 id 属性的，则为：`//input[not(@id)]`。

`xpath` 中的 `ends-with` 可能无效,原因是`ends-with` 是 xpath2.0 的语法,可能你的浏览器还只支持 1.0 的语法。

## 调试

在浏览器里，我们通常可以`F12`，然后在元素中`Ctrl+F`去使用XPath试着看能不能正确定位到这个元素。

有的时候元素需要点击某些按钮才会出来，这时需要进入调试模式，在一个元素那边右键,添加`子树修改`
![debug](/img/post/2022/04/2022-04-28-xpath-usage/debug.png)

如果能正确找到元素，就会如下所示：
![matched](/img/post/2022/04/2022-04-28-xpath-usage/matched.png)

通过元素的相对关系来查找的例子：
先找到label，向上一层之后，再找回下层中的的input,这个例子中可以直接用id什么的也可以，但是有时候没有好用的标记，就需要绕一下。

`//label[text()='邮箱地址']/../input[contains(@placeholder, "邮箱地址")]`

![by-other-element](/img/post/2022/04/2022-04-28-xpath-usage/by-other-element.png)
