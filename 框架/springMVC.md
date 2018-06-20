> servlet做控制器开发的问题：  
+ servlet写的多
+ 需要自己手动搜集参数
+ 需要自己进行类型转换    

servlet类的运行环境就是服务器  
之前：页面放在WEBROOT 下 页面 控制器 页面  
springMVC : 为了项目安全，将页面放在WEB-INF下 需要有一个起始控制器，即控制器 页面 控制器 页面
> springMVC环境搭建：  
+ 用一个中央控制器Servlet（只需要这一个servlet） -> 多个action(自己的控制器)  
+ 不用手动搜集参数  
+ 不需要类型转换
    + 导包spring核心6个包 + spring-webmvc,spring-web,spring-aop  
+ servlet作为控制器的缺点：
    + 收集参数过于繁琐；
    + 和容器耦合度过高（request,response）
    + 必须继承HttpRequest,HttpResponse
+ springMVC的控制器就是一个普通方法。
> 请求转发  
  + @Controller表示控制器   
    + 必须要有一个文件来扫描类中的注解。
  + @RequestMapping("访问路径") 代表控制器访问的路径。  页面中 name的值和 类中方法的参数保持一致 必须一致 要不然就无法收集<font color="yellow">在整个类的头部上加@RequestMapping("访问路径")是为了分层，模块化</font>   
1、将核心控制器配置到web.xml中
```xml
<!-- web.xml配置：（加载中央控制器） -->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--容器优先加载 -->
    <load-on-start>1</load-on-start>
</servlet>

<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <!--自己控制器的后缀，自己定义 拦截以.action的结尾的控制 -->
    <url-pattern>*.action</url-pattern>
</servlet-mapping>
<!--通过中央控制器来来加载自己的控制器，必须要有一个文件springmvc-servlet.xml一般与servlet名字相同 -->  
``` 
```xml
<!-- springmvc-servlet.xml的配置 -->
<!-- 扫描包下的所有控制器 -->
<!--springMVC默认的配置文件名 springmvc-servlet 改名字的话，需要如下配置：
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>servlet配置文件位置</param-value>
    </init-param>
-->
<context:component-scan base-package="com.java.controller"/>
 <!--配置页面中项目的访问路径 prefix为前缀 suffix为后缀 在这里，是“转发”的操作-->
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
```
+ 解决springMVC乱码问题(配过滤器)：  
```xml
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>
    org.springframework.web.filter.CharacterEncodingFilter
    </filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/</url-pattern>
</filter-mapping>
```  
+ 字符串自动转换：  
@ModelAttribute("user")User user
+ springMVC可以按照对象来收集参数，页面的name名字必须和实体类中的对象名一致。
+ 控制器只是根据页面传来的数据调用service层方法返回的值跳转不同的页面。  
+ equal和“==”比较数据的区别：
+ @Service 代表服务层的注解 
+ 在Controller层中@Autowired  代表自动注入 <font color="yellow">应该写在set方法上，因为自动注入是通过调用set方法来实现注入，而且自动注入需要配置xml文件来扫描自动注入的类  写在构造方法上，表示构造注入</font> 
+ 通过监听器来加载spring配置文件
```xml
 <!--监听加载spring的配置文件 -->
<listener>
    <listener-class>
    org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
<!--告诉监听器 spring配置文件的路径 -->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/service.xml</param-value>
</context-param>
```
+ 扫描自动注入的类
```xml
<!-- 需要的包xmlns:context="http://www.springframework.org/schema/context"
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context.xsd"> -->
<!--服务器扫描自动注入的类 -->
<context:component-scan base-package="com.java.service"/>
<context:component-scan base-package="com.java.dao"/>
```
+ 必须在web.xml中加配置（用监听器来加载配置文件,必须告诉监听器，spring配置文件的路径<content-param来配置）
+ @Repository 代表dao层
+ @Componet 代表一个bean 但是没有service、repository更能直观的表示出层级结构。(通用型组件)
+ @Required 也是在setter上用 但必须给bean的一个属性赋值。（检查特定的属性是否设置，若未设置，会抛出一个bean初始化异常）
```java
xml配置：
 <bean id="userDao" class="com.java.vo.User">
        <property name="username" value="李四"/>
    </bean>
    <bean id="user" class="com.java.dao.UserDao">
        <property name="user" ref="userDao"/>
    </bean>
java代码：
private User user;

public User getUser() {
    return user;
}
@Required
public void setUser(User user) {
    this.user = user;
}
```
+ @Qualifier("配置文件中对应的bean的id名") 按类型注入
+ @Resource 按名称注入  （name="" 代表按类型注入）最好写在setter方法上。  
+ @PreDestroy  销毁，服务器销毁时调用的方法 
+ @PostConstruct 初始化，服务器启动时调用的方法 
+ @PreDestroy与@PostConstruct与servlet的生命周期有关。PostConstruct在构造函数之后，init()方法之后执行，PreDestroy在destroy之前执行。
+ 初始化或者销毁时，会出现多次的原因：配置文件中注入了一次，加载配置文件时，有多少个对象，此方法被调用多少次。 
+ @GetMapping():get方法提交
+ @PostMapping():post方法提交  
+ @RequestParam String username   
 白盒测试 测试功能  
 黑盒测试 保证每个分支都要走到   
 <font color="yellow">表单提交时，只有post方法能走过滤器。尽量少用get提交。</font>  
 单向数据绑定：提交到完成页面，就不可再改变，若想改变，就必须重新提交数据  
 双向数据绑定MVVM  
 分布式：向多个服务器发出请求。
> 重定向    

重定向只能用get提交方式
```java
return "redirect:/控制器的路径";
```
> session  
+ @SessionAttributes("存入session中的名字"):注意：控制器中，有@SessionAttributes("存入session中的名字")时，一定要有model。
+ @SessionAttribute()作用在类上  
+ 一般是在重定向，需要传递参数时使用。
> 拦截器(AOP的实现)  
+ 要么继承一个类，要么实现一个接口(HandlerInterceptor)  
    jdk1.8新特性：接口中可以写普通方法，只需要在普通方法前加default
+ 加入配置文件(加入到springmvc的配置文件中)
```xml
<mvc:interceptors>
    <mvc:interceptor>
        <!--拦截地址栏路径 path可以是拦截全部的/** 也可以是拦截指定控制器下的,在指定控制器的类名上加@RequestMapping("")表示本类的访问路径，一般在类名上加此注解，是为了区分同一包下、不同类的相同访问路径名-->
        <mvc:mapping path = ""></mvc:mapping>
        <!--用哪个类进行拦截 -->
        <bean class="" ></bea>
    </mvc:interceptor>
</mvc:interceptors>
```
+ 动态可插拔的组件，使用不会对自己的类和代码有任何影响。(AOP 面向切面编程 可配置的 动态的) 






  

