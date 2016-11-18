---
title: Junit4 单元测试框架介绍
date: 2016-11-16 10:12:51
categories: java
tags: 单元测试
---
经常写junit，突然想到了一个问题，经常用什么，测试什么

# JUnit常用注解
@Before：初始化方法，在任何一个测试执行之前必须执行的代码
@After：释放资源，在任何测试执行之后需要进行的收尾工作。在每个测试方法执行之后执行一次
@Test：测试方法，在这里可以测试期望异常和超时时间
@Ignore：忽略的测试方法，标注的含义就是“某些方法尚未完成，暂不参与此次测试”;
@BeforeClass：针对所有测试，只执行一次，且必须为static void，一般用于初始化必要的消耗较大的资源，例如数据库连接等
@AfterClass：针对所有测试，将会在所有测试方法执行结束后执行一次，且必须为static void
@RunWith:指定测试类使用某个运行器,比如spring的@RunWith(SpringJUnit4ClassRunner.class)
@Parameters: 指定测试类的测试数据集合

<!--more-->
# JUnit4单元测试用例执行顺序
@BeforeClass ==> [@Before ==> @Test ==> @After] ==> @AfterClass

# 常用断言
方法|说明
---|---|---
assertArrayEquals(expecteds, actuals)|  查看两个数组是否相等。
assertEquals(expected, actual)| 查看两个对象是否相等。类似于字符串比较使用的equals()方法
assertNotEquals(first, second)| 查看两个对象是否不相等。
assertNull(object)| 查看对象是否为空。
assertNotNull(object)|  查看对象是否不为空。
assertSame(expected, actual)|   查看两个对象的引用是否相等。类似于使用“==”比较两个对象
assertNotSame(unexpected, actual)|  查看两个对象的引用是否不相等。类似于使用“!=”比较两个对象
assertTrue(condition)|  查看运行结果是否为true。
assertFalse(condition)| 查看运行结果是否为false。
assertThat(actual, matcher)|    查看实际值是否满足指定的条件
fail()| 让测试失败


# @Test
限时测试 
@Test(timeout = 1000) 单位是毫秒//当方法用时超过1000毫秒时，此方法会自动中止并执行失败

异常处理
@Test(expected=Exception.class) 其中Exception.class可以写的更加具体
如果异常，则通过

# @Parameters
进行单元测试的时候，通常一个方法需要好几个case进行测试，Junit提供参数化便于我们对方法进行多种参数的组合测试
@RunWith(Parameterized.class)





