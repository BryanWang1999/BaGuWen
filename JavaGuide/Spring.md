### 单例Bean线程安全问题

**单例Bean不是线程安全的**

**当多用户同时请求一个服务时，容器会给每一个请求分配一个线程，这时多个线程会并发执行该请求对应的业务逻辑（成员方法），如果该处理逻辑中有对该单例状态的修改，则会出现线程安全问题。**

+ **多例或加锁**
+ **在类中定义⼀个`ThreadLocal`成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中**



### Spring事务实现原理

**`@Transactional`**

**声明式事务管理是建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。**



### Spring 事务失效场景

+ **异常捕获处理**
  + **事务通知只有捕获了目标抛出的异常，才能进行后续的回滚处理，如果目标自己处理掉异常，事务通知无法捕获**
  + **在`catch`块添加 `throw new Runtime Exception(e)` 抛出**
+ **抛出受检异常**
  + **Spring 默认只会回滚非受检异常**
  + **配置rollbackFor属性  `@Transactional(rollbackFor=Exception.class)`**
+ **非public方法导致的事务失效**
  + **Spring 为方法创建代理、添加事务通知、前提条件都是该方法是 public 的**
  + **改为 public 方法**



### Bean 循环依赖

**两个或两个以上的bean互相持有对方,最终形成闭环。比如在创建A对象的同时需要使用的B对象，在创建B对象的同时需要使用到A对象**

**Spring解决循环依赖是通过三级缓存**

<img src="Spring.assets/Screen%20Shot%202023-08-18%20at%202.02.06%20PM.png" alt="Screen Shot 2023-08-18 at 2.02.06 PM" style="zoom:50%;" />

<img src="Spring.assets/Screen%20Shot%202023-08-18%20at%202.06.46%20PM.png" alt="Screen Shot 2023-08-18 at 2.06.46 PM" style="zoom: 50%;" />

**三级缓存 -> 解决代理对象问题**

<img src="Spring.assets/Screen%20Shot%202023-08-18%20at%202.11.26%20PM.png" alt="Screen Shot 2023-08-18 at 2.11.26 PM" style="zoom: 50%;" />







### 构造方法循环依赖问题

**使用`@Lazy`进行延迟加载**

<img src="Spring.assets/Screen%20Shot%202023-08-18%20at%202.13.18%20PM.png" alt="Screen Shot 2023-08-18 at 2.13.18 PM" style="zoom:50%;" />



### BeanDefinition

**Spring容器在进行实例化时，会将xml配置的<bean>的信息封装成一个BeanDefinition对象，Spring根据BeanDefinition来创建Bean对象**



### **Bean 生命周期**

1. **通过`BeanDefinition`获取bean的定义信息**
2. **调用构造函数创建bean的实例**
3. **进行bean的依赖注入，设置bean的属性值**
4. **处理 Aware 接口 (`BeanNameAware`, `BeanFactoryAware`, `ApplicationContextAware`)**
5. **执行后置处理器 `BeanPostProcessor` 的 `postProcessBeforeInitialization()` 方法**
6. **如果bean实现了`InitializingBean`接口或在配置文件中使用了`init-method`属性，Spring容器会调用bean的初始化方法**
7. **执行后置处理器 `BeanPostProcessor` 的 `postProcessAfterInitialization()` 方法(动态代理增强)**
8. **如果bean实现了`DisposableBean`接口或在配置文件中使用了`destroy-method`属性，Spring容器会调用bean的销毁方法**



### Spring 常见注解

```java
@Component @Controller @Service @Repository // 使用在类上用于实例化Bean
@Autowired // 运行时，IOC容器会提供该类型的bean对象，并赋值给该对象。按类型注入
@Qualifier // 当存在多个相同类型的 Bean 时根据名称进行依赖注入
@Scope // 定义 Bean 的作用域
@Configuration // 指定当前类是一个配置类
@ComponentScan // 用于指定 Spring 在初始化容器时要扫描的包
@Bean // 使用在方法上，使用方法返回值来创建bean实例
@Aspect // 用于定义切面类
@Before // 前置通知，此注解标注的通知方法在目标方法前被执行
@After // 后置通知，此注解标注的通知方法在目标方法后被执行
@Around // 环绕通知，此注解标注的通知方法在目标方法前、后都被执行
@Pointcut // 用于定义切点，声明在哪些方法或类上应该应用特定的通知
```



