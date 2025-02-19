# Spring面试篇

# Spring

## Spring框架的核心特性

- **IOC（控制反转，Inversion of Control）**

  IOC 是 Spring 最核心的思想，它的作用是将对象的创建和管理交给 Spring 容器，而不是由应用程序代码自行控制。这通过依赖注入（DI，Dependency Injection）来实现。

  - 核心机制：
    - Bean管理：Spring负责创建、初始化、管理Bean对象。
    - 依赖注入（DI）：Spring 通过构造函数、Setter 方法、字段注入方式管理对象的依赖关系。
    - 降低耦合：组件之间不直接依赖，而是通过Spring容器注入。

- **AOP（面向切面编程，Aspect-Oriented Programming）**

  AOP 允许我们在不修改业务逻辑的情况下添加额外功能，如日志记录、事务管理、安全控制等。

  - 核心机制：
    - 横切关注点（如日志、事务、安全控制），不需要修改业务代码即可添加功能。
    - 通过动态代理（JDK 动态代理或 CGLIB）拦截方法调用，在执行前后添加逻辑。

- **事务管理（Transaction Management）**

  Spring 通过声明式事务（`@Transactional`）和编程式事务提供强大的事务管理能力，支持多种事务管理器（JDBC、JPA、Hibernate、MyBatis等）。

- **Spring MVC（Web 框架）**

  Spring MVC 是一个基于 Servlet 的轻量级 Web 框架，提供了前后端分离、RESTful API、视图解析等功能。

  - 核心机制：
    - 采用前端控制器模式（DispatcherServlet）处理请求
    - 通过 `@Controller`、`@RequestMapping` 构建 RESTful API。

## 介绍一下IOC

Spring IOC 通过反射+工厂模式实现对象创建、管理和依赖注入。

其核心流程为**启动-->扫描-->依赖注入-->初始化-->后置处理器**

-  启动（IOC 容器启动）

  Spring IOC 容器的启动和创建主要由 `ApplicationContext` 负责，常见的启动方式分为根据XML配置`ClassPathXmlApplicationContext`或Java配置`AnnotationConfigApplicationContext`。

  ```java
  ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
  ```

- 扫描（ComponentScan 组件扫描）

  Spring 通过 `@ComponentScan` 或XML 配置扫描指定的包，找到所有的 `@Component`、`@Service`、`@Repository`、`@Controller` 注解的类，并注册成BeanDefinition。

  - 扫描指定包路径（`ClassPathBeanDefinitionScanner`）。
  - 解析 `@Component` 注解的类，生成 `BeanDefinition`（但此时 Bean 还未实例化）。
  - 注册 `BeanDefinition` 到 `BeanFactory`，等待后续实例化，也就是放入三级缓存BeanFactory。

- 依赖注入（Dependency Injection, DI）

  Spring 在实例化 Bean 时，会自动注入它的依赖对象，主要有三种方式：构造注入、Setter方法注入、字段注入。通常使用字段注入。

  - 依赖注入流程：
    - 获取 `BeanDefinition`，确定要创建的 Bean。
    - 实例化 Bean（反射创建对象）。
    - 解析 `@Autowired`，查找依赖对象：先从单例池找，单例池没有就递归创建依赖对象。**（循环依赖会在这里被解决）**
    - 完成依赖注入（调用构造方法或 setter 方法）。

- 初始化（InitializingBean 和 @PostConstruct）

  Bean实例化完成并完成依赖注入后，Spring 会执行 Bean 的初始化逻辑。先执行`@PostConstruct`注解的方法再执行重写`afterPropertiesSet()`方法。

- 后置处理器（BeanPostProcessor）

  Spring 提供 `BeanPostProcessor` 在 Bean 初始化前后进行增强，比如 AOP、事务管理等都依赖它。

  - Bean 创建完成后，调用 `postProcessBeforeInitialization()`（初始化前增强）。
  - 执行Bean的初始化方法，如`@PostConstruct`注解的方法再执行重写`afterPropertiesSet()`方法。
  - Bean 初始化完成后，调用 `postProcessAfterInitialization()`（初始化后增强）。
  - 如果有 AOP 代理，此时会生成代理对象，并返回代理对象，而不是原始 Bean。

## 介绍一下AOP

AOP 允许我们在不修改业务逻辑的情况下，添加额外功能，如日志记录、事务管理、安全控制等。

