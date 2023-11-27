# JDK

## 面向对象OOP六原则

1. 单一职责原则（Single responsibility principle）一个类只负责一项职责；
2. 里氏替换原则（Liskov Substituition Principle）：引用基类的地方必须能透明地使用其子类的对象，也就是说子类可以扩展父类的功能，但不能改变父类原有的功能；
3. 依赖倒置原则（Dependence Inversion Principle）：设计要依赖于抽象而不是具体化（尽量面向接口编程）；
4. 接口隔离原则（Interface Segregation Principle）：客户端不应该依赖它不需要的接口、一个类对另一个类的依赖应该建立在最小的接口上、接口最小化-过于臃肿的接口依据功能,可以将其拆分为多个接口；
5. 迪米特法则或最少知识原则（Law of Demeter or Least Knowlegde Principle）：一个对象应该对其他对象保持最少的了解,简单的理解就是高内聚,低耦合，一个类尽量减少对其他对象的依赖，并且这个类的方法和属性能用私有的就尽量私有化；
6. 开闭原则（Open-Close Principle）：对扩展开放，对修改关闭。

## Serializable

1. 类通过实现**Serializable**接口以启用其序列化功能；
2. **Externalizable**接口继承了**Serializable**接口，实现**Externalizable**接口也可以开启序列化功能但是要重写`writeExternal()`和`readExternal()`方法，否则反序列化后对象属性都为null；
3. 被**transient**关键字修饰的成员变量，在序列化时时会被忽略，在被反序列化后，**transient**变量的值被设置为初始值。如int的初始值为0，对象型为null；
4. **ArrayList**中`transient Object[] elementData`被**transient**修饰但是数据还是正常序列化反序列化了是因为在**ArrayList**中定义了两个方法：`writeObject()`和`readObject()`（这两个方式是反射调用的），在序列化过程中，如果被序列化的类中定义了`writeObject()`和`readObject()`方法，那么虚拟机会试图调用对象类中的`writeObject()`和`readObject()`实现进行用户自定义的序列化和反序列化，如果没有这两个方法，则默认调用的是**ObjectOutPutStream**的`defaultWriteObject()`方法和**ObjectInputStream**的`defaultReadObject()`方法，用户自定义的writeObject()和readObject()方法允许用户控制序列化的过程，比如可以在序列化的过程中动态改变序列化的数值，很灵活，**ArrayList**这样做是为了**不对null元素进行序列化**；
5. **serialVersionUID**是用来标记代码版本的，实现**Serializable**接口的类必须定义**serialVersionUID**，如果没有的话会根据属性自动生成，如果改变了累的属性，反序列化的时候会报错**serialVersionUID**不一致；
6. <span id = "序列化会破坏单例模式">序列化会破坏单例模式</span>，解决办法是在单例类中定义`readResolve()`方法，`readResolve()`方法中调用单例方法。

## Jdk中有哪些锁

大面上分是悲观锁和乐观锁

- 悲观锁：悲观的认为每次操作都不安全，所以直接加锁，性能差
- 乐观锁：乐观地不加锁，操作完再去对比，CAS，Compare And Set，compare单独的字段如version或时间戳

## IO多路复用

## <span id="ThreadLocal">ThreadLocal</span>

线程存放自己独享变量的一个空间

引用链ThreadLocal—>ThreadLocalMap—>Entry（若引用extends）

Entry是key，value对，key是当前ThreadLocal的弱引用，value是强引用

gc时会比较容易清理掉key，key为null时又找不到value了，容易内存泄漏

线程结束会结束引用，但是实际工作中都是线程池，最好不用了就调用remove();

## CountDownLatch和CyclicBarrier的区别

- CountDownLatch：**等其他**。一个或者一组线程在开始执行操作之前，必须要等到其他线程执行完才可以。我们举一个例子来说明，在考试的时候，老师必须要等到所有人交了试卷才可以走。此时老师就相当于等待线程，而学生就好比是执行的线程。
- CyclicBarrier：**互相等**。和CountDownLatch类似。同样是等待其他线程都完成了，才可以进行下一步操作，我们再举一个例子，在打王者的时候，在开局前所有人都必须要加载到100%才可以进入。否则所有玩家都相互等待。

## AQS

AbstractQueuedSynchronizer

## 设计模式 Design Patterns

