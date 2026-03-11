## Spring

### Spring中的事务是如何实现的？

Spring实现事务的本质是利用AOP完成的。它对方法前后进行拦截，在执行方法前开启事务，在执行完目标方法后根据执行情况提交或回滚事务。

### Spring中事务失效的场景有哪些？

在项目中，我遇到过几种导致事务失效的场景：

1. 如果方法内部捕获并处理了异常，没有将异常抛出，会导致事务失效。因此，处理异常后应该确保异常能够被抛出。
2. 如果方法抛出检查型异常（checked exception），并且没有在`@Transactional`注解上配置`rollbackFor`属性为`Exception`，那么异常发生时事务可能不会回滚。
3. 如果事务注解的方法不是公开（public）修饰的，也可能导致事务失效。

### Spring、SpringMVC、SpringBoot有什么区别？

 Spring 是一个基础框架，核心功能是 IoC 和 AOP，用来管理对象和依赖关系。
 SpringMVC 是 Spring 的 Web 模块，用于处理 HTTP 请求，实现 MVC 架构。
 SpringBoot 是对 Spring 的封装，通过自动配置和 starter 机制简化 Spring 项目的搭建，让开发者可以快速启动项目。

### SpringMVC的执行流程？ 

SpringMVC 的执行流程是：首先客户端请求会进入 DispatcherServlet 前端控制器，然后 DispatcherServlet 会通过 HandlerMapping 查找对应的 Controller 处理方法，接着通过 HandlerAdapter 调用 Controller 方法处理请求。Controller 处理完成后返回 ModelAndView，DispatcherServlet 再通过 ViewResolver 解析视图，最终将结果返回给客户端。

### Springboot自动配置原理？

SpringBoot 自动配置是通过 `@EnableAutoConfiguration` 实现的。启动时该注解会通过 `AutoConfigurationImportSelector` 加载 `METAINF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件中的自动配置类。每个自动配置类会通过 `@Conditional` 条件判断是否生效，如果满足条件就会自动创建相关 Bean，从而完成自动配置。

### SpringBoot Starter 机制是什么？

把一组常用依赖打包在一起，让开发者只需要引入一个依赖就能使用某个功能。

### 为什么 SpringBoot 只引入一个 starter 就能启动 Web 项目？

因为 starter 会引入 Web 开发所需的一组依赖，而 SpringBoot 启动时通过 `@EnableAutoConfiguration` 加载自动配置类，这些自动配置类会根据条件创建 DispatcherServlet、Tomcat、HandlerMapping 等核心组件，从而自动完成 Web 环境的搭建。

### Spring IOC 依赖注入方式

- 构造函数注入
- Setter方法注入
- 字段注入（@Autowired 或 @Resource）

### Spring 依赖注入实现原理

Spring 的依赖注入是由 IoC 容器完成的。
 在 Spring 启动时，容器会先扫描组件（Component）并解析成 **BeanDefinition**，把 Bean 的元数据注册到 **BeanFactory** 中。

当容器创建 Bean 时，会先进行 **实例化**，然后在 **属性填充阶段**解析 `@Autowired`、`@Resource` 等依赖注入注解。

Spring 通过 **BeanPostProcessor（AutowiredAnnotationBeanPostProcessor）** 扫描这些注解，然后从容器中查找匹配的 Bean，最后通过 **反射**把依赖对象注入到目标 Bean 中。

如果出现循环依赖，Spring 会通过 **三级缓存提前暴露 Bean** 来解决。

### Spring中bean的生命周期

Spring中bean的生命周期包括以下步骤：

1. 通过`BeanDefinition`获取bean的定义信息。
2. 调用构造函数实例化bean。
3. 进行bean的依赖注入，例如通过setter方法或`@Autowired`注解。
4. 处理实现了`Aware`接口的bean。Aware接口的作用就是让bean能拿到Spring容器里的对象，因为bean本身是不知道Spring容器存在的。
5. 执行`BeanPostProcessor`的前置处理器。 **BeanPostProcessor = Bean加工厂。Spring在Bean初始化前后，允许你对Bean做处理。**
6. 调用初始化方法，如实现了`InitializingBean`接口或自定义的`init-method`。 **这一步就是在做一些准备工作。**
7. 执行`BeanPostProcessor`的后置处理器，可能在这里产生代理对象。
8. 最后是销毁bean。

### Spring中的循环引用？

1. 循环依赖就是：

   > Bean A 依赖 Bean B，同时 Bean B 又依赖 Bean A

2. Spring 对 **单例 Bean（singleton）** 有一个巧妙的解决办法：

​	它使用 **三级缓存**（三级 Map）：

| 缓存                          | 存的是什么          | 用途                         |
| ----------------------------- | ------------------- | ---------------------------- |
| 一级：`singletonObjects`      | 完全初始化好的 Bean | 正常使用                     |
| 二级：`earlySingletonObjects` | 早期 Bean           | 半成品，用于循环依赖         |
| 三级：`singletonFactories`    | ObjectFactory       | 提前生成 Bean，可做 AOP 代理 |

三级缓存生成的半成品对象会放到二级缓存中

#### 解决流程（最直观）

假设有 A 和 B 循环依赖：

1️⃣ Spring 开始创建 A → 实例化 A（构造函数执行）

2️⃣ A 依赖 B → 开始创建 B → 实例化 B

3️⃣ B 依赖 A → Spring 去 **三级缓存**找 A

4️⃣ 在三级缓存里有 **ObjectFactory** → 调用生成 A 的引用

5️⃣ B 拿到 A 的引用 → 属性注入完成

6️⃣ 回到 A → 注入 B → 完成依赖

7️⃣ 执行初始化方法 → Bean 可以使用

✅ 循环依赖解决

三级缓存只有在 Bean 已经开始创建时才会放入 ObjectFactory。当 Spring 开始创建 A 时，**B 还没创建**，所以 B 的 ObjectFactory 还没放入三级缓存，所以 A 拿不到 B 的引用 → 必须先去容器创建 B

### 构造方法出现了循环依赖怎么解决？

Spring 的循环依赖解决机制只支持 **单例 Bean 的字段注入或 setter 注入**。如果是 **构造方法注入**，因为创建对象时必须先获取依赖，而此时 Bean 还没有实例化，无法提前暴露对象，所以 Spring 无法解决，会抛出异常。