### AOP核心概念

- **切面（Aspect）**：封装一组增强逻辑（例如日志、事务），是 AOP 的核心。

- **切点（Pointcut）**：定义哪些方法需要被增强。

- **通知（Advice）**：定义增强逻辑的具体执行时机，如 前置通知（Before）、后置通知（After）、环绕通知（Around）等。

- **代理对象（Proxy）**：Spring AOP 通过 JDK 动态代理或 CGLIB 代理 生成代理对象，并在代理对象的方法调用前后插入增强逻辑。

- **目标对象（Target）**：实际被代理的业务对象。

### 静态代理和动态代理

- **静态代理**：适用于代理对象不多、代理关系在编译时就能确定的简单场景。
  - 代理类与目标类之间有直接的关系，需要在编译时就确定。
  - 每个类都需要一个代理类，无法在运行时动态生成。
- **动态代理**：适用于代理对象较多，且在运行时需要动态生成代理类的复杂场景，如 AOP、事务管理、缓存等。

### AOP代理方式

Spring AOP基于代理对象实现，而代理对象的生成方式取决于目标对象是否实现了接口：

- JDK动态代理（基于接口）：适用于目标类实现了接口的情况。
  - 使用 `java.lang.reflect.Proxy` 生成代理对象。
  - 要求目标对象必须实现接口。
  - 代理对象会实现相同的接口，并在方法调用时拦截增强逻辑。
- CGLIB动态代理（基于子类继承）：适用于目标类没有实现接口的情况。
  - 使用 CGLIB（Code Generation Library）生成代理对象。
  - 适用于目标对象没有实现接口的情况。
  - 通过创建目标类的子类，并在方法调用时插入 AOP 逻辑。

### AOP执行流程

当我们从 Spring 容器中获取一个 Bean 时，实际返回的是一个代理对象。代理对象会在方法调用的开始、执行过程和结束时，插入相应的回调函数进行增强。

- Spring容器启动，读取所有切面配置中的切入点
- 初始化Bean，判定bean对应的类中的方法是否匹配到任意的切入点
  - 匹配失败，创建原始对象
  - 匹配成功，创建原始对象的代理对象
- 获取bean执行方法
  - 获取的bean如果是原始对象，调用方法并执行，完成操作
  - 获取的bean如果是代理对象，根据代理对象的运行模式运行原始方法与增强的内容，完成操作。

## 什么是循环依赖

- 构造方法循环依赖（Spring 无法解决）

  当两个 Bean 通过构造方法互相依赖时，Spring 无法解决，会抛出 `BeanCurrentlyInCreationException`。

  ```java
  @Component
  public class A {
      private final B b;
      public A(B b) { this.b = b; }
  }
  @Component
  public class B {
      private final A a;
      public B(A a) { this.a = a; }
  }
  ```

- Setter / 字段注入循环依赖（Spring 通过三级缓存解决）

  如果 `A` 和 `B` 之间是 setter 方法 或 字段注入，Spring可以解决

  ```java
  @Component
  public class A {
      @Autowired private B b;
  }
  @Component
  public class B {
      @Autowired private A a;
  }
  ```

## Spring是如何解决循环依赖的？

Spring 采用"三级缓存"机制来解决Setter / 字段注入形式的循环依赖。

| 三级缓存                              | 作用                                         |
| ------------------------------------- | -------------------------------------------- |
| **一级缓存（singletonObjects）**      | 存放完全初始化好的单例 Bean（可直接使用）    |
| **二级缓存（earlySingletonObjects）** | 存放提前曝光的 Bean 实例（未初始化完成）     |
| **三级缓存（singletonFactories）**    | 存放可以创建 Bean 的工厂（用于代理对象创建） |

解决的过程：

- 创建Bean实例（但不初始化）

  - 通过反射创建Bean的原始实例，但未进行依赖注入
  - 将创建Bean的工厂存入三级缓存

- 提前暴露Bean（放入二级缓存）

  - Spring允许其他Bean对象访问这个还未初始化的对象，从而解决循环依赖的问题。

- 依赖注入

  - A需要B，Spring发现B正在创建中，就会尝试从缓存获取，一级缓存找不到找二级缓存。在二级缓存找到提前暴露的B，就可以直接使用。如果B需要A，但A也提前暴露了，所以B也能拿到A。

