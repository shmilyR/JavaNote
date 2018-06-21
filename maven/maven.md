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
    + maven安装的前提条件：Maven是Java开发的，它的运行以来JDK。
        + bin 可执行的脚本命令
        + conf 配置文件
        + lib maven运行所需要的jar包
    + 配置环境变量
    + 配置本地仓库（jar包）
        + maven的仓库类型：

+ maven项目标准目录结构
+ maven常用命令
    + mvn -v 查看maven版本
+ maven整合web项目