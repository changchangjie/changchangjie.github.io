---
title: Java基础-equals与hashcode
date: 2018-10-09 15:00:00
categories: 
- java基础
---

### equals方法
euqals()作为Object的方法，用于判断两个对象是否相等
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
很明显是对两个对象的地址值进行的比较（即比较引用是否相同）。像String 、Math、Integer、Double等这些封装类已经将其重写了，比较的是内容
<!--more-->

### hashcode方法
hashCode()也是Object的方法，用于给对象返回一个hash code值。这个方法被用于hash tables，例如HashMap
```java
public native int hashCode();
```
是一个本地方法，它的实现是根据本地机器相关,像String 、Math、Integer、Double等这些封装类已经将其重写了

### hashcode作用
对于java中的set集合元素是不可重复的，元素是否重复通过equals方法判断。
但是，如果每增加一个元素就检查一次，那么当元素很多时，比较的次数就非常多了。这显然会大大降低效率。   
于是，Java采用了哈希表的原理。哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上。  
当集合要添加新的元素时，先调用这个元素的hashCode方法，就一下子能定位到它应该放置的物理位置上。如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了；如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址。所以这里存在一个冲突解决的问题。这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。  
简而言之，在集合查找时，hashcode能大大降低对象比较次数，提高查找效率

### 规定
Java对象的eqauls方法和hashCode方法是这样规定的：
1、相等（相同）的对象必须具有相等的哈希码（或者散列码）。
由于HashMap不允许存放重复元素，所以相同元素的hashcode必须不同
所以重写了equals的话必须重写hashcode
2、如果两个对象的hashCode相同，它们并不一定相同。
不同对象的hashcode可能会相同，即会产生哈希冲突

### 哈希冲突
假如两个Java对象A和B，A和B不相等（eqauls结果为false），但A和B的哈希码相等，将A和B都存入HashMap时会发生哈希冲突，也就是A和B存放在HashMap内部数组的位置索引相同这时HashMap会在该位置建立一个链表，将A和B串起来放在该位置，显然，该情况不违反HashMap的使用原则，是允许的。当然，哈希冲突越少越好，尽量采用好的哈希算法以避免哈希冲突

### 原文链接
> [Java提高篇——equals()与hashCode()方法详解](https://www.cnblogs.com/Qian123/p/5703507.html)  