- 完成初始化，存入一级缓存。

  A和B完成初始化，分别从二级缓存移除，存入一级缓存。

## Spring框架中都用到了那些设计模式

- 工厂设计模式：Spring使用工厂模式通过BeanFactory、ApplicationContext 创建 bean 对象。
- 代理设计模式：AOP
- 单例设计模式：单例池及单例池里的Bean对象。
- 包装器设计模式：针对不同的数据库可以动态的切换数据源
- 观察者模式：Spring 中“订阅-通知”的事件驱动模型就是观察者模式很经典的一个应用。

## Spring常用注解

- IOC中：
  - `@Component`：标记一个类为 Spring 容器的 Bean。
  - `@Autowired`：自动装配 Bean（依赖注入），Spring 会自动为该字段、构造器或方法注入匹配的 Bean。
- AOP中：
  - `@Transactional`：声明事务管理。标记方法或类，支持事务的自动管理。

- MVC中：
  - `@Controller`, `@RequestMapping`, `@GetMapping`等

## autowired和resource的区别

**`@Autowired`**：Spring 特有，用于自动根据类型注入 Bean。通常更常用，支持灵活的自动注入。

**`@Resource`**：JSR-250 标准，优先根据名称注入，如果没有匹配的名称，再根据类型注入。

## Spring的事务什么情况下会失效？

- 未捕获异常：如果一个事务方法中发生了未捕获的异常，并且异常未被处理或传播到事务边界之外，那么事务会失效，所有的数据库操作会回滚。
- 事务传播属性设置不当：非事务方法调用事务方法会导致事务失效。
- 多数据源的事务管理
- 事务在非公开方法中失效：`@Transactional`如果标注在私有方法和非public方法上，事务也会失效。

# SpringMVC

## MVC分层

MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范。

- **视图(view)：** 为用户提供使用界面，与用户直接进行交互。
- **模型(model)：** 代表一个存取数据的对象或 JAVA POJO。分为两类
  - 数据承载类Bean：实体类
  - 业务处理类Bean：Service或Dao对象，专门处理用户提交请求的。
- **控制器(controller)：**用于将用户请求转发给相应的 Model 进行处理，并根据 Model 的计算结果向用户提供相应响应。它使视图与模型分离。

## SpringMVC的工作流程

SpringMVC 是基于 前端控制器模式（Front Controller Pattern）的 Web 框架，它的工作流程是通过一系列的步骤来完成请求的处理和响应的返回。

流程：

- 用户通过view页面向服务端提出请求，包括表单请求、超链接请求、ajax请求等等。
- 请求到达 DispatcherServlet（前端控制器）
  - `DispatcherServlet` 是 SpringMVC 的核心控制器，负责接收所有请求，并进行分发。每个请求都会经过 `DispatcherServlet`。
  - 该控制器从 web.xml 或 Spring 配置文件中获取映射信息，知道如何分发请求。
- 请求解析和处理及选择处理器
  - `DispatcherServlet` 将请求传递给 HandlerMapping，它根据请求的 URL 查找与之匹配的处理器（Controller）。
  - `HandlerMapping`：根据请求的 URL 和配置的映射规则（比如 `@RequestMapping` 注解）找到对应的处理器（即 Controller 中的方法）。
  - `DispatcherServlet` 将请求传递给处理器方法。
- 执行控制器方法（Handler）
  - 控制器方法开始执行，处理客户端请求。控制器方法可以根据业务逻辑获取数据、处理请求、并返回 Model 数据。

- Controller再将处理结果向客户端发回响应View页面，渲染后发送给客户端。

## Handlermapping 和 handleradapter有了解吗？

- **HandlerMapping**：负责根据请求的 URL 找到匹配的Controller方法。

- **HandlerAdapter**：负责适配不同类型的 Handler，调用对应的处理方法来执行请求。

具体来说

```java
@Controller
public class MyController {
    @RequestMapping("/test")
    public String handleRequest() {
        // 处理请求的业务逻辑
        return "viewName";  // 返回视图名
    }
}
```

**`HandlerMapping`**：

