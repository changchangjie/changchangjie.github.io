---
title: Java基础-Transient关键字的使用
date: 2018-09-30 12:12:57
categories: 
- java基础
---

### transient的使用

对象实现了序列化接口时，这个类所有属性和方法都可以序列化和被反序列化，当我们不想序列化某些属性时，使用transient修饰这些属性即可，比如对于一些敏感信息(密码，银行卡号等)不希望在网络中传输  
<!--more-->
#### 传统序列化
```java
public class TransientTest {

    public static class User implements Serializable{

        private String username;

        private String password;

        public String getUsername() {
            return username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public String getPassword() {
            return password;
        }

        public void setPassword(String password) {
            this.password = password;
        }
    }

    public static void main(String[] args) {
        User user = new User();
        user.setUsername("chang");
        user.setPassword("123456");

        System.out.println("read before Serializable: ");
        System.out.println("username: " + user.getUsername());
        System.out.println("password: " + user.getPassword());

        try {
            ObjectOutputStream os = new ObjectOutputStream(
                    new FileOutputStream("/Users/admin/Desktop/user.txt"));
            os.writeObject(user); // 将User对象写进文件
            os.flush();
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }

        try {
            ObjectInputStream is = new ObjectInputStream(new FileInputStream(
                    "/Users/admin/Desktop/user.txt"));
            user = (User) is.readObject(); // 从流中读取User的数据
            is.close();

            System.out.println("\nread after Serializable: ");
            System.out.println("username: " + user.getUsername());
            System.out.println("password: " + user.getPassword());

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
输出
```java
read before Serializable: 
username: chang
password: 123456

read after Serializable: 
username: chang
password: 123456
```
#### 使用transient修饰不想被序列化的字段
```java
private transient String password;
```
输出
```java
read before Serializable: 
username: chang
password: 123456

read after Serializable: 
username: chang
password: null
```
密码字段为null，说明反序列化时根本没有从文件中获取到信息
### 静态变量不管是否被transient修饰，均不能被序列化
```java
public class TransientTest {

    public static class User implements Serializable{

        private static final long serialVersionUID = 7195941599855555739L;
        private String username;

        private static String password;

        public String getUsername() {
            return username;
        }

        public void setUsername(String username) {
            this.username = username;
        }

        public String getPassword() {
            return password;
        }

        public void setPassword(String password) {
            this.password = password;
        }
    }

    public static void main(String[] args) {
        User user = new User();
        user.setUsername("chang");
        user.setPassword("123456");

        System.out.println("read before Serializable: ");
        System.out.println("username: " + user.getUsername());
        System.out.println("password: " + user.getPassword());

        try {
            ObjectOutputStream os = new ObjectOutputStream(
                    new FileOutputStream("/Users/admin/Desktop/user.txt"));
            os.writeObject(user); // 将User对象写进文件
            os.flush();
            os.close();
        } catch (Exception e) {
            e.printStackTrace();
        }

        try {
            User.password = "1111";//在反序列化之前修改了静态变量的值
            ObjectInputStream is = new ObjectInputStream(new FileInputStream(
                    "/Users/admin/Desktop/user.txt"));
            user = (User) is.readObject(); // 从流中读取User的数据
            is.close();

            System.out.println("\nread after Serializable: ");
            System.out.println("username: " + user.getUsername());
            System.out.println("password: " + user.getPassword());

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
输出
```java
read before Serializable: 
username: chang
password: 123456

read after Serializable: 
username: chang
password: 1111
```
可知反序列化后类中静态变量的值为当前JVM中对应static变量的值，而不是序列化时的值
### ArrayList的实现
ArrayList实现了List和Serializable接口，内部由elementData数组实现，
```java
private transient Object[] elementData;
private int size;

public ArrayList(int var1) {
    if (var1 < 0) {
        throw new IllegalArgumentException("Illegal Capacity: " + var1);
    } else {
        this.elementData = new Object[var1];
    }
}

public ArrayList() {
    this(10);
}
```
可知数组的初始化容量是10，每插入一个元素时都要进行最大容量判断
```java
public boolean add(E var1) {
    this.ensureCapacity(this.size + 1);//扩容校验
    this.elementData[this.size++] = var1;//将插入的值放到尾部
    return true;
}
public void ensureCapacity(int var1) {
    ++this.modCount;
    int var2 = this.elementData.length;
    if (var1 > var2) {//超过容量需要进行Arrays.copyOf扩容
        Object[] var3 = this.elementData;
        int var4 = var2 * 3 / 2 + 1;
        if (var4 < var1) {
            var4 = var1;
        }

        this.elementData = Arrays.copyOf(this.elementData, var4);
    }
}
```
指定位置插入元素的实现
```java
public void add(int var1, E var2) {
    if (var1 <= this.size && var1 >= 0) {
        this.ensureCapacity(this.size + 1);//扩容校验
        System.arraycopy(this.elementData, var1, this.elementData, var1 + 1, this.size - var1);//将指定位置后面的元素后移
        this.elementData[var1] = var2;//添加到指定位置
        ++this.size;
    } else {
        throw new IndexOutOfBoundsException("Index: " + var1 + ", Size: " + this.size);
    }
}
```
由此可见 ArrayList的主要消耗是数组扩容以及在指定位置添加数据，在日常使用时最好是指定大小，尽量减少扩容。更要减少在指定位置插入数据的操作
### ArrayList与transient
通过源码得知ArrayList实现了序列化接口并且elementData是被transient修饰的，那不是序列化后的ArrayList丢失了原先的元素？  
ArrayList内部还有两个方法
```java
private void writeObject(ObjectOutputStream var1) throws IOException {
    int var2 = this.modCount;
    var1.defaultWriteObject();
    var1.writeInt(this.elementData.length);

    for(int var3 = 0; var3 < this.size; ++var3) {
        var1.writeObject(this.elementData[var3]);
    }

    if (this.modCount != var2) {
        throw new ConcurrentModificationException();
    }
}

private void readObject(ObjectInputStream var1) throws IOException, ClassNotFoundException {
    var1.defaultReadObject();
    int var2 = var1.readInt();
    Object[] var3 = this.elementData = new Object[var2];

    for(int var4 = 0; var4 < this.size; ++var4) {
        var3[var4] = var1.readObject();
    }
}
```
为什么不直接用elementData来序列化，而采用上述的方式来实现序列化呢？原因在于elementData是一个缓存数组，它通常会预留一些容量，等容量不足时再扩充容量，那么有些空间可能就没有实际存储元素，采用上诉的方式来实现序列化时，就可以保证只序列化实际存储的那些元素，而不是整个数组，从而节省空间和时间
### 原文链接
> [Java transient关键字使用小记](http://www.cnblogs.com/lanxuezaipiao/p/3369962.html)  
> [ArrayList中elementData为什么被transient修饰?](https://blog.csdn.net/zero__007/article/details/52166306)
