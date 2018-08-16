+ 使用IOC的原因：
    + 对象创建同一托管
    + 规范的生命周期管理
    + 灵活的依赖注入
    + 一致的获取对象方式
+ Spring-IOC注入方式和场景
<table border="2">
    <tr>
        <th align="center">XML</th>
        <th align="center">注解</th>
        <th align="center">Java配置类</th>
    </tr>
    <tr>
        <td align="center">Bean实现类来自第三方类库，如：DataSource等；需要命名空间配置,如：context,aop,mvc等</td>
         <td align="center">项目中自身开发使用的类，可直接在代码中使用注解</td>
          <td align="center">需要通过代码控制对象创建逻辑的场景，如：自定义修改依赖类库</td>
    </tr>
</table>

# 什么是Spring?
Spring是一个轻量级开源框架，Spring的核心是控制反转(IOC)和面向切面(AOP).
# Spring IOC
+ 概念：IOC就是控制反转，意思为将控制权反转到IOC容器的手中。所谓控制权，就是对象的创建以及对象之间依赖关系注入的权利。
+ 实现原理：(IOC容器和配置文件以及标准)  
控制反转我们需要IOC容器，配置文件来让容器知道需要创建的对象和对象之间的关系并且使用统一的标准和语法来解析。  
    + IOC容器：
        + IOC的体系架构：(BeanFactory、BeanDefinition)
            + BeanFactory:基本的IOC容器接口，定义了IOC容器的基本功能规范，定义bean对象及其相互的关系。
            + BeanDefintion:描述bean对象在spring的实现，bean的抽象数据结构，它包括属性参数、构造器参数以及其它具体的参数。
        + IOC容器的初始化步骤：BeanDefinition的Resource定位，载入和注册的基本过程。
        + 具体的IOC如何实现？(BeanFactory、AppkicationContext) 
            + BeanFactory(BeanFactory接口定义了容器的基本行为)  
            XMLBeanFactory实现BeanFactory接口，从名字上看就可以知道它是用来读取XML格式的BeanDefinition,BeanDefinition是对对象依赖关系管理的核心数据结构。就是那个pojo类在IOC容器中的抽象。但它读取的内部实现是由相应的Reader对象负责BeanDefinition的读入，在读入的时候，需要用资源的位置初始化一个Resource对象，这个对象就包含了BeanDefinition的路径信息。然后Reader对象会调用loadBeanDefinition()方法，参数就是Resource对象，对相应的BeanDefinition进行加载和注册。
            + Application:他除了xmlBeanFactory的功能外，
                + 提供了支持国际化的文本消息；
                + 统一的资源文件读取方式；
                + 已在监听器中注册的bean的事件，事件机制。  
                事件机制应用的是观察者模式，其中的观察者就是各个Bean（实现ApplicationListener接口），被观察者就是ApplicationContext(继承ApplicationEvent类)。每当被观察者发布一个新的事件时，就会触发通知观察者进行相应的动作。然后就是ApplicationContext的初始化：
                    + 入口就是refresh()方法，若之前有容器就关掉或者毁掉重新开启一个IOC容器。
                    + IOC容器的创建：(创建两个DefaultLisableFactory（两个工作）)
                        + 调用父类容器的构造方法为容器设置好bean资源加载器super(parent),最终调用的是父类的父类来创建applicationContext，这个父类中的构造方法中设置spring source加载器保证applicationContext有统一的资源文件读取方式。
                        + 调用父类的设置bean定义资源文件的定位路径的方法。setConfigLocations(configLocations);
                    + BeanDefinition的载入和注册：
                        + 载入：通过封装BeanDefinition的Resource对象，获取IO流，获得相应的document对象，document对象就是一颗文档树，按照Spring中Bean的规则对文档树进行解析，得到的结果由BeanDefinitionHolder持有。
                        + 注册：IOC容器底部的数据结构是HashMap表，将BeanDefinition添加到HashMap中。
