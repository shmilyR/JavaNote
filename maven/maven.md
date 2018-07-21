# maven
+ Apache公司开源项目，是项目构建工具。用来管理依赖。
+ 把面向对象的思想进一步用在了两个工程当中。（继承思想）
## <font color="blue">maven基础</font>
+ maven的好处  
相对于传统项目实现方式，maven实现，所占空间较小。
可以初步推断：<font color="yellow">maven不需要jar包，只需要jar包的坐标信息</font>
jar包放在jar包仓库中。  
jar包的坐标：公司名称+项目名称+版本信息
maven项目中需要某个jar包，只需要在Maven项目中配置需要jar包坐标信息。maven程序就会根据jar包的坐标信息去jar包仓库（maven仓库）中查找jar包。
+ maven的好处如何实现   
    + <font color="red" size="4">maven的两大核心：</font>
        + 依赖管理：就是对jar包的管理过程。
        + 项目构建：项目编码完成后，对项目进行编译、测试、打包、部署一系列操作都可以通过<font color="red">命令</font>来实现。
    + 使用maven发布项目到tomcat:
        + 到项目根路径
        + 执行命令：mvn tomcat:run
+ maven的安装、配置本地仓库
    + maven安装的前提条件：Maven是Java开发的，它的运行依赖JDK。
        + bin 可执行的脚本命令
        + conf 配置文件
        + lib maven运行所需要的jar包
    + 配置环境变量
    + 配置本地仓库（jar包）
        + maven的仓库类型：
            + 本地仓库：个人计算机上。
            + 私服：存在于本地的局域网内一台服务器，存jar包  
            本地仓库连接私服仓库的前提：安装了私服
            + 中央仓库：在互联网上。存放所有开源jar包。由maven团队维护。（注意：由于版权问题，没有Oracle的jar包）
            连接到中央仓库的前提：服务器在美国，需要外网。
    + 配置
        + 找到jar包仓库压缩包；
        + 解压到本地磁盘；
        + 配置本地仓库： 
         目的：让maven知道jar包在哪儿
+ maven项目标准目录结构
    + 与传统项目的区别：  
    对项目中的文件进行了细分。
    + src:项目源码
        + main:主要代码
            + java:Java类
            + resource:配置文件
            + webapp:页面的素材 jsp js等
        + test:单元测试代码
            + java:单元测试类
            + resource:配置文件（一般不会存入）
    + pom.xml:maven项目的核心配置文件（每个maven项目都需要有）
    ```xml
    <!--pom.xml文件例子-->
    <project            xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.companyname.project-group</groupId>
    <artifactId>project</artifactId>
    <version>1.0</version>
    </project>
    ```
    + 所有的 POM 文件需要 project 元素和三个必  须的字段：groupId, artifactId,version。
    + 在仓库中的工程标识为 groupId:artifactId:version
        + groupId:工程组的标识，在一个工程组或项目中是唯一的
        + artifactId:工程标识，通常是工程名称
        + version:工程版本号。在 artifact 的仓库中，它用来区分不同的版本
    + super pom:所有的pom都继承自一个父pom,包含了一些可以被继承的默认设置。
    + Maven 使用 effective pom（Super pom 加上工程自己的配置）来执行相关的目标
+ maven常用命令（用于项目构建）
    + mvn -v 查看maven版本
    + mvn clean：清理 清理掉项目中的target目录，即删掉.class文件
    + mvn complie:编译 将项目中的.java文件编译为.class文件
    + mvn test:执行单元测试 执行项目根目录下 src/test/java   
    单元测试的类名要求：xxxxTest.java
    + mvn package:打包  打包时，会对项目进行编译、测试 打包成war包 打包成jar包还是war包取决于项目是java工程还是web工程 打包到项目根目录下target目录
        + jar包：java工程
        + war包：web工程
    + mvn install:安装 本地多个项目公用一个jar包。会进行编译、测试、打包等操作，然后安装到本地仓库。
    + mvn help:effective-pom 查看super pom默认配置
+ maven项目的生命周期：构建生命周期是一组阶段的序列（sequence of phases），每个阶段定义了目标被执行的顺序。这里的阶段是生命周期的一部分。  
    在maven中存在三套生命周期，每一套生命周期都是相互独立的，互不影响
    <font color="red"><b><i>在一套生命周期内，在执行后面的操作时，会默认执行前面的操作。</b></i></font>
        + cleanLifeCycle:清理生命周期 clean
        + defaultLifeCycle:默认的生命周期 compile test package install deploy
        + siteLifeCycle:站点生命周期：site (会生成一个html文档，来描述项目的一些信息)
