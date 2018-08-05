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