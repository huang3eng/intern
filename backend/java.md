# Java

## 基础

### Java内存模型

> **是一种规范，Java虚拟机实现了这种规范。**屏蔽了各种硬件和操作系统的内存访问差异，让Java程序能够在各种硬件平台下，都能按照预期的方式运行。主要包括三块：Java内存模型的抽象结构、happen-before规则和对volatile内存语义的探讨。

- Java内存模型的抽象结构

  - Java线程对内存数据进行交互的规范

  ![img](https://tva1.sinaimg.cn/large/008i3skNgy1gs1g0xg9gfj31ju0u0wus.jpg)

- happen-before规则：写的代码只要在happen-before规则下，前一个操作的结果对后续操作是可见的，是不会发生重排序的。

- 对volatile内存语义的探讨：可见性和有序性(禁止重排序)

  - 使得编译器和CPU无法进行重排序，致使有序，并且写volatile变量对其他线程可见。

- 其他

  - [Java内存模型这块彻底玩儿明白了](https://www.bilibili.com/video/BV1F64y1B7sV/?spm_id_from=333.337.search-card.all.click&vd_source=f34c6fbf000c38e6ef25a6a0bdaa8453)

  - [深入浅出Java内存模型](https://javainterview.gitee.io/luffy/2021/08/19/02-Java%E5%B9%B6%E5%8F%91/09.%20%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAJava%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B/)

  - Java内存模型关注的是编程语言层面上，它是高维度的抽象。MESI是CPU缓存一致性协议，不同的CPU架构都不一样，可能有的CPU压根就没用MESI协议。

### Java内存结构

> 规定了运行时的数据区域，这是JVM「规范」的分区概念，到具体的实现落地，不同的厂商实现可能是有所区别的。

#### 五大分区

- 线程共享

  - 程序计数器：存放下一个指令的地址
  - 虚拟机栈：每次方法执行都会生成一个栈帧放入虚拟机栈
  - 本地方法栈：用于管理native（非java）方法的调用

  

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gs784qdq5sj314z0u04dh.jpg)

#### 1.7

- 自从在「JDK7」以后，就已经把「运行时常量池」和「静态常量池」转移到了「堆」内存中进行存储（对于「物理分区」来说「运行时常量池」和「静态常量池』就属于堆。

![img](https://pics6.baidu.com/feed/d31b0ef41bd5ad6e106966083e132ad2b4fd3ca5.png@f_auto?token=80373a2495ee3cdac594c5bc2518fd92)

#### 1.8

- 可以观察到元空间不在虚拟机中了

![img](https://pics7.baidu.com/feed/962bd40735fae6cd5a316d2ab36b1c2d40a70fd1.png@f_auto?token=5cd442fc7f91b6091fa20d9756b42707)

#### 主要变化

- 「JDK8」已经把「方法区」的实现从「永久代」变成「元空间」。「元空间」存储不在虚拟机中，而是使用本地内存，JVM 不会再出现方法区的内存溢出，以往「永久代」经常因为内存不够用导致跑出OOM异常。

![img](https://pic1.zhimg.com/v2-9c7846ec6af555435850d2687cccb4a0_b.jpg)

### 编译到执行

![img](https://tva1.sinaimg.cn/large/008i3skNgy1grx29rcwc5j30zq0gl0yt.jpg)

- 编译：将源码文件编译成JVM可以解释的class文件

  ![img](https://tva1.sinaimg.cn/large/008i3skNgy1gtvgejjhuhj61dy0bidhj02.jpg)

- 加载：将编译后的class文件加载到JVM中

  ![img](https://tva1.sinaimg.cn/large/008i3skNgy1gtvgkt95zyj61ig0r0q9f02.jpg)

  - 装载：查找并加载类的二进制数据，在JVM「堆」中创建一个java.lang.Class类的对象，并将类相关的信息存储在JVM「方法区」中
    - class文件是通过「类加载器」装载到jvm中的，为了防止内存中出现多份同样的字节码，使用了双亲委派机制。JDK 中的本地方法类一般由根加载器（Bootstrp loader）装载，JDK 中内部实现的扩展类一般由扩展加载器（ExtClassLoader ）实现装载，而程序中的类文件则由系统加载器（AppClassLoader ）实现装载。
  - 连接：对class的信息进行验证、为「类变量」分配内存空间并对其赋默认值。
    - 验证：验证类是否符合 Java 规范和 JVM 规范
    - 准备：为类的静态变量分配内存，初始化为系统的初始值
    - 解析：将符号引用转为直接引用的过程
  - 初始化：为类的静态变量赋予正确的初始值。

- 解释：把字节码转换为操作系统识别的指令。

  - 非热点代码直接进行解释。即时编译器把热点方法的指令码保存起来，下次执行的时候就无需重复的进行解释，直接执行缓存的机器语言。

  ![img](https://tva1.sinaimg.cn/large/008i3skNgy1gtvgm6zg5nj60u80begme02.jpg)

### 浮点数运算的时候会有精度丢失的风险

- 计算机是二进制的，而且计算机在表示一个数字时，宽度是有限的，无限循环的小数存储在计算机时，只能被截断，所以就会导致小数精度发生损失的情况。
- `BigDecimal` 可以实现对浮点数的运算，不会造成精度丢失

### 对象的相等和引用相等的区别

- 对象的相等一般比较的是内存中存放的内容是否相等。
- 引用相等一般比较的是他们指向的内存地址是否相等。

### 深拷贝和浅拷贝

- **浅拷贝**：浅拷贝会在堆上创建一个新的对象（区别于引用拷贝的一点），不过，如果原对象内部的属性是引用类型的话，浅拷贝会直接复制内部对象的引用地址，也就是说拷贝对象和原对象共用同一个内部对象。
- **深拷贝** ：深拷贝会完全复制整个对象，包括这个对象所包含的内部对象。

### == 和 equals() 的区别

#### ==

- 对于基本数据类型来说，`==` 比较的是值。
- 对于引用数据类型来说，`==` 比较的是对象的内存地址。

#### equals()

- **类没有重写 `equals()`方法** ：通过`equals()`比较该类的两个对象时，等价于通过“==”比较这两个对象，使用的默认是 `Object`类`equals()`方法。
- **类重写了 `equals()`方法** ：一般我们都重写 `equals()`方法来比较两个对象中的属性是否相等；若它们的属性相等，则返回 true(即，认为这两个对象相等)。
- `String` 中的 `equals` 方法是被重写过的，因为 `Object` 的 `equals` 方法是比较的对象的内存地址，而 `String` 的 `equals` 方法比较的是对象的值。
- `hashCode()` 和 `equals()`都是用于比较两个对象是否相等。有了 `hashCode()` 之后，判断元素是否在对应容器中的效率会更高。但是两个对象有相同的 `hashCode` 值，它们也不一定是相等。

### String

- 操作少量的数据: 适用 `String`
- 单线程操作字符串缓冲区下操作大量数据: 适用 `StringBuilder`
- 多线程操作字符串缓冲区下操作大量数据: 适用 `StringBuffer`
- 字符串对象通过“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个 `String` 对象 。在循环内使用“+”进行字符串的拼接的话，存在比较明显的缺陷：**编译器不会创建单个 `StringBuilder` 以复用，会导致创建过多的 `StringBuilder` 对象**。
- `String` 中的 `equals` 方法是被重写过的，比较的是 String 字符串的值是否相等。 `Object` 的 `equals` 方法是比较的对象的内存地址。

### try、catch

- **不要在 finally 语句块中使用 return!** 当 try 语句和 finally 语句中都有 return 语句时，try 语句块中的 return 语句会被忽略。这是因为 try 语句中的 return 返回值会先被暂存在一个本地变量中，当执行到 finally 语句中的 return 之后，这个本地变量的值就变为了 finally 语句中的 return 返回值。
-  finally 之前虚拟机被终止运行的话、程序所在的线程死亡和关闭 CPU，finally 中的代码就不会被执行
- 使用 `try-with-resources` 代替`try-catch-finally`
  - 任何实现 `java.lang.AutoCloseable`或者 `java.io.Closeable` 的对象。
  - 在 `try-with-resources` 语句中，任何 catch 或 finally 块在声明的资源关闭后运行。

### spi

> https://zhuanlan.zhihu.com/p/84337883

- 通过 URL 工具类从 jar 包的 `/META-INF/services` 目录下面找到对应的文件，
- 读取这个文件的名称找到对应的 spi 接口
- 通过 `InputStream` 流将文件里面的具体实现类的全类名读取出来，
- 根据获取到的全类名，先判断跟 spi 接口是否为同一类型，如果是的，那么就通过反射的机制构造对应的实例对象
- 将构造出来的实例对象添加到 `Providers` 的列表中

### 序列化

- 对于 Java 这种面向对象编程语言来说，我们序列化的都是对象（Object）也就是实例化后的类(Class)
- 使用 `transient` 关键字修饰的字段和`static` 变量不会被序列化

### IO流

- `InputStream`/`Reader`: 所有的输入流的基类，前者是字节输入流，后者是字符输入流。
- `OutputStream`/`Writer`: 所有输出流的基类，前者是字节输出流，后者是字符输出流。
- 为什么还要有字符流
  - 字符流是由 Java 虚拟机将字节转换得到的，这个过程还算是比较耗时；
  - 如果我们不知道编码类型的话，使用字节流的过程中很容易出现乱码问题。

### 代理

> 使用代理对象来代替对真实对象(real object)的访问，这样就可以在不修改原目标对象的前提下，提供额外的功能操作，扩展目标对象的功能。

#### 静态代理

```java
public interface SmsService {
    String send(String message);
}

public class SmsServiceImpl implements SmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}

public class SmsProxy implements SmsService {

    private final SmsService smsService;

    public SmsProxy(SmsService smsService) {
        this.smsService = smsService;
    }

    @Override
    public String send(String message) {
        //调用方法之前，我们可以添加自己的操作
        System.out.println("before method send()");
        smsService.send(message);
        //调用方法之后，我们同样可以添加自己的操作
        System.out.println("after method send()");
        return null;
    }
}

--------------------------------------------
SmsService smsService = new SmsServiceImpl();
SmsProxy smsProxy = new SmsProxy(smsService);
smsProxy.send("java");
```



#### JDK动态代理

> 使用InvocationHandler给被代理类的某个方法添加一些功能。然后使用Proxy.newProxyInstance动态生成代理对象。
>
> JDK动态代理会帮我们实现接口的方法，通过invokeHandler对所需要的方法进行增强。
>
> **JDK 动态代理有一个最致命的问题是其只能代理实现了接口的类。**

```java

// 接口和实现类
public interface SmsService {
    String send(String message);
}

public class SmsServiceImpl implements SmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}

public class DebugInvocationHandler implements InvocationHandler {
    /**
     * 代理类中的真实对象
     */
    private final Object target;

    public DebugInvocationHandler(Object target) {
        this.target = target;
    }


    public Object invoke(Object proxy, Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
        //调用方法之前，我们可以添加自己的操作
        System.out.println("before method " + method.getName());
        Object result = method.invoke(target, args);
        //调用方法之后，我们同样可以添加自己的操作
        System.out.println("after method " + method.getName());
        return result;
    }
}

// 获取代理对象的工厂类
public class JdkProxyFactory {
    public static Object getProxy(Object target) {
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(), // 目标类的类加载
                target.getClass().getInterfaces(),  // 代理需要实现的接口，可指定多个
                new DebugInvocationHandler(target)   // 代理对象对应的自定义 InvocationHandler
        );
    }
}

--------------------------------------------
  
SmsService smsService = (SmsService) JdkProxyFactory.getProxy(new SmsServiceImpl());
smsService.send("java");


```

#### CGLIB 动态代理

```java
public class AliSmsService {
    public String send(String message) {
        System.out.println("send message:" + message);
        return message;
    }
}

public class DebugMethodInterceptor implements MethodInterceptor {
    /**
     * @param o           被代理的对象（需要增强的对象）
     * @param method      被拦截的方法（需要增强的方法）
     * @param args        方法入参
     * @param methodProxy 用于调用原始方法
     */
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        //调用方法之前，我们可以添加自己的操作
        System.out.println("before method " + method.getName());
        Object object = methodProxy.invokeSuper(o, args);
        //调用方法之后，我们同样可以添加自己的操作
        System.out.println("after method " + method.getName());
        return object;
    }
}

public class CglibProxyFactory {

    public static Object getProxy(Class<?> clazz) {
        // 创建动态代理增强类
        Enhancer enhancer = new Enhancer();
        // 设置类加载器
        enhancer.setClassLoader(clazz.getClassLoader());
        // 设置被代理类
        enhancer.setSuperclass(clazz);
        // 设置方法拦截器
        enhancer.setCallback(new DebugMethodInterceptor());
        // 创建代理类
        return enhancer.create();
    }
}
-------------------------
AliSmsService aliSmsService = (AliSmsService) CglibProxyFactory.getProxy(AliSmsService.class);
aliSmsService.send("java");
```

### 并发

> - 能不能保证操作的原子性，考虑atomic包下的类够不够我们使用。
> - 能不能保证操作的可见性，考虑volatile关键字够不够我们使用
> - 如果涉及到对线程的控制（比如一次能使用多少个线程，当前线程触发的条件是否依赖其他线程的结果），考虑CountDownLatch/Semaphore等等。
> - 如果是集合，考虑java.util.concurrent包下的集合类。
> - 如果synchronized无法满足，考虑lock包下的类

#### ConcurrentHashMap

- [面试 ConcurrentHashMap ，看这一篇就够了！](https://zhuanlan.zhihu.com/p/350099474)

#### sleep和wait

- `wait()` 要释放当前线程占有的对象锁并让其进入 WAITING 状态。 `sleep()` 是让当前线程暂停执行，不涉及到对象类，也不需要获得对象锁。所以前者定义在`Object`类中，后者定义在`Thread`类中。

#### 单例实现

```java
public class Singleton {
  	// volatile关键字防止指令重排
    private volatile static Singleton uniqueInstance;
		
  	// 私有化构造方法
    private Singleton() {
    }
	
    public  static Singleton getUniqueInstance() {
       //先判断对象是否已经实例过，没有实例化过才进入加锁代码。提高效率，不是每次先上锁才判断是否初始化过
        if (uniqueInstance == null) {
            //类对象加锁，确保只有一个线程可以进去，并创建实例。
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

#### volatile

- 可以保证变量的可见性，还有一个重要的作用就是防止 JVM 的指令重排序。并不保证原子性。可以再加上利用 `synchronized` 、`Lock`或者`AtomicInteger`来保证原子性。

#### synchronized

- 只有一个线程进入临界区，偏向锁
- 多个线程交替进入临界区，轻量级锁
- 多线程同时进入临界区，重量级锁

#### 锁

- AQS(AbstractQueuedSynchronizer)：内部实现的关键就是维护了一个先进先出的队列以及state状态变量

- 非公平锁：线程执行同步代码块时，会尝试去获取锁

<img src="../../../Downloads/IMG_21B4368683D9-1.jpeg" alt="IMG_21B4368683D9-1" style="zoom:50%;" />

- 公平锁：不会尝试获取锁，直接进队列，再等待唤醒

- synchronized无论处理哪种锁（偏向锁、轻量级锁、重量级锁），都是先尝试获取，获取不到才升级放到队列上的，所以是非公平的。

- 死锁

  ![IMG_7899388934CB-1](../../../Downloads/IMG_7899388934CB-1.jpeg)

#### 线程池

- ThreadPoolExecutor在构造的时候有几个重要的参数：corePoolSize（核心线程数量）、maximumPoolSize（最大线程数量）、keepAliveTime（线程空余时间）、workQueue（阻塞队列）、handler（任务拒绝策略）
  - 首先会判断运行线程数是否小于corePoolSize，如果小于，则直接创建新的线程执行任务
  - 如果大于corePoolSize，判断workQueue阻塞队列是否已满，如果还没满，则将任务放到阻塞队列中
  - 如果workQueue阻塞队列已经满了，则判断当前线程数是否大于maximumPoolSize，如果没大于则创建新的线程执行任务
  - 如果大于maximumPoolSize，则执行任务拒绝策略（具体就是你自己实现的handler）

#### ThreadLocal

> 提供了线程的局部变量，每个线程都可以通过set/get来对这个局部变量进行操作

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gtt5mgz1e6j61bw0u0adp02.jpg)

- 内存泄露问题：ThreadLocalMap的key是弱引用指向ThreadLocal，弱引用在gc的时候容易被回收，那么就会出现key为null的情况下，此时value是强引用没有被回收。在使用完ThreadLocal的时候，调用下remove方法，会清除key为null的value。

### 注解

#### 基本概念

- 元Annotation（元注解）：所谓的元Annotation就是用来修饰注解的
  - `@Retention`注解可以简单理解为设置注解的生命周期
    - SOURCE代表着注解仅保留在源级别中，并由编译器忽略。CLASS代表着注解在编译时由编译器保留，但Java虚拟机（JVM）会忽略。RUNTIME代表着标记的注解会由JVM保留，因此运行时环境可以使用它。
    - 如果你想要在编译期间处理注解相关的逻辑，你需要继承AbstractProcessor 并实现process方法。
  - `@Target`表示这个注解可以修饰哪些地方（比如方法、还是成员变量、还是包等等）

#### AspectJ

- 使用注解`@Aspect`定义切面，然后定义切入点`Pointcut`

- 切入点分类

  ![image-20221126154946399](https://cdn.jsdelivr.net/gh/huang3eng/mdpic@master/uPic/image-20221126154946399.png)

- 常见应用：埋点、登录统一处理、异常统一处理

### 泛型

- 泛型的信息只存在编译阶段，在class字节码就看不到泛型的信息了

- 泛型擦除是有范围的，定义在类上的泛型信息是不会被擦除的

## 1.8

### Stream

> 创建流 -> 转化流 -> 聚合流

- forEach()遍历集合中的对象
- filter对流对象进行过滤
- map 方法用于映射每个元素到对应的结果
- count是对流数据统计的方法，但是count之后返回的是long类型，所以无法再进行流操作
- limit选取流数据的前多少条数据
- skip跳过流数据的前多少条数据
- sorted用于对流进行排序
- parallel用于并行操作
- Stream的静态方法concat()实现对两个流数据进行合并

### Consumer、Supplier、Predicate和Function 

#### Consumer

- lambda 表达式、方法引用的返回值都是 **Consumer 类型**，所以，他们能够作为 `forEach` 方法的参数，并且输出一个值。
-  Consumer是一个接口，并且只要实现一个 `accept` 方法，就可以作为一个**“消费者”**输出信息。

#### Supplier

- Supplier 接口可以理解为一个容器，用于装数据的。
- Supplier 接口有一个 `get` 方法，可以返回值。

## Guava

> [中文教程](https://wizardforcel.gitbooks.io/guava-tutorial/content/3.html)

## Log4j2