- 负责根据请求的 URL 找到对应的 **Handler**（即 `MyController` 中的 `handleRequest()` 方法）。
- 当用户访问 `/test` 路径时，`HandlerMapping` 会查找配置的请求映射，发现 `/test` 映射到 `MyController` 类中的 `handleRequest()` 方法。
- `HandlerMapping` 返回一个 **Handler**（即方法 `handleRequest()`）和相关的拦截器（如果有的话）。

**`HandlerAdapter`**：

- 负责执行 **Handler**（即 `handleRequest()` 方法）。
- `HandlerAdapter` 是根据 `Handler` 的类型选择合适的执行方式。`HandlerAdapter` 会调用 `handleRequest()` 方法来处理请求。

# SpringBoot

## 为什么使用springboot

- **自动配置（Auto Configuration）**
  - Spring：在Spring需要手动配置大量的 XML 配置文件，或者使用 Java 配置类进行大量的配置。
  - Spring Boot：通过自动配置，Spring Boot 会根据项目的依赖和环境自动配置相关组件，减少了大量的配置工作。例如，Spring Boot 可以自动配置数据库连接池、嵌入式服务器（如 Tomcat）、JPA 等，开发者不需要显式地定义它们。

- **内嵌Web服务器**

  - Spring：需要在外部部署 Web 服务器（如 Tomcat）并将应用打包成 WAR 文件进行部署。

  - Spring Boot：Spring Boot 默认支持嵌入式的 Web 服务器，可以将 Spring Boot 应用打包为一个 可执行 JAR 文件，不依赖外部服务器。这样应用可以直接运行，减少了配置和部署复杂性。

- **快速开发与启动（Starter POMs）**

  - Spring：在传统的 Spring 中，开发者需要自己管理所有依赖，可能会遇到版本冲突、依赖缺失等问题。
  - Spring Boot：通过 `starter` 模块，Spring Boot 提供了一些预先配置好的常用依赖（例如 `spring-boot-starter-web`、`spring-boot-starter-data-jpa` 等）。

## SpringBoot自动装配原理

Springboot的启动类都会有`@SpringBootApplication`这个注解，这个注解包含：

- @SpringBootConfiguration

  - 这个注解表示该类是 Spring Boot 应用的配置类，功能类似于传统 Spring 中的 `@Configuration` 注解，表示该类可以包含 Bean 定义。

- @EnableAutoConfiguration

  - 这个注解告诉 Spring Boot 启动自动配置功能。它会根据项目的类路径、配置文件等信息，自动配置应用程序所需的组件和服务。

- @ComponentScan

  - 这个注解让 Spring Boot 扫描当前包及其子包中的所有组件（如 `@Component`、`@Service`、`@Repository`、`@Controller` 等），并将它们自动注册到 Spring 容器中。默认会扫描 `@SpringBootApplication` 注解所在类的包及其子包。

  **自动配置原理：**

  - `SpringApplication` 启动：应用启动时，`SpringApplication.run()` 方法会启动 Spring 上下文。
  - `@EnableAutoConfiguration` 触发自动配置：`@EnableAutoConfiguration` 会通过 `spring.factories` 加载自动配置类。
  - 条件判断：自动配置类会使用 `@Conditional` 系列注解来判断是否需要进行配置。
  - Bean 注册：符合条件的自动配置类会向 Spring 容器注册相关的 Bean。

## 说几个启动器（starter)？

- **spring-boot-starter-web**：含了Spring MVC和Tomcat嵌入式服务器
- **spring-boot-starter-test**：用于单元测试和集成测试，包含常用的测试框架和工具
- **mybatis-spring-boot-starter**：由MyBatis团队提供的，用于简化在Spring Boot应用中集成MyBatis的过程。
- **spring-boot-starter-jdbc**：可以让你轻松地与MySQL数据库进行交互。
- **spring-boot-starter-data-redis**：用于集成Redis缓存和数据存储服务。

## 自己实现过SpringBoot demo吗？

- **模拟依赖**

  为了实现SpringBoot starter，因为springboot starter自带mvc和tomcat，所以我们创建自己的SpringBoot starter第一步也要先引入一下依赖，包括springmvc、web、tomcat等

  ```xml
  <dependencies>
  	<dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context</artifactId>
              <version>5.3.18</version>
          </dependency>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-web</artifactId>
              <version>5.3.18</version>
          </dependency>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-webmvc</artifactId>
              <version>5.3.18</version>
          </dependency>
  
          <dependency>
              <groupId>javax.servlet</groupId>
              <artifactId>javax.servlet-api</artifactId>
              <version>4.0.1</version>
          </dependency>
  
          <dependency>
              <groupId>org.apache.tomcat.embed</groupId>
              <artifactId>tomcat-embed-core</artifactId>
              <version>9.0.60</version>
          </dependency>
  </dependencies>
  ```

