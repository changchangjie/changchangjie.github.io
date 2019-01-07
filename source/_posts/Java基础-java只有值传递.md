---
title: Java基础-java只有值传递
date: 2018-10-09 15:40:00
categories: 
- java基础
---

Java程序设计语言总是采用按值调用。也就是说，方法得到的是所有参数值的一个拷贝，也就是说，方法不能修改传递给它的任何参数变量的内容
<!--more-->

### example1
```java
public static void main(String[] args) {
    int num1 = 10;
    int num2 = 20;

    swap(num1, num2);

    System.out.println("num1 = " + num1);
    System.out.println("num2 = " + num2);
}

public static void swap(int a, int b) {
    int temp = a;
    a = b;
    b = temp;

    System.out.println("a = " + a);
    System.out.println("b = " + b);
}
```
```java
a = 20
b = 10
num1 = 10
num2 = 20
```
解析
![解析](https://camo.githubusercontent.com/ab46506b1a5ce09a516051c35f981e55255337f9/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f32323139313334382e6a7067)
在swap方法中，a、b的值进行交换，并不会影响到 num1、num2。因为，a、b中的值，只是从 num1、num2 的复制过来的。也就是说，a、b相当于num1、num2 的副本，副本的内容无论怎么修改，都不会影响到原件本身

一个方法不能修改一个基本数据类型的参数，而对象引用作为参数就不一样

### example2
```java
public static void main(String[] args) {
    int[] arr = { 1, 2, 3, 4, 5 };
    System.out.println(arr[0]);
    change(arr);
    System.out.println(arr[0]);
}

public static void change(int[] array) {
    // 将数组的第一个元素变为0
    array[0] = 0;
}
```
```java
1
0
```
解析
![解析](https://camo.githubusercontent.com/b7bad9506150c29bb8d7debd3905bd7a71cd6611/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f333832353230342e6a7067)
array 和 arr 指向的时同一个数组对象。 因此，外部对引用对象的改变会反映到所对应的对象上。

### example3
```java
public static void main(String[] args) {
    Student s1 = new Student("小张");
    Student s2 = new Student("小李");
    swap(s1, s2);
    System.out.println("s1:" + s1.getName());
    System.out.println("s2:" + s2.getName());
}

public static void swap(Student x, Student y) {
    Student temp = x;
    x = y;
    y = temp;
    System.out.println("x:" + x.getName());
    System.out.println("y:" + y.getName());
}
```
```java
x:小李
y:小张
s1:小张
s2:小李
```
解析
![交换前](https://camo.githubusercontent.com/9d6dd0313695d309280675cd3251b47432a28814/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f38383732393831382e6a7067)
交换前s1和其副本x指向同一个元素，s2和y指向的是同一个元素
![交换后](https://camo.githubusercontent.com/6bea9b0ed65609d699207ab787f631f7ba0a9246/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d392d32372f33343338343431342e6a7067)
交换后s1指针没变，其副本x指针变了，s2指针没变，其副本y指针变了
其实交换的其两个副本的指针，原对象指针不受影响

### 参考链接
> [为什么 Java 中只有值传递](https://github.com/Snailclimb/JavaGuide/blob/master/%E9%9D%A2%E8%AF%95%E5%BF%85%E5%A4%87/%E6%9C%80%E6%9C%80%E6%9C%80%E5%B8%B8%E8%A7%81%E7%9A%84Java%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93/%E7%AC%AC%E4%B8%80%E5%91%A8%EF%BC%882018-8-7%EF%BC%89.md)  






