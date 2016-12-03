---
title: Mysql中校对集utf8_unicode_ci与utf8_general_ci的区别说明
date: 2016-12-01 11:06:28
categories: 数据库
tags: mysql
---
# 说明
utf8_unicode_ci校对规则仅部分支持Unicode校对规则算法。一些字符还是不能支持。并且，不能完全支持组合的记号。 
这主要影响越南和俄罗斯的一些少数民族语言，如：Udmurt 、Tatar、Bashkir和Mari。

<!--more-->

utf8_unicode_ci的最主要的特色是支持扩展，即当把一个字母看作与其它字母组合相等时。例如，在德语和一些其它语言 
中‘ß'等于‘ss'。 
utf8_general_ci是一个遗留的 校对规则，不支持扩展。它仅能够在字符之间进行逐个比较。这意味着utf8_general_ci 
校对规则进行的比较速度很快，但是与使用utf8_unicode_ci的 校对规则相比，比较正确性较差

# 区别
utf8_unicode_ci和utf8_general_ci对中、英文来说没有实质的差别。 
utf8_general_ci校对速度快，但准确度稍差。 
utf8_unicode_ci准确度高，但校对速度稍慢。

如果你的应用有德语、法语或者俄语，请一定使用utf8_unicode_ci。一般用utf8_general_ci就够了， 
到现在也没发现问题。。。

utf8_unicode_ci比较准确，utf8_general_ci速度比较快。通常情况下 utf8_general_ci的准确性就够我们用的了， 
在我看过很多程序源码后，发现它们大多数也用的是utf8_general_ci，所以新建数据 库时一般选用utf8_general_ci就可以了