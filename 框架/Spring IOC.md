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

+ Spring声明式事务