---
title: guava_1.3_常见Object方法
date: 2016-08-22 11:39:51
categories: guava
tags: Objects
---
# equals
当一个对象中的字段可以为null时，实现Object.equals方法会很痛苦，因为不得不分别对它们进行null检查。使用Objects.equal帮助你执行null敏感的equals判断，从而避免抛出NullPointerException。
<!--more-->

```
Objects.equal("a", "a"); // returns true
Objects.equal(null, "a"); // returns false
Objects.equal("a", null); // returns false
Objects.equal(null, null); // returns true
```
>注意：JDK7引入的Objects类提供了一样的方法Objects.equals。

# hashCode
用对象的所有字段作散列[hash]运算应当更简单。Guava的Objects.hashCode(Object...)会对传入的字段序列计算出合理的、顺序敏感的散列值。你可以使用Objects.hashCode(field1, field2, …, fieldn)来代替手动计算散列值。
>注意：JDK7引入的Objects类提供了一样的方法Objects.hash(Object...)


# toString 
好的toString方法在调试时是无价之宝，但是编写toString方法有时候却很痛苦。使用 Objects.toStringHelper可以轻松编写有用的toString方法
```
// Returns "ClassName{x=1}"
Objects.toStringHelper(this).add("x", 1).toString();
// Returns "MyObject{x=1}"
Objects.toStringHelper("MyObject").add("x", 1).toString();
```

# compare/compareTo
实现一个比较器[Comparator]，或者直接实现Comparable接口有时也伤不起
```
class Person implements Comparable<Person> {
  private String lastName;
  private String firstName;
  private int zipCode;
 
  public int compareTo(Person other) {
    int cmp = lastName.compareTo(other.lastName);
    if (cmp != 0) {
      return cmp;
    }
    cmp = firstName.compareTo(other.firstName);
    if (cmp != 0) {
      return cmp;
    }
    return Integer.compare(zipCode, other.zipCode);
  }
}
```
这部分代码太琐碎了，因此很容易搞乱，也很难调试。我们应该能把这种代码变得更优雅，为此，Guava提供了ComparisonChain。
ComparisonChain执行一种懒比较：它执行比较操作直至发现非零的结果，在那之后的比较输入将被忽略。
```
public int compareTo(Foo that) {
    return ComparisonChain.start()
            .compare(this.aString, that.aString)
            .compare(this.anInt, that.anInt)
            .compare(this.anEnum, that.anEnum, Ordering.natural().nullsLast())
            .result();
}
```
这种Fluent接口风格的可读性更高，发生错误编码的几率更小，并且能避免做不必要的工作