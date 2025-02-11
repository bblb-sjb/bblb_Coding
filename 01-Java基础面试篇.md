# 概念

## Java特点

- 平台无关
- 面向对象
- 拥有自己的内存管理机制

## Java为什么跨平台

Java 之所以能够跨平台，是因为不同操作系统都有对应版本的 Java 虚拟机（JVM）。只要在目标平台上安装相应的 JVM，就能运行 Java 的字节码文件（`.class`）。

字节码本身是跨平台的，但 JVM 并不能跨平台。不同操作系统需要不同版本的 JVM 来解析 Java 字节码，并将其转换为该平台对应的机器码，以供执行。

## JVM、JDK、JRE关系

JVM<JDK<JRE

- JVM是虚拟机，本质上是一个用来运行java字节码文件的程序。
  - 对字节码文件中的指令，实时的解释成机器码供计算机执行
  - 自动为对象、方法分配内存以及回收不再使用的对象
  - 即时编译**JIT**对热点代码进行优化，提升效率，JIT将热点代码转为机器码后存至RAM，下次运行可以直接从RAM中调用。

- JRE是虚拟机+类库，也就是java的运行时环境。

- JDK包含虚拟机jvm、编译器javac、调试器jdb和一些标准库和工具库。

# 面向对象

## 什么是面向对象，什么是封装继承多态？

面向对象就是将事物抽象成对象，提取出对应的属性或者方法，以该对象为中心，通过对象与对象之间交互来完成所需功能。

java面向对象三大特征：封装、继承、多态

- 封装：将对象的属性和方法结合起来，对外隐藏内部细节，仅通过对象的暴露出来的接口进行交互。
- 继承：子类共享父类数据结构和方法，是代码复用的主要手段。
- 多态：指多个不同类的对象对同一接口展现出不同的运行状态，也就是不同实力对同一接口表现出来的不同操作，多态分为编译时多态（重载）和运行时多态（重写）。

## 多态体现的方面

- 方法重载：同一个命名的方法可以有多种参数列表。

- 方法重写：子类可以重写父类同名行为。

- 接口与实现：多个类可以实现同一接口，多个动物类实现动物接口并调用动物接口的方法则会出发对应的实现。

