---
title: Java基础-内部类中调用外部局部变量为何用final修饰
date: 2018-10-10 10:45:00
categories: 
- java基础
---
在平时写代码中经常会在方法中起一个线程，但是在局部内部类中使用外部局部变量的话编译器会提示将外部局部变量定义为final类型，这是为什么呢
```java
public void test(final String a, String b){
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + ":"+a);
    	}
	}).start();
}
```
<!--more-->

1. 在局部内部类可以访问方法的形参或局部变量
从程序设计语言的理论上:局部内部类(即:定义在方法中的内部类),由于本身就是在方法内部,因而可以访问方法中的局部变量(形式参数或局部变量)
2. 怎么解决局部变量的生命周期与局部内部类的对象的生命周期的不一致
当在内部类试图访问外部方法中的局部变量时，外部方法的局部变量很可能已经不存在了，那么就得延续其生命，拷贝到内部类中，而拷贝会带来不一致性，从而需要使用final声明保证一致性。
3. java8的优化
1.8之前在局部内部类使用外部局部变量的话，需要将外部局部变量显式定义为final类型，不然编译期就会提示出错
1.8就不需要显式定义为final类型了，但是如果试图改变外部的局部变量引用的话，就会提示需要定义为final类型，其实是语法糖带来的效果
4. 总结
复制保证生命周期延续，final保证引用一致

### 参考链接
> [为什么匿名内部类参数必须为final类型](http://feiyeguohai.iteye.com/blog/1500108)  
> [为在jdk1.8以前匿名内部类使用外部类变量必须是final的原因](https://www.jianshu.com/p/d70f9832d66b)