### SpringMVC 常见注解

```java
@RequestMapping // 用于映射请求路径，可以定义在类上和方法上。用于类上，则表示类中的所有的方法都是以该地址作为父路径
@RequestBody // 接收http请求的json数据，将json转换为java对象
@RequestParam // 指定请求参数的名称
@PathVariable // 获取路径参数
@ResponseBody // 将controller方法返回对象转化为json对象响应给客户端
@RestController  // @Controller + @ResponseBody
```



### M**ybatis 执行流程**

1. **加载配置文件 `mybatis-config.xml`，该文件包含数据库连接信息、映射文件等**
1. **创建一个会话工厂 `SqlSessionFactory` 实例**
1. **创建 `SqlSession` 实例，用于与数据库进行交互**
1. **操作数据库的接口，`Executor`执行器，`Executor`执行器负责从映射器 `Mapper` 接口中获取 SQL 语句并执行它们，同时也负责查询缓存的维护。`Executor`接口的执行方法中有一个`MappedStatement`类型的参数，封装了映射信息**
1. **输入参数映射：将Java数据类型转化为数据库支持的类型**
1. **输出结果映射：将数据库支持的类型转化为Java数据类型**



### Mybatis 延迟加载

**延迟加载：在需要用到数据时才进行加载，不需要用到数据时就不加载数据**

**在`Mybatis`配置文件中，可以配置是否启用延迟加载 `lazyLoadingEnabled=truelfalse` ，默认是关闭的**

1. **使用CGLIB创建目标对象的代理对象**
2. **当调用目标方法时，进入拦截器方法，执行sql查询**
3. **获取数据以后，调用set方法设置屬性值，再继续调用目标方法**



### Mybatis 缓存

+ **一级缓存：基于 HashMap 的本地缓存，作用范围是在同一个 `SqlSession` 内。默认打开**

  + **查询结果会被缓存，插入、更新、删除操作会使缓存失效。**
+ **二级缓存：跨**
+ **会话的全局缓存，它可以让多个 `SqlSession` 共享缓存数据。**
  + **查询结果会被缓存，插入、更新、删除操作会使缓存失效。**
  

​	

### **Mybatis 中的 #{} 和 ${}**

+ **`#{}`：将参数值作为预编译参数，可以防止 SQL 注入攻击**
+ **`${}`：将参数的值直接拼接到 SQL 语句中，是一种简单的文本替换，可能存在 SQL 注入的风险**



### 依赖注入 - 属性注入

**在需要注入的Bean对象上加`@Autowired`注解**

**优点：**

+ **使用简单**

**缺点：**

+ **无法注入不可变对象(final 修饰的对象)**
+ **只适用于 IoC 框架，通用性差**
+ **容易引入过多依赖，违反单一职责原则**



### 依赖注入 - `Setter`注入

```java
// Setter 注入
private UserService userService;

@Autowired
public void setUserService(UserService userService) {
		this.userService = userService;
}
```

**优点：**

+ **允许在对象创建后调用 setter 方法更改依赖**



**缺点：**

+ **无法注入不可变对象(final 修饰的对象)**

+ **由于setter方法可以被多次调用，注入的对象可能会被修改**



### 依赖注入 - 构造方法注入

**优点：**

+ **可以注入不可变对象(final 修饰的对象)**
+ **注入的对象不会被修改：因为构造方法只会被执行一次**
+ **通用性更好**



### **final对象能否被注入的原因**

**Java规定被final修饰的属性只能在两个地方进行赋值：**

+ **在定义的时候就进行赋值**
+ **在构造方法内部进行赋值**

