---
title: 大数据类BigDecimal
date: 2016-07-14 11:51:32
categories: java
tags: 金融
---

Java开发银行、金融类等需要对数值进行高精度计算时，使用**BigDecimal**和**BigInteger**，而不是常见的int、long、float、double类型，特别是在处理浮点型数据。

<!--more-->

## double 计算
```
public class DoubleTest {
    public static void main(String[] args) {
        System.out.println(0.02+0.01);
        System.out.println(0.05+0.01);
    }
}
//output 
0.03
0.060000000000000005
```
为什么会出现第二种结果的数据呢？根本原因还是我们的计算机是由二进制的，而二进制是没办法来精确表示一个浮点数，CPU采用“尾数和指数”的方式（科学计数法）表达浮点数的时候存在一定的误差。

## BigDecimal的使用
创建一个BigDecimal对象有构造函数和公有静态方法（BigDecimal.valueOf）两种方式
- 构造函数包含使用基本数据类型和字符串作为参数的两种形式，推荐使用后者，如：new BigDecimal(Double.valueOf(0.09))。大家可以尝试一下，System.out.println(new BigDecimal(0.06).toString());语句的输出结果是：0.059999999999999997779553950749686919152736663818359375
- Decimal打印日志或向基本数据类型转换时，尽量使用它提供的公有方法xxxValue()，比如doubleValue()，而不是简单粗暴的一个toString()

## BigDecimal舍入模式
尽管数据库存储的是一个高精度的浮点数，但是通常在应用中展示的时候往往需要限制一下小数点的位数，比如两到三位小数即可，这时就需要使用到setScale(int newScale, int roundingMode)函数，作为BigDecimal的公有静态变量，舍入模式（Rounding Mode）的运算规则比较多，有八种
- ROUND_UP 向远离零的方向舍入。舍弃非零部分，并将非零舍弃部分相邻的一位数字加一。
- ROUND_DOWN 向接近零的方向舍入。舍弃非零部分，同时不会非零舍弃部分相邻的一位数字加一，采取截取行为
- ROUND_CEILING 向正无穷的方向舍入。如果为正数，舍入结果同ROUND_UP一致；如果为负数，舍入结果同ROUND_DOWN一致。注意：此模式不会减少数值大小
- ROUND_FLOOR 向负无穷的方向舍入。如果为正数，舍入结果同ROUND_DOWN一致；如果为负数，舍入结果同ROUND_UP一致。注意：此模式不会增加数值大小。
- ROUND_HALF_UP 向“最接近”的数字舍入，如果与两个相邻数字的距离相等，则为向上舍入的舍入模式。如果舍弃部分>= 0.5，则舍入行为与ROUND_UP相同；否则舍入行为与ROUND_DOWN相同。这种模式也就是我们常说的我们的“四舍五入”
- ROUND_HALF_DOWN 向“最接近”的数字舍入，如果与两个相邻数字的距离相等，则为向下舍入的舍入模式。如果舍弃部分> 0.5，则舍入行为与ROUND_UP相同；否则舍入行为与ROUND_DOWN相同。这种模式也就是我们常说的我们的“五舍六入”
- ROUND_HALF_EVEN 向“最接近”的数字舍入，如果与两个相邻数字的距离相等，则相邻的偶数舍入。如果舍弃部分左边的数字奇数，则舍入行为与 ROUND_HALF_UP 相同；如果为偶数，则舍入行为与 ROUND_HALF_DOWN 相同。注意：在重复进行一系列计算时，此舍入模式可以将累加错误减到最小。此舍入模式也称为“银行家舍入法”，主要在美国使用。四舍六入，五分两种情况，如果前一位为奇数，则入位，否则舍去
- ROUND_UNNECESSARY 断言请求的操作具有精确的结果，因此不需要舍入。如果对获得精确结果的操作指定此舍入模式，则抛出ArithmeticException。

例如
输入数字|UP|DOWN|CEILING|FLOOR|HALF_UP|HALF_DOWN|HALF_EVEN|UNNECESSARY
---|---|---|---|
5.5|6|5|6|5|6|5|6|抛出ArithmeticException
1.1|2|1|2|1|2|2|2|抛出ArithmeticException
1.0|1|1|1|1|1|1|1|
-1.1|-2|-1|-1|-2|-1|-1|-1|抛出ArithmeticException
