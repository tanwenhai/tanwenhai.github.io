---
title: Java集合
date: 2018-03-12 19:21:01
tags:
---

# Java集合
## ArrayList
* ArrayList使用数组存储每次添加进行扩容判断 如果容量不够增加0.5倍
* 指定位置插入数据首先会扩容校验，接着对数据复制

## Vector
* Vector底层数据结构和**ArrayList**类似，是线程安全的。
* add方法的时候扩容大小是构造函数参数**capacityIncrement**如果没传或者是0 默认是1倍

## LinkedList
* LinkedList 底层是基于双向链表实现的
* add方法在last节点插入
* get方法根据参数index判断是从头部查找还是尾部查找

## HashMap
HashMap 底层是基于数据和链表实现的。其中有两个重要的参数：
* 容量
* 负载因子
> 容量的默认大小是 16，负载因子是 0.75，当 HashMap 的 size > 容量*负载因子 时就会发生扩容(容量和负载因子都可以自由调整)。

并发场景发生扩容，调用 resize() 方法里的 rehash() 时，容易出现环形链表。这样当获取一个不存在的 key 时，计算出的 index 正好是环形链表的下标时就会出现死循环

在 JDK1.8 中对 HashMap 进行了优化： 当 hash 碰撞之后写入链表的长度超过了阈值(默认为8)，链表将会转换为红黑树。

## LinkedHashMap 
底层是继承于**HashMap**实现的，由一个双向链表所构成

LinkedHashMap 的排序方式有两种：
* 根据写入顺序排序。
* 根据访问顺序排序

## HashSet
* 底层是基于**HashMap**实现的，使Set的值做为Map的Key,PRESENT做为value
    ```java
    private static final Object PRESENT = new Object();
    ```
* add的时候调用map的put方法如果返回null之前不存在
    ```java
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
    ```
## LinkedHashSet
底层是继承于**HashSet**实现的，构造函数调用HashSet的三参构造函数使底层存储使用LinkedHashMap
```java
public LinkedHashSet() {
    super(16, .75f, true);
}

HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```