- **模拟 `@SpringBootApplication` 注解**

  Spring Boot 中的 `@SpringBootApplication` 是一个组合注解，其中包含 `@Configuration`、`@ComponentScan` 和 `@EnableAutoConfiguration`。我们可以模拟它，创建一个自定义注解 `@SJBSpringBootApplication`，其功能类似。

  ```java
  @Target(ElementType.TYPE)
  @Retention(RetentionPolicy.RUNTIME)
  @Configuration
  @ComponentScan
  public @interface SJBSpringBootApplication {
  }
  ```

- **模拟 `SpringApplication.run()`**

  Spring Boot 的启动是通过 `SpringApplication.run()` 方法来完成的。在模拟的实现中，创建了一个 `SJBSpringApplication` 类，模仿 `SpringApplication`，并实现了 `run()` 方法。

  ```java
  public class SJBSpringApplication {
      public static void run(Class<?> primarySource, String... args) {
          // 创建并刷新 Spring 容器
          AnnotationConfigWebApplicationContext applicationContext = new AnnotationConfigWebApplicationContext();
          applicationContext.register(primarySource);
          applicationContext.refresh();
  
          // 启动 Tomcat
          startTomcat(applicationContext);
      }
  }
  ```

- **模拟启动内嵌Tomcat**

  内嵌 Tomcat 是 Spring Boot 的一个核心特性

  ```java
  public static void startTomcat(WebApplicationContext applicationContext) {
      Tomcat tomcat = new Tomcat();
      Server server = tomcat.getServer();
      Service service = server.findService("Tomcat");
  
      Connector connector = new Connector();
      connector.setPort(8081);
  
      Engine engine = new StandardEngine();
      engine.setDefaultHost("localhost");
  
      Host host = new StandardHost();
      host.setName("localhost");
  
      Context context = new StandardContext();
      context.setPath("");
      context.addLifecycleListener(new Tomcat.FixContextListener());
  
      host.addChild(context);
      engine.addChild(host);
      service.setContainer(engine);
      service.addConnector(connector);
  
      tomcat.addServlet("", "dispatcher", new DispatcherServlet(applicationContext));
      context.addServletMappingDecoded("/*", "dispatcher");
  
      try {
          tomcat.start();
      } catch (LifecycleException e) {
          e.printStackTrace();
      }
  }
  ```

  这里通过 `Tomcat` 类实例化并配置了一个内嵌的 Tomcat 服务器：

  - **配置端口**：将 Tomcat 端口设置为 8081。
  - **绑定 `DispatcherServlet`**：将 `DispatcherServlet` 添加到 Tomcat，并映射到 `"/*"` 路径，处理所有的请求。
  - **启动 Tomcat**：通过 `tomcat.start()` 启动服务器。

- **启动整个应用**

  在 `UserApplication` 类中，使用 `@SJBSpringBootApplication` 注解来标记启动类，并通过 `SJBSpringApplication.run()` 启动应用。

  ```java
  @SJBSpringBootApplication
  public class UserApplication {
      public static void main(String[] args) {
          SJBSpringApplication.run(UserApplication.class, args);
      }
  }
  ```

  `UserApplication` 类作为配置类，使用 `@ComponentScan` 自动扫描并加载 `UserController`。

- **定义业务逻辑**

  `UserController` 是一个典型的 Spring MVC Controller，它处理 HTTP 请求并返回响应。使用 `@RestController` 注解使得每个方法都直接返回 JSON 格式的响应。

  ```java
  @RestController
  public class UserController {
      @GetMapping("/test")
      public String test() {
          return "sjb123";
      }
  }
  ```

  `/test` 路径的请求将由 `UserController` 的 `test()` 方法处理，返回 `"sjb123"`。

## Springboot中的注解