+ 依赖注入如何实现？在什么时候触发？
    + 何时依赖注入？
        + 用户第一次通过getBean方法或autowire索要Bean时，IOC容器触发依赖注入；
        + 当用户在Bean定义资源中\<bean>元素配置了$lazy-init$属性，即让容器在解析注册Bean定义时进行预实例化，触发依赖注入。
    + 怎样依赖注入：
        + 第一种情况：根据类的name寻找  
            + 先判断是否是单例对象，是的话创建出对象并添加进缓存；
            + 在当前BeanFactory中获取，获取不到的情况下交由父类BeanFactory进行加载，当都无法获取到时，报错。
            + 当获取到对应的BeanDefinition时，调用一个重要的方法getDependsOn()，获取所有依赖的BeanDefinition名字，然后递归调用getBean()方法进行获取，直到获取到一个不依赖其它BeanDefinition的BeamDefinition为止。  
            <font color="red">注意：可能出现循环依赖：构造循环依赖、属性循环依赖。</font>就是a正在实例化一半，a里面需要b,九八实例化一半的a放在未初始化对象的集合中，去实例化b,b里也要a,然后b发现a在未初始化对象的集合中，然后就是循环依赖，抛出异常。
            + 根据创建BeanDefinition创建对象时，会根据是否实现了接口选择不同的创建方式，实现了JDK，没实现cglib.
        + 第二种情况：对书香进行依赖注入时：根据BeanDefinition中的property属性，进行解析，比如Array、List标签。然后递归的调用了getBean()方法及逆行属性获取并依赖注入。
# Bean的声明周期
在一个bean实例被初始化时，需要执行一系列的初始化操作以达到可用的状态。同样的，当一个bean不再被调用时需要进行相关的析构操作，并从bean容器中溢出。  
Spring bean factory负责管理在spring容器中被创建的bean的生命周期。Bean的生命周期由两组回调方法组成
+ 初始化之后调用的回调方法；
+ 销毁之前调用的回调方法。
跟对象的生命周期实质上是一致的，大概分为三个阶段：
1. 实例化(为bean对象开辟空间)；
2. 初始化(对对象的属性进行依赖注入，使用setter方法)
3. 销毁  
具体来讲分为以下：
+ 实例化一个Bean,也就是我们通常所说的new,默认为单例；
+ 按照Spring上下文对实例化的Bean进行配置，也就是IOC注入；
+ 如果这个Bean实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的就是Spring配置文件中Bean的ID；
+ 如果这个Bean实现了BeanFactoryAware接口，会调用它实现的setBeanFactory().传递的是Spring工厂本身(可以用这个方法来获取到其它的Bean)
+ 如果这个Bean实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文，该方式同样可以实现前一个步骤，但比前一个好，因为ApplicationContext是BeanFactory的子接口，有更多的实现方法；
+ 如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitalization(Object obj,String s)方法，BeanPostProcessor经常被用作是Bean内容的更改，并且由于这个是在Bean初始化结束时调用After方法，也可用于内存或缓存技术；
+ 如果这个Bean在Spring配置文件中配置了$init-method$属性会自动调用其配置的初始化方法；
+ 如果这个Bean关联了BeanPostProcessor接口，将会调用postAfterInitialization(Object obj,String s)方法。以上这些工作完成之后，就可以用这个Bean了，那这个Bean是一个single的，所以一般情况下沃恩调用同一个ID的Bean会是在内容地址相同的实例。
+ 当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean接口，会调用其实现的destory方法。
+ 最后，如果这个Bean的spring配置中配置了$destory-method$属性，会自动调用其配置的销毁方法。
# IOC的好处
+ 对象创建统一管理；
+ 规范的对象生命周期管理；
+ 获取一致的对象(单例模式))；
+ 灵活的依赖注入
# Spring装配Bean的过程
+ 实例化；
+ 设置属性值；
+ 如果实现了BeanNameAware接口，调用setBeanName设置Bean的ID或者Name;
+ 如果实现BeanFactoryAware接口,调用setBeanFactory 设置BeanFactory;
+ 如果实现ApplicationContextAware,调用setApplicationContext设置ApplicationContext
+ 调用BeanPostProcessor的预先初始化方法;
+ 调用InitializingBean的afterPropertiesSet()方法;
+ 调用定制init­method方法；
+ 调用BeanPostProcessor的后初始化方法;
# Spring容器关闭的过程
+ 调用DisposeableBean的destory();
+ 调用定制的destory-method方法。