- 向上向下转型

  - 向上：父类引用指向子类对象，只能调用父类忒的那个方法，不能调用子类特有方法。

    ```java
    Animal animal = new Dog();
    animal.makeSound();
    ```

  - 向下：父类引用转为子类类型，转为子类类型必须强制转换，可以调用子类特有方法。

    ```java
    Animal animal = new Dog();
    Dog dog = (Dog) animal;
    dog.makeSound();
    dog.wagTail();`
    ```

    向下转有风险

## 面向对象设计原则

- 单一职责：一个类只负责一项职责。
- 开放封闭：对拓展开放，对修改封闭。
- 里氏替换：子类对象应该能够替换所有父类对象，并且程序的行为不会发生变化。
- 接口隔离：接口应该设计得小而专，同时通过依赖注入管理依赖关系。强调通过依赖注入来管理类之间的依赖关系，而不是在类内部直接创建依赖，从而实现松耦合。
- 依赖倒置：高层次模块不应该依赖于底层模块。
- 最少知识：一个类应该对其他对象最少了解。

## 重载重写的区别

- 重载是对同一命名方法可以拥有不同的参数列表，编译器根据调用时参数类型来自动选择调用哪个方法
- 重写是子类重写父类同名方法，通过`@override`来标注

## 抽象类和实体类区别

- 实例化：抽象类不能被实例化，只能被继承
- 方法：抽象类的方法可以没有具体实现
- 继承：
  - 一个类只能继承一个普通类，但可以实现多个接口
  - 一个类只能继承一个抽象类，但可以实现多个接口。
- 实现限制：抽象类一般是基类，供其他类继承。而实体类可以被其他类继承和使用。

## Java抽象类和接口的区别

- 两者特点：
  - 抽象类描述类的共同特性和行为，可以有具体的成员变量，构造方法和具体方法。
  - 接口可以多实现，只能有常量字段和抽象方法。

- 两者区别：
  - 实现接口的关键字`implements`，继承为`extends`，一个类可以实现多个接口，一个类只能继承一个抽象类。
  - 接口只能有定义，而抽象类可以有定义与实现。
  - 接口成员默认是常量，`public static final`，并且必须赋初值不能背修改，方法为`public abstract`的。抽象类成员默认`default`，可以被定义，抽象方法由`abstract`修饰，必须以分号结尾，不带花括号。
  - 抽象类可以由实体变量和静态变量，而接口只能由静态常量。

## 抽象类可以被实例化吗

抽象类不能通过new被实例化，但是抽象类可以有自己的构造器，在子类实例化的过程也会被调用，以便进行必要的初始化工作。

# 深拷贝浅拷贝

## 区别

浅拷贝类似于快捷方式，只是创建了一个新的对象，指向原复制对象的地址，所以如果原复制对象发生变化，他也跟着变化。

深拷贝是指在复制对象的同时，重新开辟一部分内存区域，将其的全部字面值都复制一份，创建一个新对象，与原复制对象除了值一样外没有其他关系。

## 实现深拷贝方法

- 实现Cloneable接口并重现clone方法
- 使用序列化和反序列化：将对象序列化为字节流，在通过字节流反序列化为对象实现深拷贝，要求对象和引用类型实现`Serializable`接口
- 手动复制

# 对象

## java创建对象的方式

- new `MyClass obj = new MyClass();`

- 通过反射使用Class类的newInstance()方法：

  ```java
  MyClass obj = (MyClass) Class.forName("com.example.MyClass").newInstance();
  ```

- 使用Constructor类的newInstance()方法：

  ```java
  Constructor<MyClass> constructor = MyClass.class.getConstructor(String.class);
  MyClass obj = constructor.newInstance("John");
  ```

- 使用clone()方式，`clone()` 方法是 `Object` 类的一个方法，必须实现 `Cloneable` 接口才能正常工作。

  ```java
  class MyClass implements Cloneable {
      private String name;
      MyClass(String name) {
          this.name = name;
      }
      void display() {
          System.out.println("Name: " + name);
      }
      @Override
      protected Object clone() throws CloneNotSupportedException {
          return super.clone();  // 调用 Object 的 clone() 方法
      }
  }
  public class Main {
      public static void main(String[] args) throws CloneNotSupportedException {
          MyClass original = new MyClass("John");
          MyClass cloned = (MyClass) original.clone();
          cloned.display();
      }
  }
  ```

- 使用反序列化

  ```java
  class MyClass implements Serializable {
      private String name;
      MyClass(String name) {
          this.name = name;
      }
      void display() {
          System.out.println("Name: " + name);
      }
  }
  public class Main {
      public static void main(String[] args) throws IOException, ClassNotFoundException {
          ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("obj.dat"));
          MyClass original = new MyClass("John");
          out.writeObject(original);
          out.close();
          ObjectInputStream in = new ObjectInputStream(new FileInputStream("obj.dat"));
          MyClass deserialized = (MyClass) in.readObject();
          in.close();
          deserialized.display();
      }
  }
  ```

## new出来的对象什么时候回收

new出来的对象由GC进行回收，根据一下算法进行检测：

- 引用计数法：为每个对象维护一个引用计数器，当对象被引用时加1，取消引用时减1。
- 可达性分析法：可达性分析将对象分类两类：垃圾回收的根对象GCRoot和普通对象，如果某个到GCRoot对象是可达的那么这个对象就不可回收。

# 反射

## 什么是反射

指在运行中对于任意一个类或对象都知道其所有属性及方法，这种动态获取信息及动态调用对象的方法称为java的反射机制。

- 运行时类信息访问。

- 动态对象创建，即使不知道具体类名也可以通过Class类的newInstance()或Constructor对象的newInstance()创建。

- 动态方法调用，通过调用Method的invoke方法实现，可以调用私有方法。

- 访问和修改字段值，即使时私有的也可以通过Field类的get()和set()方法调用。

  ```java
  class Person {
      private String name = "Alice";
      private void sayHello() {
          System.out.println("Hello, my name is " + name);
      }
  }
  Person person = new Person();
  Field nameField = Person.class.getDeclaredField("name");
  nameField.setAccessible(true); // 允许访问私有字段
  System.out.println("Before modification: " + nameField.get(person));
  nameField.set(person, "Bob"); // 修改私有字段的值
  Method sayHelloMethod = Person.class.getDeclaredMethod("sayHello");
  sayHelloMethod.setAccessible(true); // 允许访问私有方法
  sayHelloMethod.invoke(person); // 调用方法
  ```

## 反射应用场景

- 加载数据库驱动

  我们可以根据实际情况通过反射动态的来加载驱动类，那么在JDBC连接数据库的时候我们就可以通过Class.forName()通过反射加载对应的驱动类

  ```java
  Class.forName("com.mysql.cj.jdbc.Driver");
  Connection connection = DriverManager.getConnection(url, user, password);
  ```

  通过反射加载 MySQL JDBC 驱动类，触发其 `static` 代码块注册驱动。通过 `DriverManager` 获取数据库连接，内部会调用已注册的驱动程序的 `connect()` 方法。

- 配置文件加载

  Spring IoC 容器的本质可以类比为一个 HashMap，其中 Key 是 Bean 的名称，Value 是实例化的对象。Spring 通过反射动态加载 Bean：

  - 解析 `beans.xml` 或 `@Configuration` 类，获取需要实例化的类名。
  - 通过 `Class.forName()` 反射加载类，并使用`Constructor.newInstance()` 创建对象。
  - 通过 `Field.set()` 或 方法反射调用（setter、构造器）进行依赖注入。
  - `BeanPostProcessor` 允许在 Bean 初始化前后 进行额外处理，如AOP 代理等。

# 注解

注解可以作用在类上、方法上、字段上。注解的本质时继承了Annotation的特殊接口，具体实现时java运行时生成的动态代理。

# 异常

## 常见异常

错误和异常均继承于Throwable类

- 错误Error：OutOfMemoryError、StackOverflowError
- 异常Exception：
  - 非运行时异常：文件不存在、类未找到
  - 运行时异常：空指针、数组越界

## 异常处理

通过try-catch

```java
try {
    int result = 10 / 0; // 触发 ArithmeticException
} catch (ArithmeticException e) {
    System.out.println("捕获异常：" + e.getMessage());
} finally {
    System.out.println("无论如何都会执行的 finally 代码块！");
}
```

# object

## ==和equals

==对比的是对象的首地址是否一样，equals是对比字面值是否一样

## String、StringBuilder和StringBuffer

- String是不可变字符串，String对象是在jdk8之后是存储在堆中，具体存储位置取决于创建方式，常量创建放在字符串常量池，new出来放在普通对象中，`intern()`手动放入常量池。StringBuilder和StringBuffer也都创建在队中的普通对象里。

- StringBuffer线程安全，StringBuilder线程不安全
- 因为StringBuffer引入了线程安全，所以速度要慢于StringBuilder，但都大于String，因为String修改要频繁进行字符串常量池变更。

# 序列化

## 怎么把一个对象从一个jvm转移到另一个jvm

- 序列化与反序列化`Serializable`
- 使用中间件的消息传递机制
- 远程方法调用RPC（Remote Procedure Call）、Feign等
- 共享数据库

### 序列化和反序列化让你自己实现你会怎么做?

要么手动拼接JSON字符串，要么使用成熟的框架Fastjson、Jackson

```java
import com.alibaba.fastjson.JSON;
public class FastjsonExample {
    public static void main(String[] args) {
        MyClass obj = new MyClass("John", 30);
        String json = JSON.toJSONString(obj);// 序列化
        System.out.println(json);  // Output: {"age":30,"name":"John"}
        // 反序列化
        MyClass deserializedObj = JSON.parseObject(json, MyClass.class);
        System.out.println(deserializedObj.name);  // Output: John
        System.out.println(deserializedObj.age);   // Output: 30
    }
}
```

# 设计模式

## volatile和sychronized如何实现单例模式

`volatile` 关键字可以防止 JVM 对变量进行指令重排，`sychronized`是加锁

有两种实现方式

- 单sychronized，保证一个线程只会拿到一把锁

  ```java
  public class Singleton {
      private static Singleton instance;
      private Singleton() {}// 私有构造函数，防止外部实例化
      public static synchronized Singleton getInstance() {
          if (instance == null) {
              instance = new Singleton(); // 创建实例
          }
          return instance;
      }
  }
  ```

​	缺点：每个线程所对应的对象无论是否创建都会拿锁，性能较差

- volatile+更细粒度的锁

  ```java
  public class Singleton {
      private static volatile Singleton instance;// 使用 volatile 防止指令重排
      private Singleton() {} // 私有构造函数，防止外部实例化
      public static Singleton getInstance() {
          // 第一次检查，不加锁
          if (instance == null) {
              synchronized (Singleton.class) {
                  // 第二次检查，加锁
                  if (instance == null) {
                      instance = new Singleton(); // 创建实例
                  }
              }
          }
          return instance;
      }
  }
  ```

​	只有在第一次检查没有实例化的时候才会加锁，`volatile`保证顺序，更细粒度的锁`synchronized`保证性能。

## 代理模式和适配器模式有什么区别？

代理模式通过是增强功能，例如AOP来对目标方法进行增强，比如增加日志、事务、权限检查等。

适配器模式是为了适配不同场景不同应用。

# I/O

## BIO、和NIO、AIO区别

| 特性         | BIO                                | NIO                                         | AIO                              |
| ------------ | ---------------------------------- | ------------------------------------------- | -------------------------------- |
| **工作模式** | 阻塞 I/O，数据流模式               | 非阻塞 I/O，缓冲区 + 通道 + 选择器          | 异步 I/O，回调通知               |
| **线程模型** | 每个连接一个线程                   | 多个通道由单线程管理（通过 Selector）       | 异步 I/O，无需阻塞或轮询         |
| **I/O 操作** | 阻塞，直到 I/O 操作完成            | 非阻塞，可以轮询多个通道的事件              | 异步，不会阻塞，完成时回调通知   |
| **适用场景** | 并发连接数少，低性能需求           | 高并发、大量连接，I/O 密集型应用            | 超高并发、大数据量 I/O 操作      |
| **性能问题** | 并发连接数多时性能差（线程开销大） | 比 BIO 性能好，但需要轮询（高并发时更有效） | 性能非常高，尤其适用于高并发应用 |

## NIO如何实现

![1](.\01-java基础res\1.png)

每个客户端通过通道（Channel）与服务端进行数据交互，客户端通过端口向服务端发送连接请求。服务端使用一个线程，通过多路复用器（Selector）来监听多个客户端的连接请求和数据事件，服务端会将每个客户端的通道注册到 Selector 上进行管理。

## Netty

Netty 是建立在 Java NIO 之上的框架，它在底层使用 NIO 的 Selector（选择器）和非阻塞 I/O来处理并发的连接。与传统的 NIO 编程方式相比，有一下特点：

- 事件驱动和异步机制：通过回调机制处理I/O结果
- boss-worker：boss只负责管理，具体处理交给worker，还有更加智能的线程管理
- 内存管理优化：采用内存池化区别于传统每次读取都会分配新的字节数组
- 支持多种协议：不仅有tcp、udp还有http、https、websocket等等

# 其他

## 有一个学生类，想按照分数排序，再按学号排序，应该怎么做？

- 如果这个学生类是原生数组，直接用Array.sort配合内部函数实现自定义排序。

  ```java
  Arrays.sort(students,(a,b)->{
      if(a.getScore()!=b.getScore()){
          return b.getScore()-a.getScore();
      }else{
          return a.getId()-b.getId();
      }
  });
  ```

- 如果这个学生类是List集合，建议建议 `Student` 实现 `Comparable<Student>`以提供自定义排序

  ```java
  class Student implements Comparable<Student> {
      private int score;
      private int id;
  	... //构造、getter、setter
      @Override
      public int compareTo(Student other) {
          if (this.score != other.score) {
              return Integer.compare(other.score, this.score); // 降序
          }
          return Integer.compare(this.id, other.id); // 升序
      }
  }
  ```

## Native

在 Java 中，`native` 关键字用于声明本地方法（Native Method），这些方法的实现通常由 C 或 C++ 编写，并通过 JNI（Java Native Interface）调用。

用native声明的方法没有方法体

- native方法声明：

  ```java
  public class NativeExample {  
      public native void sayHello();// 声明一个 native 方法
      // 加载 C/C++ 库
      static {
          System.loadLibrary("NativeLib"); // 加载 "NativeLib.dll"（Windows）或 "libNativeLib.so"（Linux）
      }
      public static void main(String[] args) {
          new NativeExample().sayHello(); // 调用本地方法
      }
  }
  ```

- C/C++ 代码实现

  创建 `NativeExample.h`（通过 `javac` + `javah` 生成）

  ```java
  #include <jni.h>
  #include <stdio.h>
  #include "NativeExample.h"
  JNIEXPORT void JNICALL Java_NativeExample_sayHello(JNIEnv *env, jobject obj) {
      printf("Hello from C!\n");
  }
  ```

  编译成动态库：

  - **Windows**: `gcc -shared -o NativeLib.dll NativeExample.c -I"%JAVA_HOME%\include" -I"%JAVA_HOME%\include\win32"`
  - **Linux**: `gcc -shared -o libNativeLib.so NativeExample.c -I"$JAVA_HOME/include" -I"$JAVA_HOME/include/linux"`



