+ maven整合web项目
    + maven整合servlet（创建一个webapp工程）
        + 在使用ideal创建servlet时，发现没有servlet选项，解决方案如下：在pom.xml中引入servlet的jar包
        ```java
        <!-- https://mvnrepository.com/artifact/javax.servlet/javax.servlet-api -->  
        <dependency>  
            <groupId>javax.servlet</groupId>  
            <artifactId>javax.servlet-api</artifactId>  
            <version>3.1.0</version>  
        </dependency>  
        <!-- https://mvnrepository.com/artifact/jstl/jstl -->  
        <dependency>  
            <groupId>jstl</groupId>  
            <artifactId>jstl</artifactId>  
            <version>1.2</version>  
        </dependency> 
        ```
        + 查找依赖（jar包）maven项目不再拷贝jar包，jar包都从本地仓库中来。
        + ideal在pom.xml中查找dependency快捷键:alt+insert之后选择Dependency
        + 依赖的范围：
        <table border="2" align="center">
           <tr>
            <td>依赖范围</td>
            <td>对于编译classpath有效</td>
            <td>对于测试classpath有效</td>
            <td>对于运行classpath有效</td>
            <td>例子</td>
           </tr>
           <tr>
            <td align="center">compile</td>
            <td align="center">Y</td>
            <td align="center">Y</td>
            <td align="center">Y</td>
            <td align="center">spring-core</td>
           </tr>
            <tr>
            <td align="center">test</td>
            <td align="center">-</td>
            <td align="center">Y</td>
            <td align="center">-</td>
            <td align="center">JUnit</td>
           </tr> <tr>
            <td align="center">provided</td>
            <td align="center">Y</td>
            <td align="center">Y</td>
            <td align="center">-</td>
            <td align="center">servlet-api</td>
           </tr> <tr>
            <td align="center">runtime</td>
            <td align="center">-</td>
            <td align="center">Y</td>
            <td align="center">Y</td>
            <td align="center">JDBC驱动</td>
           </tr> 
           <tr>
            <td align="center">system</td>
            <td align="center">Y</td>
            <td align="center">Y</td>
            <td align="center">-</td>
            <td align="center">本地的maven仓库之外的类库</td>
           </tr>
        </table>
    idea中修改依赖范围：
    ```xml
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
        <!--test开发时需要，上线时，不需要 -->
    </dependency>
    ```
    compile:默认依赖范围
    <font color="red">如果将部署到tomcat之后就不再需要的jar包依赖范围设置为默认范围，即compile，打包后包含此jar包，war包部署到tomcat跟tomcat中存在的jar冲突，将会出现<font color="yellow">jar包冲突错误</font>导致代码运行失败。</font><br/>
    provided:表示运行部署到tomcat之后将不再需要此jar包 tomcat自带的jar包(jsp servlet)一定要设置为provided
    test:只在测试时使用
    runtime:
+ 运行调试maven项目:
    + 当前maven能够集成的tomcat只有tomcat-maven-plugin和tomcat7-maven-plugin
+ maven的概念模型(两大核心)：
    + 依赖管理：对jar包的管理
    + 项目构建：通过命令来构建。
## <font color="blue">maven应用</font> 
+ 传递依赖冲突解决
    + 什么是传递依赖：  
    a依赖b,b依赖c，c则为a的传递依赖，b为a的直接依赖。  
    冲突：两个项目同时依赖于同一个jar,版本重复。
    + maven的自主调解原则：
        + 第一声明者优先原则:先定义的，就用此传递依赖
        + 路径近者优先原则:直接依赖级别高于传递依赖。