- **@SpringBootApplication**：用于标注主应用程序类，标识一个Spring Boot应用程序的入口点，同时启用自动配置和组件扫描。**默认情况下，Spring 会扫描启动类所在包及其子包中的所有组件。**
- **@Controller**：标识控制器类，处理HTTP请求。
- **@RestController**：结合@Controller和@ResponseBody，返回RESTful风格的数据。
- **@Service**：标识服务类，通常用于标记业务逻辑层。
- **@Component**：通用的Spring组件注解，表示一个受Spring管理的组件。
- **@Autowired**：用于自动装配Spring Bean。
- **@RequestMapping**：用于映射HTTP请求路径到Controller的处理方法。
- **@Configuration**：用于指定一个类为配置类，其中定义的bean会被Spring容器管理。通常与@Bean配合使用，@Bean用于声明一个Bean实例，由Spring容器进行管理。
- **@GetMapping、@PostMapping、@PutMapping、@DeleteMapping：**简化@RequestMapping的GET、POST、PUT和DELETE请求。

# MyBatis

## 与传统的JDBC相比，MyBatis的优点？

- 代码简洁
  - JDBC要写大量sql语句，容易出错。
  - mybatis只要编写SQL映射文件（XML或者注解）。可以直接通过Mapper直接调用。提高开发效率。

- SQL与代码解耦
  - JDBC与代码捆绑，修改sql要改代码，难以维护。
  - mybatis：SQL 语句可以放在XML 配置文件或注解中。
- 提供 ORM（对象关系映射）支持、
  - JDBC需要手动将 `ResultSet` 转换为 Java 对象，代码冗长。
  - mybatis：可以自动映射查询结果到 Java 对象，减少手动转换的工作。
- 更灵活的SQL
  - JDBC的sql语句无法动态修改
  - mybatis：支持 动态 SQL（`<if>、<choose>、<foreach>` 等标签），可以根据条件动态生成 SQL。

- 安全性更高
  - JDBC的sql语句容易被SQL注入
  - MyBatis：使用 `#{}` 绑定参数，防止 SQL 注入

## JDBC连接数据库的步骤

- 加载数据库驱动程序，使用 `Class.forName()` 加载数据库驱动。
- 建立数据库连接，使用 `DriverManager.getConnection()` 获取数据库连接。
- 创建Statement或PreparedStatement对象，用于执行 SQL 查询或更新操作。
- 执行sql语句
  - `executeQuery()`（查询）
  - `executeUpdate()`（增、删、改）
- 处理查询结果`ResultSet`
- 关闭数据库资源（`ResultSet` → `Statement` → `Connection`）

## 如果项目中要用到原生的mybatis去查询，该怎样写？

- 配置mybatis
- 创建实体类
- 编写sql映射文件
- 编写dao接口
- 在xml文件中编写具体的sql查询语句
- 在dao层调用查询方法

## Mybatis里的 # 和 $ 的区别？

- mybatis中在处理#{}会将其替换为？，之后调用PreparedStatement 的 set 方法来赋值，执行效率更高，并且防止sql注入，提高安全性。
- 在处理${}只是简单的参数拼接，不能防止sql注入，会导致安全问题。

## MybatisPlus和Mybatis的区别？

MybatisPlus是一个基于MyBatis的增强工具库，旨在简化开发并提高效率。

| **对比项**        | **MyBatis**                           | **MyBatis-Plus (MP)**                    |
| ----------------- | ------------------------------------- | ---------------------------------------- |
| **开发效率**      | 需要手写 SQL 语句                     | 内置 CRUD 方法，大大减少 SQL 书写        |
| **代码量**        | 需要编写 `Mapper.xml` 和 `Mapper接口` | 只需继承 `BaseMapper<T>`，省去大量 `xml` |
| **SQL 复杂度**    | 适合复杂 SQL，自定义灵活              | 适合简单增删改查，复杂 SQL 仍需自定义    |
| **分页**          | 需要手写 SQL 或自己封装               | **内置分页插件**，支持 `Page<T>`         |
| **SQL 语句**      | 需要手动写 SQL，控制 SQL 执行         | **内置 CRUD 方法**，默认 SQL 生成        |
| **Lambda 表达式** | 需要自己拼接 SQL                      | **提供 LambdaQueryWrapper**，代码更优雅  |
| **事务管理**      | 依赖 Spring 事务                      | 依赖 Spring 事务                         |
| **动态 SQL**      | 通过 `if`、`choose` 等 XML 方式实现   | 通过 `Wrapper` 方式更简洁                |
| **适用场景**      | 适用于复杂 SQL、自定义查询            | 适用于**CRUD 快速开发**，减少 SQL 工作量 |

