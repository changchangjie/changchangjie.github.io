---
title: Java集合-List之LinkedList
date: 2018-10-10 19:10:00
categories: 
- java集合
---
LinkedList底层的链表结构使它支持高效的插入和删除操作，另外它实现了Deque接口，使得LinkedList类也具有队列的特性
```java
private static class Node<E> {
    E item;//节点值
    Node<E> next;//前驱节点
    Node<E> prev;//后继节点

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```
<!--more-->

### 源码分析
1. 属性
```java
transient int size = 0;//链表的元素个数
transient Node<E> first;//头节点
transient Node<E> last;//尾节点
```
2. 构造方法
```java
public LinkedList() {
}
public LinkedList(Collection<? extends E> c) {
    this();//调用无参构造方法
    addAll(c);//添加元素
}
```
3. addFirst：将元素插入到最前面

```java
public void addFirst(E e) {
	linkFirst(e);
}
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);//创建新元素,前驱节点为null,后继节点为之前的头结点
    first = newNode;//将新元素设为头结点
    if (f == null)
    	last = newNode;
    else
    	f.prev = newNode;//将新元素作为原来头结点的前驱节点
    size++;
    modCount++;
}
```
4. addLast/add：将元素插入到最后面

```java
public void addLast(E e) {
	linkLast(e);
}
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);//创建新元素,后继节点为null,后继节点为之前的尾结点
    last = newNode;//将新元素设为尾结点
    if (l == null)
    	first = newNode;
    else
    	l.next = newNode;//将新元素作为原来尾结点的后继节点
    size++;
    modCount++;
}
```
5. add(int index, E element)：在指定位置插入元素

新节点的前驱为指定位置元素的前驱，新节点的后继指定位置元素
指定位置元素的前驱节点的后继设为新元素，指定位置的前驱设为新元素
```java
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
    	linkLast(element);
    else
    	linkBefore(element, node(index));
}
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;//找出指定位置的前驱节点
    final Node<E> newNode = new Node<>(pred, e, succ);//创建新元素，后继元素即为指定位置元素，前驱为指定位置的原前驱
    succ.prev = newNode;//将新元素作为指定位置元素的新前驱
    if (pred == null)
    	first = newNode;
    else
    	pred.next = newNode;//将新元素作为指定元素原前驱节点的新后继
    size++;
    modCount++;
}
```
6. removeFirst/remove：删除第一个元素

将头结点的后继节点设为新的头结点，再把新的头结点的前驱设置null

7. removeLast：删除最后一个元素

将最后元素的前驱作为新的尾节点，再把新的尾节点的后继设为null


8. remove(int index)：删除指定位置的元素

先根据索引找到这个节点(从头结点开始找)，再修改这个节点的前驱的后继和后继的前驱即可

9. remove(Object o)：删除指定内容的元素

先根据内容找到这个节点(从头结点开始找)，再修改这个节点的前驱的后继和后继的前驱即可

### 总结
无论是根据指定索引位置进行查找还是根据指定元素内容进行查找，都需要从头结点进行遍历
插入和删除的话只需要操作前驱和后继元素即可