+ maven整合框架(spring + springMVC + mybatis)  
     面向接口开发，只需要写一个接口和方法 通过接口中的方法来找到mapper中的SQL语句。
    + 导包 
        + spring 核心包(5个 + AOP) + springMVC(web webmvc) 
        + spring-jbdc.jar包
        + mybatis
        + mybatis-spring 框架整合包
        + spring-tx 事务管理包
        + 数据库包 驱动
        + 日志(commons-logging4j与jlf4j(格式化日志的)使用上的区别)
        + 第三方的数据源 druid(阿里的)
        + springAOP必须有AOP包的支持(两个包 aspectjrt aspectjweaver(植入))
    + 核心文件配置
        + lo4j.properties
        ```properties
        log4j.rootLogger=debug,stdout
        log4j.appender.stdout=org.apache.log4j.ConsoleAppender
        log4j.appender.stdout.layout=org.apache.log4j.SimpleLayout
        log4j.logger.com.ibatis=DEBUG
        log4j.logger.com.ibatis.common.jdbc.SimpleDataSource=DEBUG
        log4j.logger.com.ibatis.common.jdbc.ScriptRunner=DEBUG
        log4j.logger.com.ibatis.sqlmap.engine.impl.SqlMapClientDelegate=DEBUG
        log4j.logger.java.sql.Connection=DEBUG
        log4j.logger.java.sql.Statement=DEBUG
        log4j.logger.java.sql.PreparedStatement=DEBUG
        ```
        + jdbc.properties
        ```properties
        jdbc.driverClassName=com.mysql.jdbc.Driver
        jdbc.url=jdbc:mysql://127.0.0.1:3306/day0408
        jdbc.username=root
        jdbc.password=lr123456789
        ```
        + applicationContext.xml(spring)
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans      xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
            <!--用spring核心配置文件来读取spring-mybatis和spring-tx.xml两个配置文件，注意次序-->
            <import resource="spring-mybatis.xml"/>
            <import resource="spring-tx.xml"/>
        </beans>
        ```
        + spring-mybatis.xml(spring和mybatis进行整合，数据库配置)
        ```xml
        <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns="http://www.springframework.org/schema/beans"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
            <!--读取jdbc.properties文件的配置信息 classpath代表类路径 -->
            <context:property-placeholder location="classpath:jdbc.properties"/>
            <!--往第三方数据源上设置 druid database 获取数据库连接 数据库连接不能关闭-->
            <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
            destroy-method="close" init-method="init">
                <!--根据需要灵活配置， 驱动类 url username password必须 -->
                <property name="driverClassName" value="${jdbc.driverClassName}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </bean>
            <!--配置数据源是为了往mybatis上注入 -->
            <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
                <!--sqlSessionFactory获取数据库连接 -->
                <property name="dataSource" ref="dataSource"/>
                <!-- 自动扫描Mapper配置文件-->
                <property name="mapperLocations" value="classpath:com/java/mapper/UserMapper/*Mapper.xml"/>
                <!--一般不写mybatis的配置文件 -->
            </bean>
                <!--配置自动查到Dao包下的类 -->
            <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
                <!--查找自己写的dao层 -->
                <property name="basePackage" value="com.java.dao"/>
                <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
            </bean>
        </beans>
        ```
        + spring-tx.xml(对dao层方式的事务进行配置 AOP)
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">
        <!--开启事务注解扫描 -->
        <aop:aspectj-autoproxy proxy-target-class="true"/>
        <!--spring jdbc事务管理器 读文件时才会注入 1、main方法直接读 2、服务器扫描注解-->
        <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"/>
        </bean>
        <!--事务增强通知 将事务加入指定的方法 isolation:事务的隔离级别-->
        <tx:advice id="txAdvice" transaction-manager="transactionManager">
            <tx:attributes>
                <tx:method name="add*" isolation="DEFAULT" propagation="REQUIRED" read-only="false" rollback-for="Exception"/>
                <tx:method name="delete*" isolation="DEFAULT" propagation="REQUIRED" read-only="false" rollback-for="Exception"/>
            </tx:attributes>
        </tx:advice>
        <!--用AOP将通知切入方法 切入点  1、execution(* ) 2
        within(*) PointCut 将事务切入指定的方法 第一个*代表任意的返回值 第二个代表此包下任意的类
        第三个*代表任意类中任意的方法 括号里的(..)代表方法的任意参数  within(com.java.dao..*)-->
        <aop:config>
            <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.java.dao.*.*(..))"/>
            </aop:config>
        </beans>
        ```
        + spring-mvc.xml(springmvc的配置文件)
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
        <context:component-scan base-package="com.java.*"/>
        <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/"/>
        <property name="suffix" value=".jsp"/>
        </bean>
        </beans>
        ```
        + mybatis-config.xml(可以省略掉)
        此处已省略，在整合时，可以不用写
        + web.xml
        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
        <display-name>Archetype Created Web Application</display-name>
        <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
        <!--告诉监听器 spring配置文件的位置 -->
         <context-param>
         <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
        </context-param>
        <welcome-file-list>
         <welcome-file>HomeController.action</welcome-file>
        </welcome-file-list>
         <!--配置中央控制器 -->
        <servlet>
         <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
        </servlet>
        <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>*.action</url-pattern>
        </servlet-mapping>
        <!--解决中文乱码问题 -->
        <filter>
            <filter-name>encodingFilter</filter-name>
            <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceRequestEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
        </filter>
        <filter-mapping>
            <filter-name>encodingFilter</filter-name>
            <url-pattern>/*</url-pattern>
        </filter-mapping>
        </web-app>
        ```
+ maven对项目进行拆分、聚合
+ 私服应用

