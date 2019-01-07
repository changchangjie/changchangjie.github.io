---
title: Java基础-static关键字
date: 2018-10-10 11:10:00
categories: 
- java基础
---

static关键字主要用于下面几种场景
### 修饰成员变量和成员方法
被 static 修饰的成员变量属于类，不属于单个这个类的某个对象，被类中所有对象共享，建议通过类名调用。被static 声明的成员变量属于静态成员变量，静态变量存放在 Java 内存区域的方法区。
方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做 Non-Heap（非堆），目的应该是与 Java 堆区分开来。
HotSpot 虚拟机中方法区也常被称为 “永久代”，本质上两者并不等价。仅仅是因为 HotSpot 虚拟机设计团队用永久代来实现方法区而已，这样 HotSpot 虚拟机的垃圾收集器就可以像管理 Java 堆一样管理这部分内存了。但这样更容易遇到内存溢出问题
<!--more-->

### 修饰静态块
静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—非静态代码块—构造方法)。 该类不管创建多少对象，静态代码块只执行一次
由于静态代码块只执行一次，其实这里也可以去实现单例模式，本质上和饿汉模式一样，实例初始化过早，一般不建议使用

### 修饰内部类(只能修饰内部类)
静态内部类与非静态内部类之间存在一个最大的区别：非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外部类，但是静态内部类却没有
所以静态内部类的创建不需要依赖外部类的创建，也不能使用外部类的非static成员变量和方法(非静态内部类可直接使用外部类的所有(包括私有)属性和方法)
静态内部类比较常见的是实现线程安全的单例模式
```java
public class Singleton {
    
    //声明为 private 避免调用默认构造方法创建对象
    private Singleton() {
    }
    
    //声明为 private 表明静态内部该类只能在外部类中被访问
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getUniqueInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
当 外部类加载时，静态内部类并没有被加载进内存。只有当调用 getUniqueInstance()方法从而触发 SingletonHolder.INSTANCE 时 SingletonHolder 才会被加载，此时初始化 INSTANCE 实例，并且 JVM 能确保 INSTANCE 只被实例化一次。
这种方式不仅具有延迟初始化的好处，而且由 JVM 提供了对线程安全的支持

### 原文链接
> [static 关键字](https://github.com/Snailclimb/JavaGuide/blob/master/Java%E7%9B%B8%E5%85%B3/static.md)  

