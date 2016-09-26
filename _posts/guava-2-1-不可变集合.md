---
title: guava-2-1-不可变集合
date: 2016-09-25 17:10:20
categories: guava
tags: 集合
---
## 实例

```
public static final ImmutableSet<String> COLOR_NAMES = ImmutableSet.of(
        "red",
        "orange",
        "yellow",
        "green",
        "blue",
        "purple");

class Foo {
    Set<Bar> bars;
    Foo(Set<Bar> bars) {
        this.bars = ImmutableSet.copyOf(bars); // defensive copy!
    }
}
```
<!--more-->
## 使用不可变集合
不可变对象有很多优点，包括：

- 当对象被不可信的库调用时，不可变形式是安全的；
- 不可变对象被多个线程调用时，不存在竞态条件问题
- 不可变集合不需要考虑变化，因此可以节省时间和空间。所有不可变的集合都比它们的可变形式有更好的内存利用率（分析和测试细节）；
- 不可变对象因为有固定不变，可以作为常量来安全使用。

## 不可变集合创建方式
- copyOf方法，如ImmutableSet.copyOf(set);
- of方法，如ImmutableSet.of(“a”, “b”, “c”)或 ImmutableMap.of(“a”, 1, “b”, 2);
- Builder工具，如
```
public static final ImmutableSet<Color> GOOGLE_COLORS =
        ImmutableSet.<Color>builder()
            .addAll(WEBSAFE_COLORS)
            .add(new Color(0, 191, 255))
            .build();
```

此外，对有序不可变集合来说，排序是在构造集合的时候完成的，如
```
ImmutableSortedSet.of("a", "b", "c", "a", "d", "b");
会在构造时就把元素排序为a, b, c, d
```

## 更智能的copyOf
ImmutableXXX.copyOf方法会尝试在安全的时候避免做拷贝——实际的实现细节不详，但通常来说是很智能的
```
ImmutableSet<String> foobar = ImmutableSet.of("foo", "bar", "baz");
thingamajig(foobar);

void thingamajig(Collection<String> collection) {
    ImmutableList<String> defensiveCopy = ImmutableList.copyOf(collection);
    ...
}
```
在这段代码中，ImmutableList.copyOf(foobar)会智能地直接返回foobar.asList(),它是一个ImmutableSet的常量时间复杂度的List视图。
作为一种探索，ImmutableXXX.copyOf(ImmutableCollection)会试图对如下情况避免线性时间拷贝：

在常量时间内使用底层数据结构是可能的——例如，ImmutableSet.copyOf(ImmutableList)就不能在常量时间内完成。
不会造成内存泄露——例如，你有个很大的不可变集合ImmutableList<String>
hugeList， ImmutableList.copyOf(hugeList.subList(0, 10))就会显式地拷贝，以免不必要地持有hugeList的引用。
不改变语义——所以ImmutableSet.copyOf(myImmutableSortedSet)会显式地拷贝，因为和基于比较器的ImmutableSortedSet相比，ImmutableSet对hashCode()和equals有不同语义。
在可能的情况下避免线性拷贝，可以最大限度地减少防御性编程风格所带来的性能开销。

## asList视图
所有不可变集合都有一个asList()方法提供ImmutableList视图，来帮助你用列表形式方便地读取集合元素。例如，你可以使用sortedSet.asList().get(k)从ImmutableSortedSet中读取第k个最小元素。

asList()返回的ImmutableList通常是——并不总是——开销稳定的视图实现，而不是简单地把元素拷贝进List。也就是说，asList返回的列表视图通常比一般的列表平均性能更好，比如，在底层集合支持的情况下，它总是使用高效的contains方法。

## 细节：关联可变集合和不可变集合
可变集合接口|  属于*JDK还是Guava* | 不可变版本
---|---|---
Collection | JDK| ImmutableCollection
List   | JDK| ImmutableList
Set| JDK| ImmutableSet
SortedSet/NavigableSet|  JDK| ImmutableSortedSet
Map| JDK| ImmutableMap
SortedMap|   JDK| ImmutableSortedMap
Multiset    |Guava|   ImmutableMultiset
SortedMultiset|  Guava|   ImmutableSortedMultiset
Multimap|    Guava  | ImmutableMultimap
ListMultimap |   Guava  | ImmutableListMultimap
SetMultimap |Guava   |ImmutableSetMultimap
BiMap  | Guava  | ImmutableBiMap
ClassToInstanceMap|  Guava |  ImmutableClassToInstanceMap
Table   |Guava   |ImmutableTable




