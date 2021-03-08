# Java基础

#### hashmap1.7跟1.8？优化点？红黑树化为什么是8？退化为什么？

答：jdk1.7的HashMap底层数据结构是数组+链表，链表是为了解决哈希冲突问题，如果哈希冲突该位置建立一个链表；而jdk1.8对HashMap进行了一些修改，最大的不同就是引入了红黑树，当链表长度超过8后就会将链表转化成红黑树。

Jdk1.8中链表长度到8 & 数组长度到64时会红黑树化，因为理想情况下使用随机哈希码，容器中节点分布在hash桶中的频率遵循泊松分布，根据泊松分布计算公式计算桶中元素个数与概率的对照表，可以看到元素为8的时候，概率已经非常小，一亿分之6，不到一千万分之一，所以原作者选择了8，是根据概率统计而选择的。

当红黑树内部节点小于等6个的时候会退化，作者选择6而不是7应该使用防止边界条件频繁树化和取消树化的考虑，对于为什么会需要退化，首先TreeNode 相比普通的 Node 来说，会有两倍的空间占用，而且在长度较小的情况下，红黑树的查找效率跟链表差别不大，毕竟HashMap是java提供的基础数据结构，必须在空间和时间上做抉择，选择链表是空间复杂度优先，选择红黑树是时间复杂度优先。

#### dp怎么玩？回溯怎么玩？递归怎么玩？stack能解决啥问题？fifo能解决啥问题？dfs怎么玩？bfs怎么玩？

答：动态规划其实将复杂问题分解成子问题，然后找出推导公式，进行动态递推的一种算法，在例如斐波那契数列，爬楼梯，棋盘走法等等问题都可以用动态规划解决。

回溯是一种算法思想，它是用递归实现的，从问题的某一种可能出发, 搜索从这种情况出发所能达到的所有可能, 当这一条路走到” 尽头 “的时候, 再倒回出发点, 从另一个可能出发, 继续搜索. 这种不断” 回溯 “寻找解的方法, 称作” 回溯法 “。

递归是一种算法结构，形式上表现为直接或间接的自己调用自己。

stack是一个FILO的数据结果，其中最典型的应用就是jvm内部的栈，一个线程的栈里面有多个栈帧(方法)，栈帧里面又有每个方法的局部变量表、操作数栈等，正是因为这样的结构才会让递归的回溯表现出走到头然后倒回去再选择另一条分支。stack也是个很适合用来检查左右符号是否匹配。

fifo就是解决排队的问题，即队列，例如一些任务调度，线程池里面都有队列来进行缓冲。

dfs深度遍历，一般使用递归来编写，

bfs广度优先遍历，一般使用队列来编写，适合用在一下查找最短路径问题

#### 双亲委派模型。JDBC和双亲委派模型关系

答：双亲委派模型就是当一个类收到了类加载请求，它首先不会舱室自己去加载这个类，而是把这个请求委派给父 类加载器去完成，每一个层次类加载器都是如此，因此所有的加载请求都应该传送到启动类加载器(Bootstrap ClassLoader)中，只有当父类加载器反馈自己无法完成这个请求的时候(在它的加载路径下没有找到所需加载的Class)，子类加载器才会舱室自己去加载。

采用双亲委派模型的一个好处是比如加载位于rt.jar包的类java.lang.Object，不管是哪个加载器加载这个类，最终都是委托给顶层的启动类加载器进行加载，这样就保证了使用不同的类加载器最终得到的都是同一个Object的·Class对象。

JDBC和双亲委派模型关系：

加载数据库驱动类有两种方式：1：直接Class.forName("com.mysql.jdbc.Driver")，这种没有破坏双亲委派模型。

2：使用java SPI，在META-INF/services/java.sql.Driver文件指明当前的Driver使用哪一个。

当时使用Java SPI时就破坏了双亲委派模型，因为ServiceLoader.load代码在DriverManager类里面，这个类在rt.jar包中，因此是启动类加载器来加载你指定的驱动如com.mysql.jdbc.Driver，但是这个驱动类肯定不在%JAVA_HOME%lib下，所以肯定是如法加载mysql的这个类的，那么这个问题是如何解决的呢，按照目前情况来看，这个类只有应用类加载器才能加载，也就是说启动类加载器必须有方法获取应用类加载器，然后通过它去加载，这就是所谓的线程上下文加载器，线程上下文加载器可以通过Thread.setContextClassLoader方法设置，如果不特殊设置会从父类继承，一般是应用程序类加载器。线程上下文类加载器让父类加载器能够通过调用子级类加载器来加载类，这打破双亲委派模型的原则。

