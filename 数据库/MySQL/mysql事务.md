# 事务
+ 事务是访问并更新数据库中各种数据项的一个程序执行单元。
+ MySQL通过set autocommit、start transaction、commit或rollback来支持本地事务。
## 事务的四大特性：
+ 原子性：是指整个数据库事务是不可分割的工作单位。只有使事务中所有的数据库操作都执行成功，才算整个事务成功。事务中任何一个SQL语句执行失败，已经执行成功的SQL语句也必须撤销，数据库状态应该退回到执行事务前的状态。
+ 一致性：一致性指事务将数据从一种状态转变为下一种一致的状态。在事务开始之前或事务结束以后，数据库的完整性约束没有被破坏。
+ 隔离性：也叫做并发控制、可串行化、锁等。要求每个读写事务的对象对其他事务的操作对象能相互分离，即该事务提交前对其他事务都不可见。通常使用锁来实现。
+ 持久性：事务一旦提交，结果就是永久性的。即使宕机等故障，数据库也能使数据恢复。<font color="red">只能从事务本身来考虑事物的永久性。持久性保证事务系统的高可靠性。</font>
## 不考虑隔离性带来的问题
+ 脏读：就是在不同的事务下，当前事务可以读到另外事务未提交的数据。就是可以读到脏数据。
    + 脏数据：事务对缓冲池中行记录的修改，并且还没有被提交。
    + 脏页：指的是在缓冲池中已经被修改的页，但是还没有刷新到磁盘中，即数据库实例内存中的页和磁盘中的页的数据是不一样的
+ 不可重复读：是指在一个事务内多次读取同一数据集合。在这个事务还没有结束时，另外一个事务也访问该同一数据集合，并做了一些DML操作。
    + 不可重复读与脏读的区别：
        + 脏读是得到未提交数据，而不可重复读读到的是已提交的数据。但是违反了事务一致性的要求。
+ 丢失更新：一个数据的更新操作会被另一个事务的更新操作所覆盖，从而导致数据的不一致。
+ 幻读：两次获取的数据不一致(第一个事务对表中的数据进行修改,这种修改涉及到了表中全部的数据行,同时第二个事务也操作表中的数据,是向表中插入一条数据,此时第一个事务就会发现还没有修改的数据行,就像产生了幻觉一样
## 事务的分类
+ 扁平事务：是应用程序成为原子操作的基本组成模块。由BEGIN WORK开始，ROLLBACK 或 COMMIT结束。主要限制是：不能提交或回滚事务的某一部分。
+ 带有保存点的扁平事务：除了支持扁平事务的操作外，允许事务在执行过程中回滚到同一事务中较早的一个标志。 保存点用来通知系统应该记住事务当前的状态，以便当之后发生错误时，事务能回到保存点当时的状态。
+ 链事务：
+ 嵌套事务：
+ 分布式事务：通常是一个运行环境下的扁平事务。
## 事物的实现
+ redo
+ undo
## 事务的隔离级别
+ READ UNCOMMITTED：称为浏览访问，仅仅针对事务而言。(未提交读)
+ READ COMMITTED：游标稳定。(提交读,解决了脏读)
+ REPEATABLE READ：没有幻读的保护。(InnoDB默认的存储引擎)(解决脏读、不可重复读)
+ SERIALIZABLE：隔离。(解决脏读,不可重复读,虚读(但是会导致锁表))  
  解决丢失更新:添加锁机制
<font color="red">隔离级别越低，事务请求的锁越少或保持锁的时间就越短。</font>