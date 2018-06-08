+ servlet做控制器开发的问题：
    + servlet写的多
    + 需要自己手动搜集参数
    + 需要自己进行类型转换    

servlet类的运行环境就是服务器  
之前：页面放在WEBROOT 下 页面 控制器 页面  
springMVC : 为了项目安全，将页面放在WEB-INF下 需要有一个起始控制器，即控制器 页面 控制器 页面
> springMVC环境搭建：  
+ 用一个中央控制器Servlet -> 多个action(自己的控制器)  
+ 不用手动搜集参数  
+ 不需要类型转换
    + 导包spring核心6个包 + spring-webmvc,spring-web,spring-aop  