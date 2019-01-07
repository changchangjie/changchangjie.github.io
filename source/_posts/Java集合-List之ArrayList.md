---
title: Java集合-List之ArrayList
date: 2018-10-10 15:10:00
categories: 
- java集合
---

ArrayList 的底层是数组队列，相当于动态数组。与 Java 中的数组相比，它的容量能动态增长
```java
transient Object[] elementData;
```
transient作用是为了序列化时只序列化数组内的实际元素
<!--more-->

### 源码分析
1. 属性
```java
transient Object[] elementData;//保存ArrayList数据的数组
private static final int DEFAULT_CAPACITY = 10;//缺省容量
private static final Object[] EMPTY_ELEMENTDATA = {};//空对象数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};//缺省空对象数组
private int size;//实际元素大小，默认为0
```

2. 构造方法
```java
public ArrayList() {
	this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;//并不会真正产生实例化的数组，而是引用一个静态的空数组
}

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
    	this.elementData = new Object[initialCapacity];//创建指定大小的数组
    } else if (initialCapacity == 0) {
    	this.elementData = EMPTY_ELEMENTDATA;//并不会真正产生实例化的数组，而是引用一个静态的空数组
    } else {
    	throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}

public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();//转成数组
    if ((size = elementData.length) != 0) {//数组不为空
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);//数组拷贝生成新数组对象
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

3. add方法
```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // 扩容判断
    elementData[size++] = e;//添加到末尾
    return true;
}
private void ensureCapacityInternal(int minCapacity) {
	ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}
private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
    return Math.max(DEFAULT_CAPACITY, minCapacity);//使用无参构造实例化时，引用的是空数组，添加第一个元素这里扩展成默认容量10
    }
    return minCapacity;
}
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
    grow(minCapacity);//扩容
}
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;//旧容量
    int newCapacity = oldCapacity + (oldCapacity >> 1);//新容量扩成原来的1.5倍 右移n就是除以2的n次方
    if (newCapacity - minCapacity < 0)
    	newCapacity = minCapacity;//扩到1.5倍还不够的话，一步到位
    if (newCapacity - MAX_ARRAY_SIZE > 0)
    	newCapacity = hugeCapacity(minCapacity);//最大容量判断
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);//拷贝
}
```
4. addAll方法
```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  //先扩容
    System.arraycopy(a, 0, elementData, size, numNew);//将目标数组元素全部拷贝到原元素组后面
    size += numNew;
    return numNew != 0;
}
```
5. remove方法
```java
//根据索引位置去删除
public E remove(int index) {
    rangeCheck(index);//检查索引是否合法

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
    System.arraycopy(elementData, index+1, elementData, index,
    numMoved);//从指定位置后面往前复制
    elementData[--size] = null; //赋值为空，有利于进行GC

    return oldValue;//返回旧值
}

//根据元素内容去删除
public boolean remove(Object o) {
    if (o == null) {//可以看出Arraylist允许元素为null
    for (int index = 0; index < size; index++)
    	if (elementData[index] == null) {
    		fastRemove(index);
    		return true;
    	}
    } else {
        for (int index = 0; index < size; index++)
        	if (o.equals(elementData[index])) {
        		fastRemove(index);//先找到元素位置，在根据索引位置去删除
        		return true;
        	}
    }
    return false;
}

private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
    	System.arraycopy(elementData, index+1, elementData, index,
    numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```

5. get方法
```java
public E get(int index) {
    rangeCheck(index);

    return elementData(index);
}
```

### 减少扩容次数
在创建ArrayList之前知道要插入的对象的数目的话，直接使用有参构造即可
当创建ArrayList之后知道要插入的对象的数目的话，最好提前扩容，不然每添加到一定元素就需要扩容拷贝，可调用ensureCapacity(int minCapacity)方法扩容一次即可，来提高效率
```java
//扩容到可容纳minCapacity
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
    // any size if not default element table
    ? 0
    // larger than default for default empty table. It's already
    // supposed to be at default size.
    : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
    	ensureExplicitCapacity(minCapacity);
    }
}
```

### ArrayList与数组的区别
既然ArrayList内部由数组实现,那二者有什么区别呢？
1. 数组可存放基本类型和对象类型，而ArrayList只可存放对象类型
2. ArrayList提供了更丰富的api操作
3. Arraylist大小是动态变化的，Array定长