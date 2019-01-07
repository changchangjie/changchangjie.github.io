---
title: Spring-Spring Quartz和Task的区别
date: 2018-10-19 14:00:00
categories: 
- Spring
---

目前比较常用的定时任务解决方案一般就是Spring Quartz和Task

### spring-quartz
spring-quartz只是spring对quartz的一个包装而已。其实现是在spring-context-support中
1. 特点
- 默认多线程异步执行
- 单个任务时，在上一个调度未完成时，下一个调度时间到时，会另起一个线程开始新的调度。业务繁忙时，一个任务会有多个调度，可能导致数据处理异常
- 多个任务时，任务之间没有直接影响，多任务执行的快慢取决于CPU的性能
<!--more-->

2. 使用
```xml
    <bean id="job" class=" xx.xx.xx.Job" />
    <bean id="cronTask" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
        <property name="targetObject" ref="job" />
        <property name="targetMethod" value="runWork" />
        <!-- false表示job不会并发执行，默认为true-->
        <property name="concurrent" value="false" />
    </bean>
    <!—配置触发器-->
    <bean id="doWork" class="org.springframework.scheduling.quartz.CronTriggerBean">
        <property name="jobDetail" ref="cronTask" />
        <!—每天凌晨0点1分执行-->
        <property name="cronExpression" value="0 01 00 * * ?" />
    </bean>
    <!—最后配置调度工厂-->
    <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
            <list>
                <ref local="doWork"/>
            </list>
        </property>
    </bean>
```

### spring-task
Spring从3.0开始增加了自己的任务调度器，它是通过扩展java.util.concurrent包下面的类来实现的，也称spring-schedule
1. 特点

- 默认单线程同步执行
- 单个任务时，当前次的调度完成后，再执行下一次任务调度
- 多个任务时，一个任务执行完成后才会执行下一个任务。若需要任务能够并发执行，需手动设置线程池

2. 默认单线程同步执行
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
    http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.0.xsd">

    <task:annotation-driven />

    <bean id="myTask" class="me.changjie.task.MyTask"></bean>

    <!--<task:scheduler id="scheduler" pool-size="5" />-->

    <!--<task:scheduled-tasks scheduler="scheduler">-->
    <task:scheduled-tasks>
        <!--两个任务间隔10s执行-->
        <task:scheduled ref="myTask" method="method1" cron="0 33 13 * * ?" />
        <task:scheduled ref="myTask" method="method2" cron="10 33 13 * * ?"/>
    </task:scheduled-tasks>

</beans>
```
```java
public class MyTask {

    public void method1() throws InterruptedException {
        System.out.println("method1 execute thread:"+Thread.currentThread().getName());
        System.out.println("method1 execute time:"+new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
        TimeUnit.SECONDS.sleep(20);
    }

    public void method2() {
        System.out.println("method2 execute thread:"+Thread.currentThread().getName());
        System.out.println("method2 execute time:"+new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
    }

}
```
输出
```java
method1 execute thread:pool-1-thread-1
method1 execute time:2018-10-19 13:33:00
method2 execute thread:pool-1-thread-1
method2 execute time:2018-10-19 13:33:20
```

3. 设置线程池使其任务并行

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
    http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.0.xsd">

    <task:annotation-driven />

    <bean id="myTask" class="me.changjie.task.MyTask"></bean>

    <task:scheduler id="scheduler" pool-size="5" />

    <task:scheduled-tasks scheduler="scheduler">
    <!--<task:scheduled-tasks>-->
        <task:scheduled ref="myTask" method="method1" cron="0 36 13 * * ?" />
        <task:scheduled ref="myTask" method="method2" cron="10 36 13 * * ?"/>
    </task:scheduled-tasks>

</beans>
```
输出
```java
method1 execute thread:scheduler-1
method1 execute time:2018-10-19 13:36:00
method2 execute thread:scheduler-2
method2 execute time:2018-10-19 13:36:10
```

### 对比
易用性
- spring-quartz配置较复杂(任务、触发器、调度工厂)，但拥有spring-task所有的功能
- spring-task配置较简单，但是默认单线程同步，为了避免任务阻塞，最好配置成线程池异步执行

异常处理
-  Quartz的某次执行任务过程中抛出异常，不影响下一次任务的执行，当下一次执行时间到来时，定时器会再次执行任务
-  SpringTask不同，一旦某个任务在执行过程中抛出异常，则整个定时器生命周期就结束，以后永远不会再执行定时器任务，所以最好全范围catch掉