# SpringCloud

## 了解SpringCloud吗，说一下他和SpringBoot的区别

Spring Boot是用于构建单个Spring应用的框架，而Spring Cloud则是用于构建分布式系统中的微服务架构的工具，Spring Cloud提供了服务注册与发现、负载均衡、断路器、网关等功能。

两者可以结合使用，通过Spring Boot构建微服务应用，然后用Spring Cloud来实现微服务架构中的各种功能。

## 用过那些微服务组件？

- **Nacos**：注册中心 & 配置中心，负责服务注册与发现，以及动态配置管理。

- **Feign**：基于 HTTP 的声明式 RPC 客户端，简化微服务之间的调用。

- **Spring Cloud Gateway**：API 网关，负责路由、负载均衡、权限控制等功能。

- **Sentinel**：流量控制 & 熔断限流，提供高可用保护，防止雪崩效应。

## rpc框架有那些

RPC（Remote Procedure Call，远程过程调用）框架用于在分布式系统中实现不同节点间的通信。通过 RPC，客户端可以像调用本地方法一样调用远程服务器上的方法。常见的 RPC 框架有很多，以下是一些常用的 RPC 框架：

- **gRPC**：高效、支持流的 RPC 框架，基于 HTTP/2 和 Protocol Buffers。

- **Dubbo**：阿里巴巴开源的高性能 RPC 框架，支持分布式服务治理。

- **Spring RMI**：基于 Java 的远程方法调用（RMI）机制，适用于 Spring。

- **Thrift**：由 Facebook 开发，支持跨语言的 RPC 框架。

- **OpenFeign**：Spring Cloud 的声明式 HTTP 客户端，简化微服务间的通信。

## OpenFeign原理

OpenFeign 通过 **动态代理** 实现远程调用：

1. **接口定义**：定义一个接口，使用 Feign 注解（如 `@GetMapping`、`@PostMapping`）。
2. **代理生成**：Feign 会为这个接口创建一个动态代理。
3. **远程调用**：当调用接口方法时，Feign 会自动生成 HTTP 请求，并将响应返回给调用者。

## 负载均衡算法

Nginx 负载均衡算法主要用于将流量分发到多个后端服务器，提高系统的吞吐量和可用性。

- **轮询（默认）**

  - 优点：简单
  - 缺点：无法动态感知服务器的实时负载情况。

- **加权轮询**：为每台服务器分配权重

  - 使用场景：性能不同的服务器，分配不同的流量
  - 优点：适用于不同性能的服务器，可以让高性能服务器处理更多请求。
  - 缺点：无法动态感知服务器的实时负载情况。

- **IP哈希**：根据客户端 IP 地址计算哈希值，每个 IP 地址始终访问同一台服务器。

  - 使用场景：需要会话保持的应用，如购物车、支付系统等。
  - 优点：确保相同的用户请求始终被路由到相同的服务器，适用于状态依赖型应用。
  - 缺点：服务器变更会影响映射，如果一台服务器下线，原本映射到该服务器的请求不会自动转移。可能会导致流量不均衡，如果某些 IP 地址的请求特别多，会给对应服务器造成过载。

- **最少连接**：Nginx 将请求转发给当前**活跃连接数最少**的服务器，以保证负载均衡。

  - 使用场景：服务器处理能力不同，或请求处理时间长短不一的情况下。
  - 优点：适用于长连接、慢请求的场景，如 WebSocket、数据库连接池等。
  - 缺点：需要 Nginx 维护服务器连接状态，有一定计算开销。

- **健康检查**：除了负载均衡策略，Nginx 还可以通过 `max_fails` 和 `fail_timeout` 设置服务器的健康检查和故障转移。

  - 优点：服务器异常时，自动切换到健康的服务器，提高系统可用性。
  - 缺点：需要配合 `proxy_next_upstream` 进一步优化容错机制。


## SpringCloudAlibaba主要包括那些

- **Nacos**（服务注册与配置管理）
- **RocketMQ**（消息队列）
- **Sentinel**（流量控制与熔断降级）
- **Dubbo**（高性能 RPC 框架）
- **Seata**（分布式事务）







































































