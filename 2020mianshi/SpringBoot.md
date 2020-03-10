# SpringBoot



## 1. Spring基础

### 1.1 Spring概述

#### 1.1.1 Spring简史

第一阶段：xml配置，Spring 1.x时代

第二阶段：注解配置，Spring 2.x时代，随着jdk1.5带来的注解支持，Spring提供了声明Bean注解，大大减少了配置。我们一般引用的基本配置用**xml**，业务配置用**注解**。

第三阶段：从Spring 3.x到现在，Spring提供了Java配置的能力。Spring 4.x和SpringBoot都推荐使用Java配置。



#### 1.1.2 Spring的模块

* （1）核心容器（Core Container）

  Spring-Core：核心工具类,Spring 其他模块大量使用 Spring-Core；
  Spring-Beans：Spring 定义 Bean 的支持；
  Spring-Context：运行时 Spring 容器；
  Spring-Context-Support：Spring 容器对第三方包的集成支持；
  Spring-Expression：使用表达式语言在运行时查询和操作对象；

* （2）AOP

  Spring-AOP：基于代理的AOP支持;

  Spring-Aspects：基于AspectJ的AOP支持；

* （3）消息（Messaging）

  Spring-Messaging：对消息架构和协议的支持；

* （4）Web

  Spring-Web：提供基础的 Web 集成的功能，在Web 项目中提供 Spring 的容器；

  Spring-Webmvc：提供基于Servlet 的 Spring MVC；

  Spring-WebSocket：提供 WebSocket 功能；

  Spring-Webmvc-Portlet：提供Portlet 环境支持；

* （5）数据访问/集成（Data Access/Integration）

  Spring-JDBC：提供以JDBC 访问数据库的支持;
  Spring-TX：提供编程式和声明式的事务支持;
  Spring-ORM：提供对对象/关系映射技术的支持;
  Spring-OXM：提供对对象/xml映射技术的支持;
  Spring-JMS：提供对JMS的支持。



#### 1.1.3 Spring基础配置

Spring框架本身有四大原则:

1）使用POJO进行轻量级和最小侵入式开发

2）通过依赖注入和基于接口编程实现松耦合

3）通过AOP和默认习惯进行声明式编程

4）使用AOP和模板（template）减少模块化代码



#### 1.1.4 依赖注入

控制反转（IOC）和依赖注入（DI）在Spring环境下是等同的概念，控制反转是通过依赖注入实现的。所谓的依赖注入，指的是容器负责创建对象和维护对象之间的依赖关系，而不是通过自身的创建和解决自己的依赖。

依赖注入是为了解耦，体现出一种”组合“的概念。原因是通过继承使得一个类具有某项功能，将会使子类和父类耦合，而通过”组合“降低了耦合。

Spring IOC容器（ApplicationContext）负责创建bean，并通过容器将功能类Bean注入到你需要的Bean中去。

* 声明bean的注解

  * @Component
  * @Controller
  * @Service
  * @Repository

* 注入bean的注解

  * @Autowired： Spring提供的注解，按照类型装配，如果该类型存在多个或者不存在都会抛出异常，可以配合@Qualifier注解实现按照名称进行装配
  * @Inject： JSR-330提供的注解，按照类型装配，如果需要按名称进行装配，则需要配合@Named注解
  * @Resource：JSR-250提供的注解，一般会指定一个name属性

  参考：<https://www.cnblogs.com/pjfmeng/p/7551340.html> 



#### 1.1.5 AOP

AOP：面向切面编程，相对于OOP面向对象编程。

Spring的AOP的存在目的是为了解耦。AOP可以让一组类共享相同的行为。在OOP中，知道呢个通过继承类和实现接口，来使代码的耦合度增强，且类继承只能为单继承，阻碍更多行为添加到一组类上，AOP弥补了OOP的不足。Spring支持AspectJ的注解式切面编程。

（1）使用@Aspect声明式一个切面。

（2）使用@After、@Before、@Around定义建言（advice），可直接将拦截规则（切点）作为参数。

（3）为@After、@Before、@Around设置的参数为切点，切点可以单独配置，然后再将切点作为参数赋给@After、@Before、@Around

（4）符合条件的每一个被拦截处为连接点（JoinPoint）。

> 关于切点和连接点的区别:
>
> 切点:满足条件的所有被扫描到的目标方法.
>
> 连接点:可以理解为所有满足切点扫描条件的所有时机。例如@After的连接点是在被扫描到的目标方法的执行前，这个执行前可以理解为一个时机；而@Around的连接点是被扫描到的目标方法的执行前和执行后都是连接点,它有两个时机。可以这么说：切点定义了何处，而连接点则是定义了何时。



## 2 Spring 常用配置

### 2.1 Bean的Scope（范围、作用域）

（1）Singleton：一个Spring容器只有一个bean实例，可以理解为单例，如果声明bean时未指定，则默认就是它

（2）Prototype：每次调用都新建一个bean实例

（3）Request：Web项目中，为每个http request新建一个bean实例

（4）Session：Web项目中，为每个http session新建一个bean实例

（5）GlobalSession：这个只在portal应用中有用，给每一个global http session新建一个Bean实例。