##### 额外知识：

启动类/根类加载器（BootStrap ClassLoader）  ——》%JAVA_HOME%lib路径下的jar包

拓展类加载器（Extension ClassLoader）            ——》%JAVA_HOME%jre/lib/ext路径下的jar包

系统/应用类加载器(Application ClassLoader)       ——》CLASSPATH路径下指定，如未设置则为应用程序当前路径

自定义类加载器														——》继承ClassLoader ，实现Class<?> findClass(String name)方法，使用方法：Class.forName("com.sign.Hello", true, new HelloClassLoader());



#### TCP四次挥手，TIME_WAIT发生在哪一方 TIME_WAIT过多如何处理 

答：发生在主动关闭方，默认两个MSL，linux上MSL默认1分钟，window上默认2分钟。TIME_WAIT过多可以通过修改系统配置来降低。

##### 额外知识：

三次握手：

主动连接方(客户端)：发起连接指令：SYN = 1, seq = x，然后进入SYN-SENT状态

被动连接方(服务端)：起始LISTEN状态，接收到客户端的连接指令后，回应确认连接指令SYN=1, ACK = 1, seq = y, ack = x + 1，然后进入SYN-RCVD状态

主动连接方(客户端)：收到服务端的确认连接指令后，回应确认收到指令ACK = 1, seq = x + 1, ack = y + 1，进入ESTAB-LISHED状态

被动连接方(服务端)：收到确认连接指令后进入ESTAB-LISHED状态

四次挥手：

主动关闭方(客户端)：主动发起关闭指令FIN = 1, seq = u，状态由ESTAB-LISHED状态变成FIN-WAIT-1

被动关闭方(服务器)：接受到关闭指令，回应确认收到指令ACK = 1, seq = v, ack = u + 1，状态由ESTAB-LISHED状态变成CLOSE-WAIT

主动关闭方(客户端)：收到服务端的确认收到指令，状态由FIN-WAIT-1变成FIN-WAIT-2

被动关闭方(服务器)：一段时间后，向主动关闭方发出确认关闭指令FIN = 1, ACK = 1, seq = w, ack = u + 1，状态由CLOSE-WAIT变成LAST-ACK

主动关闭方(客户端)：收到服务端的确认关闭指令，回应确认收到指令ACK = 1, seq = u + 1, ack = w + 1，状态由FIN-WAIT-2变成TIME-WAIT，两个MSL后变成CLOSED

被动关闭方(服务器)：收到客户端的确认收到指令后，状态由LAST-ACK变成CLOSED



* HashMap底层结构 put操作讲一下,HashMap、HashMap如何保证线程安全、ConcurrentHashMap

* 从ConcurrentHashMap一路问到锁&锁优化->LongAdder->伪共享->缓存行填充->cas等诸多技术细节；

* 观察者模式与中介者模式有什么区别？

* 手写一个基于懒汉式的双重检测的单例。

  ```
  public class Singleton{
  	private Singleton instance;
  	
  	private Singleton(){}
  	
  	public Singleton getInstance() {
  		if(instance == null) {
  			synchronized(Singleton.class){
  				if (instance == null) {
  					instance = new Singleton();
  				}
  			}
  		}
  		return instance;
  	}
  }
  ```

  

* HashMap相关?为什么要引入红黑树？ 如何在红黑树中插入一个节点。 链表是如何转换为红黑树的？

* 对ConcurrentHashMap的理解，⽐如在什么地⽅会涉及到线程安全问题以及ConcurrentHashMap是如何解决的？

* Http请求的完全过程

* HashMap扩容的触发条件是什么

* HashMap的实现原理，什么是hash碰撞，怎样解决hash碰撞？

* mysql 的sql本身没问题的情况下，没走索引原因

* mysql快照是怎么实现的

* mysql分页有什么优化 

* 讲一下Http，HTTP安全不？HTTPS如何解决的？HTTP的数字证书如何认证？TCP与UDP区别？TCP为什么要四次？ 为什么TIME_WAIT 等待的时间是 2MSL？已经主动关闭连接了为什么还要保持资源一段时间呢？ TIME_WAIT 过多有什么危害？如果已经建⽴了连接，但是客户端突然出现故障了怎么办？保活机制说一下？

