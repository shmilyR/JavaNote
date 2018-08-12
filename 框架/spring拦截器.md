# 拦截器
拦截器的主要作用是：拦截用户的请求并进行相应的处理。  
拦截器只拦截controller,不拦截jsp,html.  
# 默认使用Spring的拦截器(AOP的实现)
springMVC中的拦截器，它拦截的目标是：请求的地址。比MethodInterceptor先执行。  
实现有两种方法：实现HandlerInterceptor和继承HandlerInterceptorAdapter类。      
实现思路：继承HandlerInterceptorAdapter并重写其它三个方法  
步骤：  
+ 首先在springmvc.xml中加入自己定义的CommonInterceptor 
+ 实现自定义拦截器的具体逻辑，实现具体的三个方法：
    + preHandle在业务处理器DispatcherServlet处理请求之前被调用；springmvc中的Interceptor是链式调用的，在一个应用中或者说是一个请求中，可以同时存在多个Interceptor。每个Interceptor的调用会依据它的声明顺序依次执行，而且最先执行的都是Interceptor的preHandle方法，所以可以在这个方法中进行一些前置初始化操作或者是对当前请求的一个预处理，也可以在这个方法中进行一些判断来决定请求是否要继续进行下去。该方法的返回值的boolean类型的，当它返回为false时，表示请求结束，后续的Interceptor和Controller都不会再执行；当返回为true时，就会继续调用下一个Interceptor的preHandle方法，如果已经是最后一个Interceptor的时候就会调用当前请求的Controller方法。
    + postHandler在Controller之后的业务处理器DispatcherServlet处理请求完成后，生成视图之前执行，在当前请求进行处理之后，也就是在Controller方法调用之后执行，但是它会在DispatcherServlet进行视图返回渲染之前被调用，所以，我们可以在这个方法中对Controller处理之后的ModelAndView对象进行操作。postHandler方法被调用的方向跟preHandler是相反的，也就是说先声明的Interceptor的postHandler方法反而会后执行。
    + afterCompletion在DispatcherServlet完全处理完请求后被调用，可用于清理资源等。该方法将在整个请求结束之后，也就是在DispatcherServlet渲染了视图之后执行。这个方法的主要作用是用于进行资源清理工作的。
# AOP+自定义注解实现拦截器
是AOP中的拦截器，拦截的目标是方法，并不一定要求是controller中的方法，有两种实现方法：实现MethodInterceptor接口和利用Aspectj的注解或配置。  
实现的原理：
+ 在需要过滤的方法上加自定义的一个注解；
+ 通过对注解获取的方法进行切面编程；
+ 通过RequestContextHolder来获取，request和response
+ 获取session进行判断
# HandlerInterceptor和MethodInterceptor区别：
HandlerInterceptor拦截的是请求地址，所以针对地址做一些验证、预处理等操作比较合适。  
MethodInterceptor用的是AOP的实现机制。用于对普通的方法进行拦截。
# 过滤器
过滤器的运行是依赖于servlet容器的，跟springmvc等框架并没有关系。并且，多个过滤器的执行顺序跟xml中定义的先后顺序有关。  
并不是过滤器完全不能使用spring注入bean,刚开始我认为是因为spring没有加载，之后了解到web应用启动的顺序是：tomcat(服务器)->监听器->过滤器->servlet->拦截器  
+ 首先web容器会去找web.xml文件，读取整个文件，创建一个ServleetContext(上下文)，整个web项目的所有部分共享这个上下文资源。
+ 然后创建监听器，来读取spring中的配置文件(即applicationContext.xml)进行spring容器的初始化；
+ 然后加载过滤器时，若要注入Bean，由于没有bean在servlet才加载，无法获取bean
+ 加载servlet时，springmvc的dispathservlet启动的时候进行spring容器的注入。即读取applocation-service配置，bean会被初始化和注入；
+ 加载拦截器，由于Bean已经加载，故可以在拦截器中使用bean的注入。
# 拦截器和过滤器的区别：
+ 拦截器：基于Java反射机制，属于面向切面编程(AOP)的一种运用，基于web框架的调用。 
+ 主要特点：
    + 可以使用Spring的依赖注入进行一些业务操作，即可以在拦截器内部注入service类(过滤器不可用)，同一个拦截器实例在一个controller生命周期之内可以多次调用。
+ 过滤器：依赖于servlet容器。在实现上基于函数回调
    + 主要特点：对几乎所有的请求有效(这是区别于拦截器重要的一点)
    + 缺点：一个过滤器实例只能在容器初始化时调用一次。
# 区别整理
+ 拦截器是基于Java反射机制的，过滤器是基于函数回调的；
+ 拦截器由spring的标准所支持，过滤器由servlet的标准所支持；
+ 拦截器不依赖于servlet容器，过滤器依赖于servlet容器；
+ 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用；
+ 拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问；
+ 拦截器可以获取IOC容器里的各个bean,而过滤器不行。在拦截器里注入一个service，可以掉用业务逻辑；
+ 拦截器的可用范围更广，过滤器只能在请求的前后起作用，但是拦截器可以在方法前后，抛出异常前后起作用，使用具有更大的弹性。