- 单例模式 Singleton：
  1. 饿汉式：类加载时就初始化属性；
  2. 懒汉式 Lazy Loading：线程不安全，加synchronize可解决但效率下降；
  3. 懒汉式 Lazy Loading：synchronize关键字 + 双重非空校验 + volatile关键字；
  4. 静态内部类：加载时不会加载内部类，可以实现懒加载，同时JVM能保证单例；
  5. 枚举：集合所有优点，且还能防止反序列化（普通方式的单例模式如何防止反序列化破坏单例，参考[这里](#序列化会破坏单例模式)）。
- 观察者模式 Observer
  1. 封装、继承、多态；
  2. 被观察者、事件、观察者；
  3. 事件持有被观察者，被观察者持有观察者。

## 线程池

线程池中的七大参数：

1. **corePoolSize**：线程池中的常驻核心线程数。

   数量设定：

   **cpu密集型任务**建议：为cpu核心数+1，太多的话会带来额外的线程切换损耗；

   **io密集型任务**建议：io阻塞时间越长的话线程数可以更多，2 x cpu核心数或更多，参考线程数 = cpu核心数 x （1 +  线程等待时间 / 线程任务运行总时间）

   运行时间可以通过**VisualVM**查看，计算公式是基于理想环境，没有其他线程影响，现实环境很难达到因为有很多别的工作线程，垃圾回收、操作系统啥的。

2. **maximumPoolSize**：线程池能够容纳同时执行的最大线程数，此值大于等于1。

3. **keepAliveTime**：多余的空闲线程存活时间，当空间时间达到keepAliveTime值时，多余的线程会被销毁直到只剩下corePoolSize个线程为止。

4. **unit：keepAliveTime**的单位。

5. **workQueue**：任务队列，被提交但尚未被执行的任务。

6. **threadFactory**：表示生成线程池中工作线程的线程工厂，用户创建新线程，一般用默认即可。

7. **handler**：拒绝策略，表示当线程队列满了并且工作线程大于等于线程池的最大显示数(maxnumPoolSize)时如何来拒绝请求执行的runnable的策略。

工作流程分析

- 线程池中线程数小于corePoolSize时，新任务将创建一个新线程执行任务，不论此时线程池中存在空闲线程；
- 线程池中线程数达到corePoolSize时，新任务将被放入workQueue中，等待线程池中任务调度执行；
- 当workQueue已满，且maximumPoolSize>corePoolSize时，新任务会创建新线程执行任务；
- 当workQueue已满，且提交任务数超过maximumPoolSize，任务由RejectedExecutionHandler处理；
- 当线程池中线程数超过corePoolSize，且超过这部分的空闲时间达到keepAliveTime时，回收该线程；
- 如果设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize范围内的线程空闲时间达到keepAliveTime也将回收。

​	![image-20230824130655782](.\pic\image-20230824130655782.png)

# JVM

## 调优

# Spring

## 事务传播特性

spring支持7种事务传播行为，分别为：

1. **propagation_requierd**：默认的事务传播级别，如果上下文中已经存在事务，那么就加入到事务中执行，如果当前上下文中不存在事务，则新建事务执行。
2. **propagation_supports**：如果上下文存在事务，则加入到事务执行，如果没有事务，则使用非事务的方式执行。

3. **propagation_mandatory**：上下文中必须要存在事务，否则就会抛出异常。

4. **propagation_required_new**：每次都会新建一个事务，如果上下文中有事务，则将上下文的事务挂起，当新建事务执行完成以后，上下文事务再恢复执行。

5. **propagation_not_supported**：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

6. **propagation_never**：以非事务方式执行操作，如果当前事务存在则抛出异常。

7. **propagation_nested**：嵌套事务。如果上下文中存在事务，则嵌套事务执行，如果不存在事务，则新建事务。

说明：

Spring 默认的事务传播行为是 PROPAGATION_REQUIRED，它适合于绝大多数的情况。假设 ServiveY#methodY() 都工作在事务环境下（即都被 Spring 事务增强了），假设程序中存在如下的调用链：Service1#method1()->Service2#method2()->Service3#method3()，那么这 3 个服务类的 3 个方法通过 Spring 的事务传播机制都工作在同一个事务中。

## Spring IoC 的容器构建流程（8分）

![image-20230816112447783](C:.\pic\image-20230816112447783.png)

## Bean的生命周期

bean 的生命周期主要有以下几个阶段，深色底的5个是比较重要的阶段。

![image-20230816112941685](.\pic\image-20230816112941685.png)

## BeanFactory 和 FactoryBean 的区别（6分）

BeanFactory：Spring 容器最核心也是最基础的接口，本质是个工厂类，用于管理 bean 的工厂，最核心的功能是加载 bean，也就是 getBean 方法，通常我们不会直接使用该接口，而是使用其子接口。

FactoryBean：该接口以 bean 样式定义，但是它不是一种普通的 bean，它是个工厂 bean，实现该接口的类可以自己定义要创建的 bean 实例，只需要实现它的 getObject 方法即可。

FactoryBean 被广泛应用于 Java 相关的中间件中，如果你看过一些中间件的源码，一定会看到 FactoryBean 的身影。

一般来说，都是通过 FactoryBean#getObject 来返回一个代理类，当我们触发调用时，会走到代理类中，从而可以在代理类中实现中间件的自定义逻辑，比如：RPC 最核心的几个功能，选址、建立连接、远程调用，还有一些自定义的监控、限流等等。

## BeanFactory 和 ApplicationContext 的区别（6分）

BeanFactory：基础 IoC 容器，提供完整的 IoC 服务支持。

ApplicationContext：高级 IoC 容器，BeanFactory 的子接口，在 BeanFactory 的基础上进行扩展。包含 BeanFactory 的所有功能，还提供了其他高级的特性，比如：事件发布、国际化信息支持、统一资源加载策略等。正常情况下，我们都是使用的 ApplicationContext。

![image-20230816152817870](.\pic\image-20230816152817870.png)

这边以电话来举个简单的例子：我们家里使用的 “座机” 就类似于 BeanFactory，可以进行电话通讯，满足了最基本的需求。

而现在非常普及的智能手机，iPhone、小米等，就类似于 ApplicationContext，除了能进行电话通讯，还有其他很多功能：拍照、地图导航、听歌等。

## Spring 的 AOP 是怎么实现的（5分）

本质是通过动态代理来实现的，主要有以下几个步骤。

1、获取增强器，例如被 Aspect 注解修饰的类。

2、在创建每一个 bean 时，会检查是否有增强器能应用于这个 bean，简单理解就是该 bean 是否在该增强器指定的 execution 表达式中。如果是，则将增强器作为拦截器参数，使用动态代理创建 bean 的代理对象实例。

3、当我们调用被增强过的 bean 时，就会走到代理类中，从而可以触发增强器，**本质跟拦截器类似**。

## 多个AOP的顺序怎么定（6分）

通过 Ordered 和 PriorityOrdered 接口进行排序。PriorityOrdered 接口的优先级比 Ordered 更高，如果同时实现 PriorityOrdered 或 Ordered 接口，则再按 order 值排序，值越小的优先级越高。

## Spring 的 AOP 有哪几种创建代理的方式（9分）

Spring 中的 AOP 目前支持 JDK 动态代理和 Cglib 代理。

通常来说：如果被代理对象实现了接口，则使用 JDK 动态代理，否则使用 Cglib 代理。另外，也可以通过指定 proxyTargetClass=true 来实现强制走 Cglib 代理。

## JDK 动态代理和 Cglib 代理的区别（9分）

1. JDK 动态代理本质上是实现了被代理对象的接口，而 Cglib 本质上是继承了被代理对象，覆盖其中的方法。
2. JDK 动态代理只能对实现了接口的类生成代理，Cglib 则没有这个限制。但是 Cglib 因为使用继承实现，所以 Cglib 无法代理被 final 修饰的方法或类。

3. 在调用代理方法上，JDK 是通过反射机制调用，Cglib是通过FastClass 机制直接调用。FastClass 简单的理解，就是使用 index 作为入参，可以直接定位到要调用的方法直接进行调用。
4. 在性能上，JDK1.7 之前，由于使用了 FastClass 机制，Cglib 在执行效率上比 JDK 快，但是随着 JDK 动态代理的不断优化，从 JDK 1.7 开始，JDK 动态代理已经明显比 Cglib 更快了。

## JDK 动态代理为什么只能对实现了接口的类生成代理（7分）

根本原因是通过 JDK 动态代理生成的类已经继承了 Proxy 类，所以无法再使用继承的方式去对类实现代理。

## Spring 的事务隔离级别是如何做到和数据库不一致的？（5分）

比如数据库是可重复读，Spring 是读已提交，这是怎么实现的？

Spring 的事务隔离级别底层其实是基于数据库的，Spring 并没有自己的一套隔离级别。

DEFAULT：使用数据库的默认隔离级别。

READ_UNCOMMITTED / READ_COMMITTED / REPEATABLE_READ / SERIALIZABLE

Spring 的事务隔离级别本质上还是通过数据库来控制的，具体是在执行事务前先执行命令修改数据库隔离级别，命令格式如下：

```
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED
```

## **Spring 事务的实现原理（8分）**

Spring 事务的底层实现主要使用的技术：**AOP（动态代理）** + **ThreadLocal** + **try/catch**。

动态代理：基本所有要进行逻辑增强的地方都会用到动态代理，AOP 底层也是通过动态代理实现。

[ThreadLocal](#ThreadLocal)：主要用于线程间的资源隔离，以此实现不同线程可以使用不同的数据源、隔离级别等等。

try/catch：最终是执行 commit 还是 rollback，是根据业务逻辑处理是否抛出异常来决定。

Spring 事务的核心逻辑伪代码如下：

```java
public void invokeWithinTransaction() {
    // 1.事务资源准备
    try {
        // 2.业务逻辑处理，也就是调用被代理的方法
    } catch (Exception e) {
        // 3.出现异常，进行回滚并将异常抛出
    } finally {
        // 现场还原：还原旧的事务信息
    }
    // 4.正常执行，进行事务的提交
    // 返回业务逻辑处理结果
}
```

详细流程如下图所示：

![image-20230816155619413](.\pic\image-20230816155619413.png)

## Spring 怎么解决循环依赖的问题（9分）

Spring 是通过提前暴露 bean 的引用来解决的，具体如下：

Spring 首先使用构造函数创建一个 “不完整” 的 bean 实例（之所以说不完整，是因为此时该 bean 实例还未初始化），并且提前曝光该 bean 实例的ObjectFactory（提前曝光就是将 ObjectFactory 放到 singletonFactories 缓存）.

通过 ObjectFactory 我们可以拿到该 bean 实例的引用，如果出现循环引用，我们可以通过缓存中的 ObjectFactory 来拿到 bean 实例，从而避免出现循环引用导致的死循环。


举个例子：A 依赖了 B，B 也依赖了 A，那么依赖注入过程如下。

检查 A 是否在缓存中，发现不存在，进行实例化

通过构造函数创建 bean A，并通过 ObjectFactory 提前曝光 bean A

A 走到属性填充阶段，发现依赖了 B，所以开始实例化 B。

首先检查 B 是否在缓存中，发现不存在，进行实例化

通过构造函数创建 bean B，并通过 ObjectFactory 曝光创建的 bean B

B 走到属性填充阶段，发现依赖了 A，所以开始实例化 A。

检查 A 是否在缓存中，发现存在，拿到 A 对应的 ObjectFactory 来获得 bean A，并返回。

B 继续接下来的流程，直至创建完毕，然后返回 A 的创建流程，A 同样继续接下来的流程，直至创建完毕。

这边通过缓存中的 ObjectFactory 拿到的 bean 实例虽然拿到的是 “不完整” 的 bean 实例，但是由于是单例，所以后续初始化完成后，该 bean 实例的引用地址并不会变，所以最终我们看到的还是完整 bean 实例。

## Spring 能解决构造函数循环依赖吗（6分）

答案是不行的，对于使用构造函数注入产生的循环依赖，Spring 会直接抛异常。

为什么无法解决构造函数循环依赖？

上面解决逻辑的第一句话：“首先使用构造函数创建一个 “不完整” 的 bean 实例”，从这句话可以看出，构造函数循环依赖是无法解决的，因为当构造函数出现循环依赖，我们连 “不完整” 的 bean 实例都构建不出来。

# Database

## MySQL存储引擎



## MySQL数据库多少数据才考虑拆分数据表table

什么样的mysql数据表table需要拆分：

通常我们根据表的体积、表的行数、访问特点来衡量表是否需要拆分

一般数据表拆分标准如下：

1. 表的体积大于2G或行数大于1000w,以单表主键等简单形式访问数据，这个时候需要分表

2. 表的体积大于2G或行数大于500W,以两表jion，小范围查询（结果集小100行）等形式访问数据，这个时候需要分表

3. 表的体积大于2G或行数大于200w,以多表join，范围查询，order by，group by，高频率等复杂形式访问数据，尤其DML，这个时候需要分表

4. 表的字段中含有text等大字段的、varchar(500)以上的、很少使用的字符型字段拆分成父子表，这种分表可以和以上联合使用

5. 数据有时间过期特性的，需要做数据分表归档处理

只要达到上面任何一个标准，都需要做分表处理

## 数据库事务的四个特性ACID

**ACID**，是指[数据库管理系统](https://baike.baidu.com/item/数据库管理系统?fromModule=lemma_inlink)（[DBMS](https://baike.baidu.com/item/DBMS?fromModule=lemma_inlink)）在写入或更新资料的过程中，为保证[事务](https://baike.baidu.com/item/事务?fromModule=lemma_inlink)（transaction）是正确可靠的，所必须具备的四个特性：[原子性](https://baike.baidu.com/item/原子性?fromModule=lemma_inlink)（atomicity，或称不可分割性）、[一致性](https://baike.baidu.com/item/一致性?fromModule=lemma_inlink)（consistency）、[隔离性](https://baike.baidu.com/item/隔离性?fromModule=lemma_inlink)（isolation，又称独立性）、[持久性](https://baike.baidu.com/item/持久性?fromModule=lemma_inlink)（durability）。

- Atomicity（原子性）：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被恢复（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- Consistency（一致性）：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。（也可以理解为Correctness，表明事务中的操作是正确的，比如符合表约束啥的）
- Isolation（隔离性）：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。
- Durability（持久性）：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

## MySQL数据库事务的隔离级别

```sql
select @@transaction_isolation; # 当前隔离级别
select @@autocommit; # 自动提交：1为开启、0为关闭
```

先复习一些概念

- 脏读：指当前事务读取到另外一个事务未提交的数据
- 不可重复读：指在一个事务中对同一条数据读取两次但是结果不同
- 幻读/续读：在一个事务中同样的查询条件，查出了不同条数的数据

接下来说事务隔离级别

- RU	Read—Uncommitted（读未提交）

​		可以读到其他事务未提交的数据，隔离性差，会出现脏读（当前内存读）、不可重复读、幻读。

- RC	Read—Committed（读已提交）

​		可以读到其他事务已提交的数据，隔离性一般，不会出现脏读问题，但是会出现不可重复读、幻读。

- RR	Repeatable—Read（可重复读）(MySQL默认级别)

​		可以防止脏读（当前内存读），防止不可重复读问题，防止会出现的幻读问题，但是并发能力较差；

​		会使用next lock锁进制，来防止幻读问题，但是引入锁进制后，锁的代价会比较高，比较耗费CPU资源，占用系统性能；

- SR	Serializable （串行化/序列化）

​		隔离性比较高，可以实现串行化读取数据，但是事务的并发度就没有了；
​		这是事务的最高级别，在每条读的数据上，加上锁，使之不可能相互冲突。

一些验证：

在解释分析说明相应的隔离级别名词前，需要对数据库事务隔离级别进行调整，以及关闭自动提交功能：

```sql
SELECT @@transaction_isolation; -- 查看当前事务隔离级别
SELECT @@autocommit; -- 查看当前是否自动提交：1是0否
show variables like 'autocommit'; -- 查看当前是否自动提交：ON是OFF否
set GLOBAL autocommit=1; 
-- 注意上面这条命令如果是在一些数据库连接工具使用这条命令的话有可能没效果因为这些工具需要去设置中更改，另外不加global也行，确保本次会话不自动提交就行了
set GLOBAL transaction_isolation='READ-UNCOMMITTED';
set GLOBAL transaction_isolation='READ-COMMITTED';
set GLOBAL transaction_isolation='REPEATABLE-READ';
set GLOBAL transaction_isolation='SERIALIZABLE';
```

数据准备：

```sql
use test;
create table t1 ( id int not null primary key auto_increment,
a int not null,
b varchar(20) not null,
c varchar(20) not null) charset = utf8mb4 engine = innodb;

begin;
insert into t1(a,b,c)
values
(5,'a','aa'),
(7,'c','ab'),
(10,'d','ae'),
(13,'g','ag'),
(14,'h','at'),
(16,'i','au'),
(20,'j','av'),
(22,'k','aw'),
(25,'l','ax'),
(27,'o','ay'),
(31,'p','az'),
(50,'x','aze'),
(60,'y','azb');
commit;
-- 最后确认一下两个SQL会话窗口，即不同的事务查看的数据、事务隔离级别、自动提交设置是否一致
```

开始验证：

- 脏读

  - RU级别下

    ```sql
    -- 数据库A会话窗口操作
    mysql> begin;
    mysql> update t1 set a=10 where id=1;-- 只是在内存层面进行数据页中数据修改
    -- 此时进行B会话操作
    mysql> rollback;-- 进行事务回滚操作
    
    -- 数据库B会话窗口操作
    mysql> begin;
    mysql> select * from t1 where id=1;
    +----+----+---+----+
    | id   | a   | b  | c   |
    +----+----+---+----+
    |  1   | 10 | a  | aa |
    +----+----+---+----+
    1 row in set (0.01 sec)-- 在A会话窗口没提交的事务修改，被B会话窗口查询到了
    -- 此时进行A会话回滚
    mysql> select * from t1 where id=1;
    +----+----+---+----+
    | id   | a   | b  | c   |
    +----+----+---+----+
    |  1   | 5   | a  | aa |
    +----+----+---+----+
    1 row in set (0.01 sec)-- 在A会话窗口进行回滚后，在B窗口查询的数据又恢复了
    ```
    
    

- 不可重复读

  - RU级别下（同RU级别下的脏读）

  - RC级别下

    ```sql
    -- 数据库A会话窗口操作
    mysql> use test;
    mysql> begin;mysql> select * from t1 where id=1;
    +----+---+---+----+
    | id   | a  | b  | c  |
    +----+---+---+----+
    |  1   | 5  | a  | aa |
    +----+---+---+----+
    1 row in set (0.00 sec) -- 此时进行B会话操作 A窗口事务查询信息 = B窗口事务查询信息
    mysql> update t1 set a=10 where id=1;-- A窗口事务进行修改
    -- 此时进行B会话操作
    mysql> commit;-- A窗口事务进行提交# 
    -- 数据库B会话窗口操作
    mysql> use test;
    mysql> begin;mysql> select * from t1 where id=1;
    +----+---+---+----+
    | id   | a  | b  | c  |
    +----+---+---+----+
    |  1   | 5  | a  | aa |
    +----+---+---+----+
    1 row in set (0.00 sec)-- A窗口事务查询信息 = B窗口事务查询信息
    mysql> select * from t1 where id=1;
    +----+---+---+----+
    | id   | a  | b  | c  |
    +----+---+---+----+
    |  1   | 5  | a  | aa |
    +----+---+---+----+
    1 row in set (0.00 sec)-- B窗口事务查询信息，不能看到A窗口事务未提交的数据变化，避免了脏数据问题；
    mysql> select * from t1 where id=1;
    +----+---+---+----+
    | id   | a  | b  | c  |
    +----+---+---+----+
    |  1   | 10 | a  | aa |
    +----+---+---+----+
    1 row in set (0.00 sec)-- A窗口事务提交之后，B窗口事务查询信息和之前不同了    
    ```
  
  
    - RR级别下
  
      ```sql
      -- 数据库A会话窗口操作
      mysql> use test;
      mysql> begin;
      mysql> select * from t1;-- 确认初始数据信息
      mysql> update t1 set a=10 where id=1;-- A窗口事务进行修改
      -- 此时进行B会话操作
      mysql> commit;-- A窗口事务进行提交
      -- 此时进行B会话操作
      
      -- 数据库B会话窗口操作
      mysql> use test;
      mysql> begin;
      mysql> select * from t1;-- 确认初始数据信息
      mysql> select * from t1 where id=1;
      +----+---+---+----+
      | id   | a  | b  | c  |
      +----+---+---+----+
      |  1   | 5  | a  | aa |
      +----+---+---+----+
      1 row in set (0.00 sec)-- B窗口事务查询信息，不能看到A窗口事务未提交的数据变化，避免了脏数据问题；
      mysql> select * from t1 where id=1;
      +----+---+---+----+
      | id   | a  | b  | c  |
      +----+---+---+----+
      |  1   | 5  | a  | aa |
      +----+---+---+----+
      1 row in set (0.00 sec)-- A窗口事务提交之后，B窗口事务查询信息和之前是相同的；
      -- 在RR级别状态下，同一窗口的事务生命周期下，每次读取相同数据信息是一样，避免了不可重复读问题
      mysql> commit;
      mysql> select * from t1 where id=1;-- 在RR级别状态下，同一窗口的事务生命周期结束后，看到的数据信息就是修改的了
      ```
      
  


- 幻读

  - RU/RC级别下



```
  -- 数据库A会话窗口操作
  mysql> use test;
  mysql> select * from t1;
  +----+----+---+-----+
  | id | a  | b | c   |
  +----+----+---+-----+
  |  1 | 10 | a | aa  |
  |  2 |  7 | c | ab  |
  |  3 | 10 | d | ae  |
  |  4 | 13 | g | ag  |
  |  5 | 14 | h | at  |
  |  6 | 16 | i | au  |
  |  7 | 20 | j | av  |
  |  8 | 22 | k | aw  |
  |  9 | 25 | l | ax  |
  | 10 | 27 | o | ay  |
  | 11 | 31 | p | az  |
  | 12 | 50 | x | aze |
  | 13 | 60 | y | azb |
  +----+----+---+-----+
  13 rows in set (0.00 sec)-- 查看获取A窗口表中数据
  mysql> alter table t1 add index idx(a);-- 在A窗口中，添加t1表的a列为索引信息
  mysql> begin;
  mysql> update t1 set a=20 where a<20;-- 在A窗口中，将a<20的信息均调整为20
  -- 此时进行B会话操作
  mysql> commit;-- 在A窗口中，进行事务提交操作，是在B窗口事务没有提交前
  mysql> mysql> select * from t1;-- 在A窗口中，查看数据信息，希望看到的a是没有小于20的，但是结果看到了a存在等于10的（即出现了幻读）
  
  -- 数据库B会话窗口操作
  mysql> use test;
  mysql> select * from t1;
  +----+----+---+-----+
  | id | a  | b | c   |
  +----+----+---+-----+
  |  1 | 10 | a | aa  |
  |  2 |  7 | c | ab  |
  |  3 | 10 | d | ae  |
  |  4 | 13 | g | ag  |
  |  5 | 14 | h | at  |
  |  6 | 16 | i | au  |
  |  7 | 20 | j | av  |
  |  8 | 22 | k | aw  |
  |  9 | 25 | l | ax  |
  | 10 | 27 | o | ay  |
  | 11 | 31 | p | az  |
  | 12 | 50 | x | aze |
  | 13 | 60 | y | azb |
  +----+----+---+-----+
  13 rows in set (0.00 sec)-- 查看获取B窗口表中数据
  mysql> begin;
  mysql> insert into t1(a,b,c) values(10,'A','B')-- 在B窗口中，插入一条新的数据信息 a=10 
  mysql> commit;-- 在B窗口中，进行事务提交操作
```

  - RR级别下

    ```sql
      -- 数据库A会话窗口操作
      mysql> use test;
      mysql> select * from t1;-- 查看获取A窗口表中数据
      mysql> alter table t1 add index idx(a);-- 在A窗口中，添加t1表的a列为索引信息
      mysql> begin;
      mysql> update t1 set a=20 where a>20;-- 在A窗口中，将a>20的信息均调整为20
    
      -- 数据库B会话窗口操作
      mysql> use test;
      mysql> select * from t1;-- 查看获取B窗口表中数据
      mysql> begin;
      mysql> insert into t1(a,b,c) values(30,'sss','bbb');-- 在B窗口中，插入一条新的数据信息 a=30，但是语句执行时会被阻塞，没有反应；
    
      -- 数据库C会话窗口操作
      mysql> show processlist;-- 在C窗口中，查看数据库连接会话信息，insert语句在执行，等待语句超时（默认超时时间是50s）
      -- 因为此时在RR机制下，创建了行级锁(阻塞修改)+间隙锁(阻塞区域间信息插入)=next lock
      -- 区域间隙锁 < 左闭右开(可用临界值)  ;  区域间隙锁 > 左开右闭（不可用临界值）
    ```


​      





# Solution

## 如何实现接口幂等性，有那些方法

1. **前端防重复提交**；
2. **数据库主键唯一约束**；
3. **每次请求设置一个唯一标识**，比如支付接口用订单ID作为唯一标识，然后去重；
4. **每次处理后标记已处理（增加额外的状态字段）**，比如生成交易流水号，下次请求判断是否已有交易流水号；
5. **申请预置令牌**，比如JWT等，携带在请求中，接口保存每个令牌第一次请求的处理结果，同一个令牌只处理一次；

## 单点登录有哪些实现方法及实现原理

## 分布式锁的实现方式及原理

Reddison + RedLock

**Reddison**：setnx + expire + watchDog lua脚本确保原子性

**RedLock**：解决主从架构下redis数据不一致问题

## 如何解决数据与缓存不一致

# Orm

## 使用 Mybatis 时，调用 DAO接口时是怎么调用到 SQL 的（7分）

简单点说，当我们使用 Spring+MyBatis 时：

1. DAO接口会被加载到 Spring 容器中，通过动态代理来创建
2. XML中的SQL会被解析并保存到本地缓存中，key是SQL 的 namespace + id，value 是SQL的封装
3. 当我们调用DAO接口时，会走到代理类中，通过接口的全路径名，从步骤2的缓存中找到对应的SQL，然后执行并返回结果

# Redis

## Redis数据类型

- String

  - string类型使用的数据结构有三种：int、aw、embstr，当保存的是数字类型的时候，底层存的是int类型，当保存的是一个字符串，并且字符串大于32字节的时候，底层存的是raw类型，当保存的是一个字符串，并且字符串小于等于32字节的时候，底层存的是embstr类型。
  - embstr只需要分配一次内存空间，因为redis对于embstr类型的，会分配一个连续的内存空间，而对于raw类型，则会需要分配两次内存（redisObject和sdshdr分别申请）空间，embstr格式的全部数据都在一串连续的空间中，所以使用这种格式的数据，更好的利用缓存带来的优势。
  - 任何append操作，都会使得其他的类型转换成raw类型，因为embstr格式，redis并没有给它有任何修改的api，所以本质上，embstr是只读的，所以append操作，会直接把其转成raw格式。

- List

  - List类型使用的数据结构有两种：ziplist,linkedlist

  - 当列表对象可以同时满足以下两个条件时，列表对象使用ziplist编码：

    1. 列表对象保存的所有字符串元素的长度都小于64字节；
    2. 列表对象保存的元素数量小于512个。

    不能满足这两个条件的列表对象需要使用linkedlist编码。
    以上两个条件的上限值是可以修改的，具体请看配置文件中关于list-max-ziplist-value选项和list-max-ziplist-entries选项的说明。

- Hash

  - hash类型使用的数据结构有两种:ziplist和hashtable

  - 当满足所有以下情况时，hash使用ziplist：

    1. hash保存的key和value的大小都小于64字节；
    2. hash里面key的数量小于512个。

  - 使用ziplist的情况，ziplist里面存的数据如下所示：

    ![image-20230821114534893](.\pic\image-20230821114534893.png)

    每次hash新增key的时候，key和value按顺序push到list的末尾。

  - 案例：hash使用的是ziplist（100个key）
    在实际使用过程中，hash的数据结构有时候很重要。比如说有一个hash结构，存的key是uid，value是时间戳。
    使用hmget这个指令，获取用户的信息。比如说传入的用户有一万个，hash里面存了100个，那么hmget需要遍历是10000*100次。如果是十万个用户，那这个遍历次数更多了。因为ziplist结构，每次查询，都可能遍历100次（大部分uid不存在）。
    而如果使用的是hashtable的话，遍历次数就少了100（元素数量）次。
    解决方案：如果key的数量较少使用的是ziplist，不使用hmget，而是一次性获取全部的值，在本地构建map去运算

- Set

  - set使用的数据结构有两种：intset和hashtable

  - 当set同时满足以下条件的时候，set使用intset结构：

    1. set里面存储的元素都是int类型。
    2. set最大数量小于512个。

  - ```c
    typedef struct intset{
        //编码方式
        uint32_t encoding;
        //集合包含的元素数量
        uint32_t length;
        //保存元素的数组
        int8_t contents[];
    }intset;
    ```

    如上代码，可以看出，intset是使用contents数组来保存元素的，contents数组是有序的，判断数字是否在可以通过二分查找进行。set结构上

  - hashtable跟普通的hashtable的区别就是其value都是null。这跟java中的Set实现很像，在java中，HashSet的底层实现就是HashMap，只不过，value都是无意义的Object对象。

    ![image-20230821131419211](.\pic\image-20230821131419211.png)

- ZSET

  - set使用的数据结构有以下两种：ziplist和skiplist

    当使用skiplist结构是，ZSET实际上是使用dict和skiplist一起实现的。
    因为其需要两者的特性，所以需要两者结合在一起使用。比如说，zrange 需要使用跳表的特性，才能快速range，而判断这个成员的分数等，则需要使用dict结构快速定位成员。

    ```c
    typedef struct zset 
    {
        zskiplist *zsl;    
         dict *dict;
    } zset;
    ```

  - 当有序集合对象可以同时满足以下两个条件时，对象使用ziplist编码：

    1. zset保存的元素数量小于128个；
    2. zset保存的所有元素成员的长度都小于64字节；



## Redis为什么快

1. 纯内存访问

2. 单线程避免上下文切换

3. 渐进式ReHash：两个全局哈希表多次拷贝，必满一次拷贝所有，造成IO阻塞；

   缓存时间戳：需要获取时间的场景比如TTL（Time To Live）每次要获取时间戳都是一次系统调用，系统调用对于单线程的Redis来说太奢侈，所以有个定时任务，每秒更新一次时间，需要时间的时候都是去缓存中拿。

## 持久化策略

RDB/AOF/混合

- RDB
- AOF
- 混合

## 缓存雪崩、击穿、穿透

- 雪崩：大量的应用请求无法在 Redis 缓存中进行处理，紧接着，应用将大量请求发送到数据库层，导致数据库层的压力激增。

  发生原因/出现场景：

  1. Redis服务正常但是大量数据同时过期
  2. Redis 缓存实例发生故障宕机，导致大量请求无法得到处理

  解决方案：

  1. 避免给大量的数据设置相同的过期时间。如果业务层的确要求有些数据要同时失效，你可以在用 EXPIRE 命令给每个数据设置过期时间时，给这些数据的过期时间增加一个较小的随机数。这样，不同数据的过期时间有所差别，但差别又不会太大，既避免了大量数据同时过期，同时也保证了这些数据基本在相近的时间失效，仍然能满足业务需求。

  2. 两个建议

     1. 服务熔断和请求限流

        监测 Redis 缓存所在机器和数据库所在机器的负载指标，例如每秒请求数、CPU 利用率、内存利用率等。如果我们发现 Redis 缓存实例宕机了，而数据库所在机器的负载压力突然增加（例如每秒请求数激增），此时，就发生缓存雪崩了。大量请求被发送到数据库进行处理。我们可以启动服务熔断机制，暂停业务应用对缓存服务的访问，从而降低对数据库的访问压力。属于时候诸葛亮，缓存已雪崩，防止引发连锁的数据库雪崩，甚至是整个系统的崩溃。是有损方案。

     2. 提前预防

        通过主从节点的方式构建 Redis 缓存高可靠集群。如果 Redis 缓存的主节点故障宕机了，从节点还可以切换成为主节点，继续提供缓存服务，避免了由于缓存实例宕机而导致的缓存雪崩问题。

- 击穿：

  发生原因/出现场景：热点数据缓存失效

  解决方案：为了避免缓存击穿给数据库带来的激增压力，我们的解决方法也比较直接，对于访问特别频繁的热点数据，我们就不设置过期时间了。这样一来，对热点数据的访问请求，都可以在缓存中进行处理，而 Redis 数万级别的高吞吐量可以很好地应对大量的并发请求访问。如果这个热点 key 的内容会变，我们可以使用单独的后台线程去构建缓存。但在缓存重构期间，会出现数据不一致情况。

  如果一定要设置过期时间，我们可以考虑使用锁机制来限制回源（过期后查询数据库）的并发。比如下面示例代码使用 Redisson 来获取一个基于 Redis 的分布式锁，客户端在查询数据库之前先尝试获取锁，这样，可以把回源到数据库的并发限制在 1：

  ```java
  @Autowired
  private RedissonClient redissonClient;
   
  @GetMapping("right")
  public String right() {
      String data = stringRedisTemplate.opsForValue().get("hotsopt");
      if (StringUtils.isEmpty(data)) {
          RLock locker = redissonClient.getLock("locker");
          //获取分布式锁
          if (locker.tryLock()) {
              try {
                  data = stringRedisTemplate.opsForValue().get("hotsopt");
                  //双重检查，因为可能已经有一个B线程过了第一次判断，在等锁，然后A线程已经把数据写入了Redis中
                  if (StringUtils.isEmpty(data)) {
                      //回源到数据库查询
                      data = getExpensiveData();
                      stringRedisTemplate.opsForValue().set("hotsopt", data, 5, TimeUnit.SECONDS);
                  }
              } finally {
                  //别忘记释放，另外注意写法，获取锁后整段代码try+finally，确保unlock万无一失
                  locker.unlock();
              }
          }
      }
      return data;
  }
  ```

  

- 穿透：缓存穿透是指要访问的数据既不在 Redis 缓存中，也不在数据库中，导致请求在访问缓存时，发生缓存缺失，再去访问数据库时，发现数据库中也没有要访问的数据。此时，应用也无法从数据库中读取数据再写入缓存，来服务后续请求，这样一来，如果应用持续有大量请求访问数据，就会同时给缓存和数据库带来巨大压力。

  发生原因/出现场景：

  1. 业务层误操作：缓存中的数据和数据库中的数据被误删除了，所以缓存和数据库中都没有数据；
  2. 恶意攻击：专门访问数据库中没有的数据。

  解决方案：

  1. 一旦发生缓存穿透，我们就可以针对查询的数据，在 Redis 中缓存一个空值或是和业务层协商确定的缺省值（例如，库存的缺省值可以设为 0）。紧接着，应用发送的后续请求再进行查询时，就可以直接从 Redis 中读取空值或缺省值，返回给业务应用了，避免了把大量请求发送给数据库处理，保持了数据库的正常运行。但这种方式可能会把大量无效的数据加入缓存中。

  2. 使用布隆过滤器快速判断数据是否存在，数据不存在时避免从数据库中查询。

  3. 基于***布隆过滤器***的快速检测特性，我们可以在把数据写入数据库时，使用布隆过滤器做个标记。当缓存缺失后，应用查询数据库时，可以通过查询布隆过滤器快速判断数据是否存在。如果不存在，就不用再去数据库中查询了。这样一来，即使发生缓存穿透了，大量请求只会查询 Redis 和布隆过滤器，而不会积压到数据库，也就不会影响数据库的正常运行。

  4. 在请求入口的前端进行请求检测。

     缓存穿透的一个原因是有大量的恶意请求访问不存在的数据，所以，一个有效的应对方案是在请求入口前端，对业务系统接收到的请求进行合法性检测，把恶意的请求（例如请求参数不合理、请求参数是非法值、请求字段不存在）直接过滤掉，不让它们访问后端缓存和数据库。这样一来，也就不会出现缓存穿透问题了。

- 总结

  跟缓存雪崩、缓存击穿这两类问题相比，缓存穿透的影响更大一些。从预防的角度来说，我们需要避免误删除数据库和缓存中的数据；从应对角度来说，我们可以在业务系统中使用缓存空值或缺省值、使用布隆过滤器，以及进行恶意请求检测等方法。

  ![image-20230816151834872](.\pic\image-20230816151834872.png)

# Net

## TCP/IP三次握手四次挥手过程以及原因

# Distributed System

## CAP理论
- 什么是CAP

  CAP 理论可以表述为，一个分布式系统最多只能同时满足一致性（Consistency）、可用性（Availability）和分区容忍性（Partition Tolerance）这三项中的两项。

  ![image-20230816141008673](.\pic\image-20230816141008673.png)

  在分布式中P是必须要有的，所以分布式有CP和AP两种模式。AP的就是可用性强，一致性弱；CP就是一致性强,可用性弱。我们可以把强弱理解成优缺点。

  CAP 原则指示3个要素最多只能同时实现两点，不可能三者兼顾，由于网络硬件肯定会出现延迟丢包等问题，但是在分布式系统中，我们必须保证部分网络通信问题不会导致整个服务器集群瘫痪，另外即使分成了多个区，当网络故障消除的时候，我们依然可以保证数据一致性，所以我们必须保证分区容错性；

  至于剩下的一致性和可用性，我们需要二选一，但是鱼和熊掌不可兼得，假设我们选择一致性，那我们就不能让用户访问无法进行数据同步的机器，毕竟该机器上的数据和其他正常机器上的不一致，但是这样我们就丢弃了可用性；假设我们选择可用性，那我们就可以让用户访问无法进行数据同步的服务器，虽然保证了可用性，但是我们无法保证数据一致性。

- Consistency（一致性）

  指“所有节点同时看到相同的数据”，即更新操作成功并返回客户端完成后，所有节点在同一时间的数据完全一致，等同于所有节点拥有数据的最新版本。我们可以更深入的理解一致性，它是指任何的读写都应该看起来是“原子”的，或串行的，写后面的读一定能读到前面写的内容，所有的读写请求都好像被全局排序；

- Availability（可用性）

  指“任何时候，读写都是成功的”，即服务一直可用，而且是正常响应时间。我们平时会看到一些公司的对外说自己系统稳定性已经做到 3 个 9、4 个 9，即 99.9%、99.99%，这里的 N 个 9 就是对可用性的一个描述，叫做 SLA，即服务水平协议。比如我们说年度 99.99% 的 SLA，则计算公式如下：

  1年 = 365天 = 8760小时

  99.99 = 8760 * 0.0001 = 0.876小时 = 0.876 * 60 = 52.6分钟

  是不是很牛逼，系统一年里只有52.6分钟不能提供服务。

- Partition Tolerance（分区容忍性）

  指“当部分节点出现消息丢失或者分区故障的时候，分布式系统仍然能够继续运行”，即系统容忍网络出现分区，并且在遇到某节点或网络分区之间网络不可达的情况下，仍然能够对外提供满足一致性和可用性的服务。

- 分区容忍性和可用性的区别

  分区容忍性和可用性二者很像，在这里我们做一下简单的总结:

  分区容忍性：因为网络等硬件引起的问题，一台服务器崩溃了，保证能在其他服务器上也能顺利完成业务。

  可用性：因为软件代码层面的问题，一台服务器上的服务崩溃了，保证能在其他服务器上完成该业务。

  二者主要区别是：分区容忍性更偏向于硬件引起的问题；可用性更偏向于软件代码层面的问题。

  在分布式系统中，由于系统的各层拆分，P 是确定的，CAP 的应用模型就是 CP 架构和 AP 架构。分布式系统所关注的，就是在 Partition Tolerance 的前提下，如何实现更好的 A （系统可用性）和更稳定的 C（数据一致性）。

- CP和AP的典型应用

  CP典型应用就是电商的产品价格，商家修改价格后要实时生效。

  AP的典型应用就是各大内容网站的点赞和评论功能，用户不太在意点赞和评论的实时性，更在意的时查看感兴趣内容的可访问性

# MQ（Message Queue）

## 为什么要使用消息队列

1. 解耦（把任务扔到消息队列就算结束了）
2. 异步（消费结果异步通知）
3. 流量削峰填谷（算是时间换空间）

## 消息队列有什么优缺点

优点见为什么要使用消息队列，列一下缺点：

1. 可用性降低（多加一个组件，就多了一个可能会出故障的组件），解决办法是架集群，多主多从
2. 系统复杂性提高，有了生产者和消费者的概念。带来的额外问题：消息重复了怎么办、消息丢了怎么办、消息顺序不对怎么办
3. 缺点二带来的一致性问题

## 常见消息队列

|                  | ActiveMQ | RabbitMQ | RocketMQ | Kafka |
| :--------------: | :------: | :------: | :------: | :---: |
| 性能<br>（单机并发） |   6000   |  12000<br>如果对并发要求不高建议Rabbit<br>因为Rabbit稳定性高<br>Active到6000容易出问题  |   10万   | 100万 |
| 数据持久化 | 支持<br>（开启持久化影响性能） | 支持<br>（开启持久化影响性能） | 天生支持 | 天生支持 |
| 多语言支持 | 主流都支持 | 主流都支持 | Java | 主流都支持 |
| 总结 | 缺乏大规模应用的案例<br>一般不推荐 | 优点：高可用（Erlang）<br>管理界面优秀<br>内部机制难了解，也是因为Erlang<br>集群不支持动态扩展 | 模型简单、接口易用、功能多<br>阿里大规模应用案例<br>性能好<br>缺点就是只支持Java | 优点：天生分布式<br>性能最好<br>大数据支持好<br>缺点：运维难度大<br>对带宽有一定要求<br> |



# Algorithm

## Sort

![image-20230828143925596](.\pic\image-20230828143925596.png)

### 1. BubbleSort（冒泡排序）

```java
public class BubbleSort {
    public static void bubbleSort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n; i++) {
            boolean swapped = false;
            for (int j = 0; j < n - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    // 交换位置
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                    swapped = true;
                }
            }
            // 如果没有发生交换，说明已经排好序，可以提前结束
            if (!swapped) {
                break;
            }
        }
    }

    public static void main(String[] args) {
        int[] myArray = {64, 34, 25, 12, 22, 11, 90};
        bubbleSort(myArray);
        
        System.out.print("Sorted array: ");
        for (int num : myArray) {
            System.out.print(num + " ");
        }
    }
}
```

在上述示例代码中，`bubbleSort` 方法接受一个整数数组作为参数，并对该数组进行冒泡排序。在每一轮遍历中，通过比较相邻元素并交换位置，逐步将较大的元素“冒泡”到数组的末尾。如果在某一轮遍历中没有发生任何交换，说明数组已经排好序，可以提前结束排序过程。最后，在 `main` 方法中演示了如何使用该排序算法对数组进行排序并输出结果。

### 2. SelectionSort（选择排序）

```java
public class SelectionSort {
    public static void selectionSort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            int minIndex = i;
            for (int j = i + 1; j < n; j++) {
                // 找到未排序部分中的最小元素的索引
                if (arr[j] < arr[minIndex]) {
                    minIndex = j;
                }
            }
            // 将最小元素与当前位置元素交换
            int temp = arr[i];
            arr[i] = arr[minIndex];
            arr[minIndex] = temp;
        }
    }

    public static void main(String[] args) {
        int[] myArray = {64, 34, 25, 12, 22, 11, 90};
        selectionSort(myArray);
    
        System.out.print("Sorted array: ");
        for (int num : myArray) {
            System.out.print(num + " ");
        }
    }

}
```

在上述示例代码中，`selectionSort` 方法接受一个整数数组作为参数，并对该数组进行选择排序。选择排序的核心思想是每一轮遍历中找到未排序部分的最小元素，并将其与当前位置元素交换。具体步骤如下：

1. 在每一轮遍历中，首先假定未排序部分的第一个元素是最小的。
2. 通过遍历未排序部分，找到真正的最小元素的索引。
3. 将找到的最小元素与未排序部分的第一个元素交换位置。

通过重复以上步骤，逐渐将最小的元素从未排序部分移到已排序部分的末尾，实现排序。

在 `main` 方法中，演示了如何使用选择排序算法对数组进行排序并输出结果。选择排序的时间复杂度也是O(n^2)，与冒泡排序类似，因此在大规模数据排序时效率较低。

### 3. InsertionSort（插入排序）

```java
public class InsertionSort {
    public static void insertionSort(int[] arr) {
        int n = arr.length;
        for (int i = 1; i < n; i++) {
            int key = arr[i];
            int j = i - 1;

            // 将比 key 大的元素向后移动，为 key 腾出插入位置
            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }
            arr[j + 1] = key; // 插入 key 到正确位置
        }
    }
    
    public static void main(String[] args) {
        int[] myArray = {64, 34, 25, 12, 22, 11, 90};
        insertionSort(myArray);
    
        System.out.print("Sorted array: ");
        for (int num : myArray) {
            System.out.print(num + " ");
        }
    }

}
```

在上述示例代码中，`insertionSort` 方法接受一个整数数组作为参数，并对该数组进行插入排序。插入排序的思想是从数组的第二个元素开始，将其与已排序的子数组比较，然后插入到正确的位置。具体步骤如下：

1. 从数组的第二个元素开始，将当前元素保存在变量 `key` 中。
2. 在已排序的子数组中，从右向左比较元素，如果比 `key` 大，将这些元素后移一位，直到找到合适的插入位置。
3. 将 `key` 插入到找到的插入位置。

最终，通过重复以上步骤，整个数组会逐步被分为已排序和未排序的部分，直到所有元素都被放置到正确的位置。

在 `main` 方法中，演示了如何使用插入排序算法对数组进行排序并输出结果。插入排序相对于冒泡排序来说，通常会有更好的性能，特别是在处理较小规模的数据时。

### 4. ShellSort（希尔排序）

```java
public class ShellSort {
    public static void shellSort(int[] arr) {
        int n = arr.length;
        

        // 初始间隔（步长）设置为数组长度的一半，然后逐步缩小间隔
        for (int gap = n / 2; gap > 0; gap /= 2) {
            for (int i = gap; i < n; i++) {
                int temp = arr[i];
                int j;
                // 在当前间隔内进行插入排序
                for (j = i; j >= gap && arr[j - gap] > temp; j -= gap) {
                    arr[j] = arr[j - gap];
                }
                arr[j] = temp;
            }
        }
    }
    
    public static void main(String[] args) {
        int[] myArray = {64, 34, 25, 12, 22, 11, 90};
        shellSort(myArray);
    
        System.out.print("Sorted array: ");
        for (int num : myArray) {
            System.out.print(num + " ");
        }
    }

}
```

在上述示例代码中，`shellSort` 方法接受一个整数数组作为参数，并对该数组进行希尔排序。希尔排序的核心思想是通过将数组分割成多个子序列，然后在每个子序列内使用插入排序。初始的间隔（步长）可以自行定义，一般是数组长度的一半，然后逐步缩小间隔直至为1。通过多次缩小间隔，使得数组的元素逐渐接近正确的位置，最终实现排序。

希尔排序的时间复杂度取决于间隔序列的选择，不同的间隔序列可能导致不同的性能表现。虽然希尔排序的时间复杂度不是固定的，但它通常表现良好，特别是对于中等规模的数据集。

### 5. MergeSort（归并排序）

```java
public class MergeSort {
    public static void mergeSort(int[] arr) {
        int n = arr.length;
        if (n <= 1) {
            return; // 递归终止条件，如果数组长度小于等于1，无需继续分割和合并
        }

        int mid = n / 2;
        int[] left = new int[mid];
        int[] right = new int[n - mid];
    
        // 将原数组分割成两个子数组
        System.arraycopy(arr, 0, left, 0, mid);
        System.arraycopy(arr, mid, right, 0, n - mid);
    
        // 递归地对子数组进行归并排序
        mergeSort(left);
        mergeSort(right);
    
        // 合并两个有序子数组
        merge(arr, left, right);
    }
    
    public static void merge(int[] arr, int[] left, int[] right) {
        int i = 0, j = 0, k = 0;
        
        // 比较并合并两个子数组
        while (i < left.length && j < right.length) {
            if (left[i] <= right[j]) {
                arr[k++] = left[i++];
            } else {
                arr[k++] = right[j++];
            }
        }
    
        // 将剩余元素合并进数组
        while (i < left.length) {
            arr[k++] = left[i++];
        }
        while (j < right.length) {
            arr[k++] = right[j++];
        }
    }
    
    public static void main(String[] args) {
        int[] myArray = {64, 34, 25, 12, 22, 11, 90};
        mergeSort(myArray);
    
        System.out.print("Sorted array: ");
        for (int num : myArray) {
            System.out.print(num + " ");
        }
    }
}
```

在上述示例代码中，`mergeSort` 方法接受一个整数数组作为参数，并对该数组进行归并排序。归并排序的核心思想是将数组分割成两个子数组，然后递归地对这些子数组进行排序，最后将排序好的子数组合并成一个有序数组。`merge` 方法用于合并两个有序子数组。

归并排序的时间复杂度是稳定的，为 O(n log n)，适用于各种数据规模。虽然它需要额外的空间来存储子数组和合并结果，但其稳定性和效率使得它成为一种常用的排序算法。

### 6. QuickSort（快速排序）

```java
public class QuickSort {
    public static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int pivotIndex = partition(arr, low, high); // 找到基准元素的正确位置
            quickSort(arr, low, pivotIndex - 1); // 对基准元素左边的子数组进行快速排序
            quickSort(arr, pivotIndex + 1, high); // 对基准元素右边的子数组进行快速排序
        }
    }

    public static int partition(int[] arr, int low, int high) {
        int pivot = arr[high]; // 选择数组最后一个元素作为基准元素
        int i = low - 1; // i 是小于基准的子数组的最后一个元素的索引
    
        for (int j = low; j < high; j++) {
            if (arr[j] < pivot) {
                i++;
                // 交换 arr[i] 和 arr[j]
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
    
        // 将基准元素放置到正确的位置
        int temp = arr[i + 1];
        arr[i + 1] = arr[high];
        arr[high] = temp;
    
        return i + 1; // 返回基准元素的索引
    }
    
    public static void main(String[] args) {
        int[] myArray = {64, 34, 25, 12, 22, 11, 90};
        quickSort(myArray, 0, myArray.length - 1);
    
        System.out.print("Sorted array: ");
        for (int num : myArray) {
            System.out.print(num + " ");
        }
    }

}
```

### 7. HeapSort（堆排序）

```java
public class HeapSort {
    public static void heapSort(int[] arr) {
        int n = arr.length;

        // 构建最大堆
        for (int i = n / 2 - 1; i >= 0; i--) {
            heapify(arr, n, i);
        }
    
        // 逐步将最大元素移到末尾，然后重新调整堆
        for (int i = n - 1; i > 0; i--) {
            // 交换根节点（最大元素）和当前末尾元素
            int temp = arr[0];
            arr[0] = arr[i];
            arr[i] = temp;
    
            // 重新调整堆
            heapify(arr, i, 0);
        }
    }
    
    public static void heapify(int[] arr, int n, int i) {
        int largest = i; // 初始化最大元素索引
        int left = 2 * i + 1;
        int right = 2 * i + 2;
    
        // 比较左子节点和根节点
        if (left < n && arr[left] > arr[largest]) {
            largest = left;
        }
    
        // 比较右子节点和当前最大节点
        if (right < n && arr[right] > arr[largest]) {
            largest = right;
        }
    
        // 如果最大节点不是根节点，则交换它们，并递归地调整堆
        if (largest != i) {
            int swap = arr[i];
            arr[i] = arr[largest];
            arr[largest] = swap;
    
            heapify(arr, n, largest);
        }
    }
    
    public static void main(String[] args) {
        int[] myArray = {64, 34, 25, 12, 22, 11, 90};
        heapSort(myArray);
    
        System.out.print("Sorted array: ");
        for (int num : myArray) {
            System.out.print(num + " ");
        }
    }

}
```

在上述示例代码中，`heapSort` 方法实现了堆排序算法。堆排序的核心思想是首先构建一个最大堆（或最小堆），然后逐步将堆顶元素（最大元素）移动到数组的末尾，同时调整堆，以确保堆的性质。`heapify` 方法用于调整堆的性质，保证根节点的值大于等于子节点的值（最大堆情况）。

堆排序的时间复杂度为 O(n log n)，并且具有稳定性。堆排序的优势在于它不需要额外的空间来存储子数组，因此在某些情况下可能比其他排序算法更加适用。

### 8. CountingSort（计数排序）

```java
public class CountingSort {
    public static void countingSort(int[] arr) {
        int n = arr.length;

        // 找出数组中的最大值和最小值
        int max = arr[0];
        int min = arr[0];
        for (int num : arr) {
            if (num > max) {
                max = num;
            }
            if (num < min) {
                min = num;
            }
        }
    
        int range = max - min + 1; // 计算范围
        int[] countArray = new int[range]; // 计数数组
    
        // 统计每个元素的出现次数
        for (int num : arr) {
            countArray[num - min]++;
        }
    
        // 将计数数组转换为前缀和数组，表示每个元素在排序后的数组中的位置
        for (int i = 1; i < range; i++) {
            countArray[i] += countArray[i - 1];
        }
    
        // 构建排序后的数组
        int[] sortedArray = new int[n];
        for (int i = n - 1; i >= 0; i--) {
            int num = arr[i];
            sortedArray[countArray[num - min] - 1] = num;
            countArray[num - min]--;
        }
    
        // 将排序后的数组拷贝回原数组
        System.arraycopy(sortedArray, 0, arr, 0, n);
    }
    
    public static void main(String[] args) {
        int[] myArray = {4, 2, 2, 8, 3, 3, 1};
        countingSort(myArray);
    
        System.out.print("Sorted array: ");
        for (int num : myArray) {
            System.out.print(num + " ");
        }
    }

}
```

在上述示例代码中，`countingSort` 方法实现了计数排序算法。计数排序的核心思想是统计每个元素的出现次数，然后通过计算前缀和数组来确定每个元素在排序后的数组中的位置。最后，根据前缀和数组构建排序后的数组。

计数排序适用于输入元素范围较小的情况，当输入范围较大时，它可能需要较多的空间。它的时间复杂度为 O(n + k)，其中 n 是元素个数，k 是元素范围。计数排序在输入元素范围有限的情况下，具有较好的性能。

### 9. BucketSort（桶排序）

```java
import java.util.ArrayList;
import java.util.Collections;

public class BucketSort {
    public static void bucketSort(int[] arr) {
        int n = arr.length;

        // 创建桶
        int numBuckets = 5; // 设置桶的数量
        ArrayList<ArrayList<Integer>> buckets = new ArrayList<>(numBuckets);
        for (int i = 0; i < numBuckets; i++) {
            buckets.add(new ArrayList<>());
        }
    
        // 将元素分散到桶中
        for (int num : arr) {
            int bucketIndex = num / numBuckets; // 根据元素大小确定桶的索引
            buckets.get(bucketIndex).add(num);
        }
    
        // 对每个桶内的元素进行排序
        for (ArrayList<Integer> bucket : buckets) {
            Collections.sort(bucket);
        }
    
        // 将桶内的元素合并成有序数组
        int index = 0;
        for (ArrayList<Integer> bucket : buckets) {
            for (int num : bucket) {
                arr[index++] = num;
            }
        }
    }
    
    public static void main(String[] args) {
        int[] myArray = {64, 34, 25, 12, 22, 11, 90};
        bucketSort(myArray);
    
        System.out.print("Sorted array: ");
        for (int num : myArray) {
            System.out.print(num + " ");
        }
    }

}
```

在上述示例代码中，`bucketSort` 方法实现了桶排序算法。桶排序（Bucket Sort）是一种分布式排序算法，核心思想是将元素分散到多个桶中，然后对每个桶内的元素进行排序，最后将桶内的元素合并成有序数组。

在这个示例中，我们将元素分散到 5 个桶中，每个桶覆盖一定的元素范围。然后对每个桶内的元素进行排序，可以使用标准库提供的排序函数。最后，将桶内的元素按顺序合并到原数组中。

桶排序的时间复杂度可以是线性的，但是它对输入数据的分布和桶的设置都有一定的要求。在某些场景下，桶排序可以具有较好的性能。

### 10. RadixSort（基数排序）

```
import java.util.Arrays;

public class RadixSort {
    public static void radixSort(int[] arr) {
        int n = arr.length;
        int maxNum = Arrays.stream(arr).max().getAsInt(); // 找到最大的元素

        for (int exp = 1; maxNum / exp > 0; exp *= 10) {
            countingSortByDigit(arr, n, exp);
        }
    }
    
    public static void countingSortByDigit(int[] arr, int n, int exp) {
        int[] output = new int[n];
        int[] count = new int[10]; // 基数为0到9的数字
    
        // 统计每个基数出现的次数
        for (int i = 0; i < n; i++) {
            int digit = (arr[i] / exp) % 10;
            count[digit]++;
        }
    
        // 计算每个基数的前缀和，得到每个基数在排序后的位置
        for (int i = 1; i < 10; i++) {
            count[i] += count[i - 1];
        }
    
        // 构建排序后的数组
        for (int i = n - 1; i >= 0; i--) {
            int digit = (arr[i] / exp) % 10;
            output[count[digit] - 1] = arr[i];
            count[digit]--;
        }
    
        // 将排序后的数组拷贝回原数组
        System.arraycopy(output, 0, arr, 0, n);
    }
    
    public static void main(String[] args) {
        int[] myArray = {170, 45, 75, 90, 802, 24, 2, 66};
        radixSort(myArray);
    
        System.out.print("Sorted array: ");
        for (int num : myArray) {
            System.out.print(num + " ");
        }
    }

}
```

基数排序（Radix Sort）是一种非比较的排序算法，它按照数字的每个位来排序元素。

在上述示例代码中，`radixSort` 方法实现了基数排序算法。基数排序的核心思想是按照数字的每个位来排序元素，从最低位到最高位逐渐排序。`countingSortByDigit` 方法用于在每一位上使用计数排序。

基数排序的时间复杂度是 O(nk)，其中 n 是元素个数，k 是元素的位数。基数排序适用于元素较大但位数较小的情况，尤其在位数相同的情况下，它可以具有较好的性能。