socpe的使用，

```java
@Service
@Scope("prototype")
public class UserServiceImpl{
    ...
}
```



### 2.2 Spring EL和资源调用

* 注入普通字符串

  ```java
  @Value("Tom Smith")
  private String name;
  ```

* 注入操作系统属性

  ```java
  @Value("#{systemProperties['os.name']}")
  private String osName;
  ```

* 注入表达式运算结果

  ```java
  @Value("#{T(java.lang.Math).random() * 100.0}")
  private double randomNumber;
  ```

* 注入其他bean属性

  ```java
  @Value("#{demoService.another}")
  private String fromAnother;
  ```

* 注入文件资源

  ```java
  @Value("classpath:com/lee/demo/test.txt")
  private Resource testFile;
  ```

* 注入网址资源

  ```java
  @Value("http://www.baidu.com")
  private Resource testURL;
  ```

* 注入properties配置文件

  ```java
  @Configuration
  @PropertySource("......") // 必须
  public class DbConfig{
      @Value("${db.username}")
      private String username;
      
      @Value("${db.password}")
      private String password;
      
      @Value("${db.url}")
      private String url;
      
      // 必须
      @Bean
      public static PropertySourcesPlaceholderConfigurer propertyConfigure() {
          return new PropertySourcesPlaceholderConfigurer();
      }
  }
  ```

  注意:上面注释的"必须"的意思是在Spring中是这样的,因为SpringBoot中如果在`application.properties`文件中生命的db.xxx属性以后,直接使用@Value("xxx")即可。

### 2.3 Bean的初始化和销毁

* Java配置方法

  ```java
  @Bean(initMethod="init",destroyMethod="destroy")
  public HelloService helloService(){
      return new HelloService();
  }
  ```

* 注解方式

  利用JSR-250的@PostConstruct和@PreDestroy注解,直接在类中指定即可

  ```java
  @Service
  public class UserServiceImpl{
      @PostConstruct // 在构造方法执行完之后执行
      public void init(){}
      
      ...
          
      
      @PreDestroy // 在Bean销毁之前执行
      public void destroy(){}
  }
  ```



### 2.4 事件 Application Event

Spring的事件为Bean与Bean之间的消息通信提供了支持。当一个Bean处理完一个任务之后，希望另一个Bean知道并能做相应的处理，这时我们就需要让另外一个Bean监听当前Bean所发送的事件。

Spring的事件发送需要遵循如下的流程：

（1）自定义事件，继承ApplicationEvent

（2）自定义事件监听器，实现ApplicationListener

（3）使用容器发布事件 （使用ApplicationContext来发布事件）



## 3 Spring 高级话题

### 3.1 Spring Aware

* BeanNameAware

  获得容器中Bean名称

* BeanFactoryAware

  获得当前 bean factory ,这样可以调用容器服务

* ApplicationContextAware

  当前application context,可以调用容器的服务

* MessageSourceAware

  获得message source,这样可以获得文本信息

* ApplicationEventPublisherAware

  应用事件发布器,可以发布事件,可以发布事件

* ResourceLoaderAware

  获得资源加载器,可以获得外部资源文件

使用举例:

```java
@Service
public class EventPublisherService implements ApplicationEventPublisherAware {
    private ApplicationEventPublisher publisher;

    /**
     * 可以看到 不用实现ResourceLoaderAware,使用自动装配也是可以的
     */
    @Autowired
    private ResourceLoader resourceLoader;

    /**
     * bean加载的时候会自动回调,然后将值设置给成员变量publisher
     *
     * @param applicationEventPublisher
     */
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.publisher = applicationEventPublisher;
    }
}
```



### 3.2 Spring中多线程支持

Spring中使用多线程非常简单

(1) 开启异步支持，线程池配置

```java
@Configuration
@EnableAsync
public class ExecutorConfig implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
        // 设置其他参数
        threadPoolTaskExecutor.setCorePoolSize(5);
        return threadPoolTaskExecutor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return null;
    }
}
```



（2）在需要异步执行的方法上面加上 `@Async` 注解

### 3.3 计划任务

（1）开启计划任务支持: @EnableScheduling

（2）配置计划任务执行的线程池（计划任务默认是放在容量为1个线程的线程池中执行，即任务与任务之间是串行执行的）

（3）使用@Scheduled定义计划任务

代码示例入下：

```java
@Configuration
@EnableScheduling
public class ScheduleConfig implements SchedulingConfigurer {
    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        scheduledTaskRegistrar.setScheduler(Executors.newScheduledThreadPool(5));
    }

    @Scheduled(cron = "0/5 * * * * ?")
    public void sayHello() {
        System.out.println(Thread.currentThread().getName() + ">>>hello" + System.currentTimeMillis());
    }

    @Scheduled(cron = "0/5 * * * * ?")
    public void sayBye() {
        System.out.println(Thread.currentThread().getName() + ">>>bye" + System.currentTimeMillis());
    }
}
```



### 3.4 @Conditional + Condition

@Conditional 根据某一个特定条件创建一个特定的Bean。可以根据需要实现自己的Condition，然后将其放入到@Conditional注解中去。



