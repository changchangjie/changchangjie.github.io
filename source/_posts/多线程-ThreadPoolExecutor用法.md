---
title: 多线程-ThreadPoolExecutor用法
date: 2018-10-12 15:10:00
categories: 
- 多线程
---

### 属性
```java
int corePoolSize;//核心线程数，默认情况下核心线程会一直存活，即使处于闲置状态也不会受存keepAliveTime限制。除非将allowCoreThreadTimeOut设置为true
int maximumPoolSize//线程池所能容纳的最大线程数。超过这个数的线程将被阻塞。当任务队列为没有设置大小的LinkedBlockingDeque时，这个值无效
long keepAliveTime//非核心线程的闲置超时时间，超过这个时间就会被回收
TimeUnit unit//指定keepAliveTime的单位
BlockingQueue<Runnable> workQueue//线程池中的任务队列，常用的是：ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue
ThreadFactory threadFactory//线程工厂，提供创建新线程的功能
RejectedExecutionHandler handler//拒绝策略，线程池线程数达到最大值并且任务队列满时触发
```
<!--more-->
### 方法
1. getCorePoolSize()：返回核心线程数量，即corePoolSize
2. getPoolSize()：返回当前线程池中线程数，刚创建线程池时线程数为0，直到有任务提交过来才创建线程
3. getQueue().size()：获取当前任务队列的实际容量

### 任务队列
1. SynchronousQueue：直接提交的队列

该任务队列没有容量，新任务总是提交给线程执行，如果活跃线程达到最大值，新任务提交会执行拒绝策略，这种队列会导致线程数过多或太容易触发拒绝策略一般不建议使用

2. ArrayBlockingQueue：有界任务队列

构造函数必须传容量表示任务队列的最大值
新任务提交，若线程池实际线程数小于核心线程数的话会创建线程，等于核心线程数的话会把任务放入队列中，队列满的话创建新的线程，达到最大线程数并且队列满的话执行拒绝策略

3. LinkedBlockingQueue：无界任务队列

新任务提交，若线程池实际线程数小于核心线程数的话会创建线程，等于核心线程数的话会把任务放入队列中，由于是无界（其实也是有最大值的），队列基本不会满，新任务会一直放入队列中而不会创建新的线程，使用不当会发生内存溢出
所以一般会设置个容量，队列满时创建新线程，达到最大线程数并且队列满的话执行拒绝策略

### 拒绝策略
1. AbortPolicy：默认拒绝策略，新任务会被拒绝并抛出异常
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
	throw new RejectedExecutionException("Task " + r.toString() +
	" rejected from " +
	e.toString());
}
```
2. DiscardPolicy：直接丢弃被拒绝的任务
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
}
```
3. DiscardOldestPolicy：如果执行程序尚未关闭，则位于工作队列头部的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程）
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    if (!e.isShutdown()) {
    	e.getQueue().poll();
    	e.execute(r);
    }
}
```
4. CallerRunsPolicy：交由调用者线程来执行任务
```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    if (!e.isShutdown()) {
    	r.run();
    }
}
```






