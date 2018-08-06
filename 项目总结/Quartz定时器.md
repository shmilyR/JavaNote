# 用法
在某一个有规律的时间点干某件事，并且时间的触发的条件可以非常复杂。
# 定时调度的几种实现方法：
1. Java自带的java.util.Timer类，这个类允许调度一个java.util.TimerTask任务。使用这种方法可以让程序按照某一个频度执行，但不能在指定时间运行。
2. 使用Quatrz,这是一个功能比较强大的调度器，可以让程序在指定的时间执行，也可以按照某一个频度执行。
3. Spring3.0以后自带的task，相当于一个轻量级的Quartz.
# quartz常用API
1. Job接口 需要自己来实现
2. JobDetail 提供一些监听，主要用于扩展
3. Trigger 触发器 可以人为是具体的实现方法
4. Calendar 记录所有的任务触发点 
5. scheduler 调度器 独立的运行容器，所有的任务相关的东西全部放在调度器里面
6. schedulerFactory 调度工厂 Spring提供的
## 表达式：cronExpression 
1. [秒] [分] [时] [日] [月] [周] [年]
   参考了Linux的cron
定时器的简单配置：
1. 写出自己的任务类
```java
public class MyTask {
    public void myFirstTask() {
        System.out.println("定时器测试");
    }
}
```
2. 配置spring-quartz.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <!--注入自己所要执行的方法 -->
    <bean id="MyTask" class="com.java.task.MyTask"/>
    <!--关联要执行的方法的位置 -->
    <bean id="jobDetail" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
        <property name="targetObject" ref="MyTask"/>
        <property name="targetMethod" value="myFirstTask"/>
    </bean>
    <!--配置触发器 -->
    <bean id="cronTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="jobDetail"/>
        <property name="cronExpression" value="0/5 * * * * ? " />
    </bean>
    <!--配置调度工厂 -->
    <bean id="scheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
            <list>
                <ref bean="cronTrigger"/>
            </list>
        </property>
    </bean>
</beans>
```
3. web.xml的配置
```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:spring-quartz.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
</web-app>
```