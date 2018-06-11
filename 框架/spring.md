jms:java消息机制  
orm:对象映射  
# 面试必问  
+ HTTP:每次都必须有客户端发起向服务器端的请求，才可以获取信息  
 缺点：如果客户端不发起请求请求服务端的信息是无法到达客户端  
 > websocket:不需要客户端的请求，服务器可以向客户端发送信息（H5新特性）  
    阮一峰    

+ HTTP协议原理  
+ HTTPS协议原理 
+ 热部署：每次修改代码不需要启动服务器  
+ springBoot：集成服务器，写好之后不用再发布，可以直接运行。 
+ spring security  权限框架 
+ spring Cloud  微服务架构 云计算
+ spring Cloud Data Flow  
+ 状态码（web状态码）： 1 2 3 4 5  
+ 代码重构  
+ js  
+ 前端优化：雅虎军规   
+ 单例 工厂 代理 设计模式
# spring  
> IOC 控制反转 （默认是非延迟加载）   

    非延迟加载： 加载容器就生成对象   
    优点：一加载容器就可以发现错误，即使不使用这个错误。  
    缺点：影响性能  
    采用延迟加载只有使用对象时才发现错误。   
+ spring 与Struts、Hibernate区别：  
+ spring全用的是反射机制。  
+ spring在配置时，一般配置变化较小的，而不配置变化较大的，比如：实体类
+ ### 在分成services.xml和daos.xml时，需要导入所依赖的xml
+ 此时，主xml文件必须与所依赖的xml文件在同一路径下，当需要导入的xml文件必须要与主xml文件不在同一目录下时，最好不要使用“/”（即相对路径）。  
此时有两种解决方案：
    + 使用../来使其回到父目录下。这样做，创建了一个当前控制台的一个外部依赖文件。
    + 使用绝对路径。
```java
services.xml
<import resource="所依赖的xml"></import>
    <bean>
    ......
    </bean>
daos.xml
    <bean>
    ......
    </bean>
```  
+ 注解配置必须使用服务器来读取（扫描） 所以，在配置时，只需要配置服务器   
 缺点是：侵入代码  
+ IOC注入的两种方式：setter 和 构造方法  
    + setter:就是在xml配置中，使用property来配置，就相当于此时的xml文件映射到了所配属性的setter方法，通过setter方法来注入此时所需要的对象。
    + 构造方法：constructor-arg ref=“所依赖的xml”
```java
xml中：
    <bean id="userController" class="com.java.controller.UserController">
        <constructor-arg ref="userService"/>
        <!--<property name="service" ref="userService"></property>-->
    </bean>  
java代码：
    public UserController(UserService service) {
        this.service = service;
    } 
```
> BeanFactory  
## Bean定义中的属性
 + class：
 + name:alias 重命名
 + scope:
   + singleton 单例模式  构造方法私有化 此时，得到的所有对象都是同一个
   + prototype 原型模式  多个对象 
 + init-method  servlet 初始化的方法 servlet的生命周期 
 + 依赖注入（autowire 注入方式）  
   + no : 默认  不注入 需要有一个参照 (会报错 NullPointerException)
   + byName : 按照名称注入
   + byType : 按照类型注入  
   + constructor : 
+ property 
    + callback : 
+ lazy-init : 延时加载 默认是false
    + 非延迟加载，在初始化容器时，直接就加载对象，优点：可以在容器加载时，同时进行注入，同时可以检查错误  缺点：性能会受影响
    + 延迟加载，需要的时候才会加载对象 优点：性能高 缺点：无法快速的检查错误。
```java
    //依赖注入
    <property id = " ">
        <bean class=" "></bean>
    </property>

    <!--initMethod: 初始化的方法  映射时 自动调用的方法
     scope : 创建一个对象（singleton），还是多个对象(prototype) -->

    <bean id = " " class=" " init-Mtehod=" " scope=" ">

    </bean>
```  
+ depends on :  数据库连接时 依赖 不需要注入 在Dao层直接拿  util中方法只能写成静态的 <b>不能再从容器里面直接拿依赖项的对象</b>

```java
    //dao层
    DBUtils.getConnection();
    //xml中的写法 
    <bean id = " " class=" " depends on=" ">

    </bean>
```
+ ApplicationContext 是BeanFactory的一个子类。  
> 集合的注入   
+ HashMap 底层 红黑树

> 注解配置（必须使用服务器来扫描注解）注解必须放在springMVC使用下