* 说一下undolog， redolog MySQL如何保证redo log和binlog的数据是一致的，如果一个sql执行很慢，你能分析一下原因吗？ 为什么数据库会选错了索引

* 对乐观锁和悲观锁的理解；

* hashMap什么情况下会出现循环链表？concurrentHashMap写的时候用什么锁？

* 定义Integer x=20 Integer y=200 在内存里是个什么过程？ volite关键字的原理？它能保证原子性吗？AtomicInteger底层怎么实现的？

* threadLocal关键字有用过吗？如果没有重写initialValue方法就直接get会怎样？

* 1.java的基本数据类型与包装类； 2、final修饰变量类方法； 3、String为什么是不可变的，以及new String(“abc”)创建了几个对象； 4、String、StringBuffer、以及StringBuilder的区别； 5、static修饰变量，方法，代码块； 6、重写跟重载的区别； 7、接口跟抽象类； 8、反射、继承、枚举、异常等知识点； 9、为什么要重写hashcode和equals方法，以及hashcode相同equals是否相同

* 集合相关 1、ArrayList的底层实现、扩容过程、add过程、Fail-Fast机制； 2、ArrayList与Linkedlist、Vectot的区别； 3、如何获得一个线程安全的List； 4、CopyOnWriteArrayList是如何实现线程安全的； 5、Linkedlist的底层实现，以及如何使用LinkedList实现一个LRU； 6、TreeSet、HashSet、LinkedHashSet的底层实现以及之间的区别； 7、PriorityQueue、LinkedBlockingQueue、ArrayBlockingQueue的实现以及区别； 8、HashMap的底层实现，扩容过程，达到阈值一定会扩容吗、put过程、树化过程，如何确定负载因子、以及为什么线程不安全和1.8做了哪些优化； 9、HashMap与HashTable的区别，如何获得一个线程安全的Map； 10、ConcurrentHashMap为什么是线程安全的，以及1.8做了哪些优化； 11、LinkedHashMap的底层实现，以及如何实现LRU； 12、TreeMap的底层实现； 13、迭代器的实现；

* 1. 面向对象的特点有哪些？ 2. 列举几个java常用的package及其作用 3. 接口和抽象类有什么联系和区别 4. 重载和重写有什么区别 5. java有哪些基本数据类型？ 6. Java支持的数据类型有哪些？什么是自动拆装箱？ 7. int 和 Integer 有什么区别 8. 数组有没有length()方法？String有没有length()方法？ 9. Java中符号>>和>>>有什么区别？ 10. Java类的实例化顺序 11. 什么是值传递和引用传递 （1）值传递是对基本型变量而言的,传递的是该变量的一个副本,改变副本不影响原变量. （2）引用传递一般是对于对象型变量而言的,传递的是该对象地址的一个副本, 并不是原对象本身 。 12. String能被继承吗？为什么？ 13. String和StringBuilder、StringBuffer的区别？

* 深拷贝和浅拷贝。 

* Integer a1 = new Integer(2); Integer a2 = new Integer(2); a1.equals(a2)的结果？？ 

* 为什么在重写equals方法的时候要重写hashcode的方法？

* HTTP 1.0 和 HTTP 2.0 的区别 HTTP 2.0 做了哪些优化

* JDK中偏向锁、自旋锁、轻量级锁、重量级锁的区别？ JDK锁自旋的自旋阈值了解吗？如何调整自旋次数？

* 如果相等hash对象太多，那么怎么解决迭代的影响？

* 服务器CPU数量及线程池数量的关系？

* 一亿条记录，内存中肯定放不下，要怎么找出其中最大的十条？

* 为什么要用读写锁而不用synchronized这种同步锁？ 

* 事务隔离性的理解，为什么会有脏读，可重复读，提交读等。

* 了解哪些设计模式，6个设计原则分别是什么？每种设计原则体现的设计模式是哪个？

* 如何实现session共享？用Redis该如何实现？

* 常见的stackoverflowexception,outofmemoryexception是怎么回事；

* top和jstack命令用过没，jstack命令的nid是什么意思，怎么查看java某个进程的线程？

* ABA怎么发生的，怎么解决ABA问题

* Mysql对联合索引有优化么？会自动调整顺序么？哪个版本开始优化？

* 什么是死锁？ 2.死锁产生的条件？ 3.怎样避免死锁？

* TCP三次握手和四次挥手的流程，为什么断开连接要4次,如果握手只有两次，会出现什么 .TIME_WAIT和CLOSE_WAIT的区别 .说说你知道的几种HTTP响应码 
