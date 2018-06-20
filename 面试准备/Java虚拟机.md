# ClassLoader  
+ ClassLoader基本概念:   
    Java程序并不是一个可执行文件，而是由许多独立的类文件组成的，每一个文件对应一个Java类。此外，这些类文件并非全部装入内存，而是根据程序需要逐渐载入。
+ ClassLoader加载流程：  
    当运行一个程序时，JVM启动，运行bootstrap classloader(启动类加载器),该ClassLoader加载Java核心API(ExtClassLoader(作用是用来加载Java的扩展API，也就是/lib/ext中的类)和AppClassLoader(用来加载用户机器上CLASSPATH设置目录中的class的，通常在没有指定ClassLoader的情况下，我们自定义的类就有该ClassLoader进行加载)也在此时被加载)，然后调用ExtClassLoader加载扩展API，最后AppClassLoader加载CLASSPATH目录下定义的Class.  
<font color="yellow">一个类加载的过程使用了一种父类委托模式</font>   
原因:(1)避免重复加载
    (2)安全因素  
<font color="purple">ClassLoader是负责执行装入类的相关信息，不负责所有类的所有信息。</font>所以，需要从本地文件系统装入文件。
    
