spring-core：IoC、DI

（1）包含框架基本的核心工具类，其它组件都要使用到这个包里的类。
（2）定义并提供资源的访问方式。

spring-beans：Spring主要面向Bean编程（BOP）、BeanFactory

（1）Bean的定义。
（2）Bean的解析。
（3）Bean的创建。

spring-context：ApplicationContext

（1）为Spring提供运行时环境，保存对象的状态。
（2）扩展了BeanFactory。

spring-aop：最小化的动态代理实现

（1）JDK动态代理。
（2）Cglib
（3）只能使用运行时织入，仅支持方法级编织，仅支持方法执行切入点

spring-aspectj + spring-instrument：Full AspectJ

Spring中AOP织入的三种方式：

（1）编译期织入。
（2）类加载期织入。
（3）运行期织入。

| Spring AOP | AspectJ |
| :-: | :-: |
| 在纯Java中实现 | 使用Java编程语言的扩展实现 |
| 不需要单独的编译过程 | 除非设置LTW，否则需要AspectJ编译期（ajc） |
| 只能使用运行时织入 | 运行时织入不可用。支持编译时、编译后和加载时织入 |
| 功能不强-仅支持方法级编织 | 更强大-可以编织字段、方法、构造函数、静态初始值设定项、最终类/方法等…… |
| 只能在由Spring容器管理的bean上实现 | 可以在所有域对象上实现 |
| 仅支持方法执行切入点 | 支持所有切入点 |
| 代理是由目标对象创建的，并且切面应用在这些代理上 | 在执行应用程序之前（在运行时）前，各方面直接在代码中进行织入 |
| 比AspectJ慢多了 | 更好的性能 |
| 易于学习和应用 | 相对于Spring AOP来说更复杂 |
