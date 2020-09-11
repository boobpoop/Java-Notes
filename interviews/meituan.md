1. 写一个函数，找到一个文件夹下所有文件，包括子文件夹
2. 如何判断链表中是否有环？
servlet和MVC的区别？

-    Servlet是接口，运行在Web 服务器中来处理用户的请求。然后Servlet接口提供了5个方法：初始化init，销毁destory，处理请求service，获取Servlet参数getServletConfig，获取Servlet信息getServletInfo。当然主要的方法还是前三个方法，体现了servlet的生命周期，从容器创建时要做什么的init，处理请求并返回时做什么的service，到请求结束后销毁servlet做什么的destory.   接下来，什么是SpringMVC呢，这是Spring提供的一个mvc框架，通过把保存了数据Map的Model，页面显示的View，控制器Controller三者分离，简化了web项目开发的复杂度，降低了分工的难度。    Spring Boot将tomcat设为默认容器，tomcat是一个Servlet容器，所以Spring MVC的入口也毫无意外的是一个Servlet，也就是前置控制器DispatcherServlet，作为Spring MVC架构的核心，前置控制器能够拦截请求，将其分发给Controller处理。handler在这里，是指包含了我们请求的Controller类和Method方法的对象。


适配器模式
- 目标接口：Target，该角色把其他类转换为我们期望的接口
- 被适配者: Adaptee 原有的接口，也是希望被改变的接口
- 适配器： Adapter, 将被适配者和目标接口组合到一起的类


mysql主从复制如何复制

- master开启bin-log功能，日志文件用于记录数据库的读写增删
需要开启3个线程，master IO线程，slave开启 IO线程 SQL线程，
Slave 通过IO线程连接master，并且请求某个bin-log，position之后的内容。
MASTER服务器收到slave IO线程发来的日志请求信息，io线程去将bin-log内容，position返回给slave IO线程。
slave服务器收到bin-log日志内容，将bin-log日志内容写入relay-log中继日志，创建一个master.info的文件，该文件记录了master ip 用户名 密码 master bin-log名称，bin-log position。
slave端开启SQL线程，实时监控relay-log日志内容是否有更新，解析文件中的SQL语句，在slave数据库中去执行。


mysql的ACID，如何保证持久性
- Mysql对于数据的修改总是先修改内存中的数据，按照功能划分为，数据buffer，索引buffer，锁buffer，relog等。。数据buffer是用户空间的，osbuffer是内核空间的
- 修改数据之后，数据buffer直接刷到os buferr ,然后调用磁盘同步函数fsync，不就可以直接实现持久化吗？直接将osbufer更新到磁盘，会使得如果机器宕机，数据丢失，要在写用户的数据buffer前，将操作指令加入到redo log中。



进程调度算法
- 先来先服务调度算法
- 短作业(进程)优先调度算法
- 优先权调度算法的类型。
为了照顾紧迫型作业，使之在进入系统后便获得优先处理，引入了最高优先权优先(FPF)调度算法。此算法常被用于批处理系统中，作为作业调度算法，也作为多种操作系统中的进程调度算法，还可用于实时系统中。当把该算法用于作业调度时，系统将从后备队列中选择若干个优先权最高的作业装入内存。当用于进程调度时，该算法是把处理机分配给就绪队列中优先权最高的进程，这时，又可进一步把该算法分成如下两种。
- 高响应比优先调度算法。
在批处理系统中，短作业优先算法是一种比较好的算法，其主要的不足之处是长作业的运行得不到保证。如果我们能为每个作业引入前面所述的动态优先权，并使作业的优先级随着等待时间的增加而以速率a 提高，则长作业在等待一定的时间后，必然有机会分配到处理机。该优先权的变化规律可描述为：操作系统由于等待时间与服务时间之和就是系统对该作业的响应时间，故该优先权又相当于响应比RP。
- 时间片轮转法
- 多级反馈队列调度算法


java基本数据类型以及对应字节数
- java的char多少字节：2字节

sleep和wait
- sleep是Thread类的方法，导致此线程暂停执行指定时间，给其他线程执行机会，但是依然保持着监控状态，过了指定时间会自动恢复，调用sleep方法不会释放锁对象。

当调用sleep方法后，当前线程进入阻塞状态。目的是让出CPU给其他线程运行的机会。但是由于sleep方法不会释放锁对象，所以在一个同步代码块中调用这个方法后，线程虽然休眠了，但其他线程无法访问它的锁对象。这是因为sleep方法拥有CPU的执行权，它可以自动醒来无需唤醒。而当sleep()结束指定休眠时间后，这个线程不一定立即执行，因为此时其他线程可能正在运行。

- wait方法是Object类里的方法，当一个线程执行到wait()方法时，它就进入到一个和该对象相关的等待池中，同时释放了锁对象，等待期间可以调用里面的同步方法，其他线程可以访问，等待时不拥有CPU的执行权，否则其他线程无法获取执行权。当一个线程执行了wait方法后，必须调用notify或者notifyAll方法才能唤醒，而且是随机唤醒，若是被其他线程抢到了CPU执行权，该线程会继续进入等待状态。由于锁对象可以时任意对象，所以wait方法必须定义在Object类中，因为Obeject类是所有类的基类。


数据表新增一列
- 1、ALTERTABLE table_name ADD column_name datatype

要删除表中的列，请使用下列语法： 

- ALTERTABLE table_name DROP COLUMN column_name


为什么java8的hashmap使用红黑树？为什么8的时候转化为树，6的时候退化？如果删除结点，会怎么样
- 红黑树的平均查找长度是log(n)，如果长度为8，平均查找长度为log(8)=3，链表的平均查找长度为n/2，当长度为8时，平均查找长度为8/2=4，红黑树的查找效率更高，这才有转换成树的必要；
链表长度如果是小于等于6，6/2=3，而log(6)=2.6，虽然速度也很快的，但是转化为树结构和生成树的时间并不会太短

lambda是怎么实现的
- lambda 的实现是通过 invokedynamic 指令来实现的。先在运行时动态解析出调用点限定符所引用的方法，然后再执行该方法，在此之前的4条调用指令，分派逻辑是固化在java虚拟机内部的，而invokedynamic指令的分派逻辑是由用户所设定的引导方法决定的。

反射

vector和arrayList区别
- vector是线程（Thread）同步（Synchronized）的，所以它也是线程安全的，而Arraylist是线程异步（ASynchronized）的，是不安全的。如果不考虑到线程的安全因素，一般用Arraylist效率比较高。
如果集合中的元素的数目大于目前集合数组的长度时，vector增长率为目前数组长度的100%,而arraylist增长率为目前数组长度。
的50%.如过在集合中使用数据量比较大的数据，用vector有一定的优势。

多个socket出现time_wait()状态是在什么情况下发生的，应该如何解决？
- net.ipv4.tcp_tw_recycle = 1    （表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭）
- TCP_NODELAY = 1 (tcp防止粘包)
- tcp_syncookies = 1 (防止ddos flood攻击)

手撕代码题，判断是否为回文链表，能否用O(1)的空间实现

堆排序原理（不懂！放过我吧……

手撕：全排列并打印（如abc之类
16、手撕：树的深度（dfs

dijkstra算法原理、时间复杂度
- 在一个有向加权图中，求一个点到其他任意点的最短路径，首先需要有一个数组来表示当前点到其他点的最短路径长度。核心思路是从顶点 A 往外延伸，不断更新 A 到其他点的距离，我们称这个操作为松弛。O(e·logv)。

文件太大怎么读取（怎么读取啊？？
- Nio的MappedByteBuffer可以读取：
- MappedByteBuffer map=channel.map(FileChannel.MapMode.READ_ONLY,begin,length); //上面begin为long，表示在文件中的起始位置，length表示缓冲长度，


3.熟悉红黑树吗？（不会）
- 每个节点都有红色或黑色
- 树的根始终是黑色的 (黑土地孕育黑树根， )
- 没有两个相邻的红色节点（红色节点不能有红色父节点或红色子节点，并没有说不能出现连续的黑色节点）
- 从节点（包括根）到其任何后代NULL节点(叶子结点下方挂的两个空节点，并且认为他们是黑色的)的每条路径都具有相同数量的黑色节点

4.B树和B+树了解吗？
- B树： 每个节点都存储key和data，所有节点组成这棵树，并且叶子节点指针为null。
- B+树： 只有叶子节点存储data，叶子节点包含了这棵树的所有键值，叶子节点不存储指针。

- innodb：data存的是数据本身。索引也是数据。数据和索引存在一个XX.IDB文件中，所以也叫聚集索引。


13.排序算法，复杂的，稳定性，快排的实现和优化。
13.编程题：给定有序数组，找到两个数使得它俩的和等于目标值。
14.发散题：3只老鼠，8瓶药，其中有一瓶毒药，假设喝到毒药后一小时死亡，可以多瓶混合一起喝，怎么才能用最少的时间找到哪一瓶是毒药。（只要1小时）
- https://zhidao.baidu.com/question/1734400719873404787.html


3.假设给定你四个字段分别为一个int，一个double，一个日期类型，一个varchar(20)的数据，共100万条，计算数据大小（不会）
- datetime占用8字节,timestamp占用4字节。
- double 8字节。

如果对象大部分都是存活的，少部分需要清除，用什么算法
- 新生代绝大部分对象朝生熄灭，只有少部分存活，采用复制算法最合适不过。老年代对象即使进行了垃圾回收，对象的存活率也高，所以采用标记清除或标记整理算法都是不错的选择。

5说说对象创建到消亡的过程
- 去常量池当中定位类的符号引用；
- 检查这个类的符号引用是否被加载，解析和初始化过；
- 如果没有，则进行类加载的过程，第二节会讲到类加载的过程；如果已经加载过，则虚拟机为对象分配内存；
- 虚拟机为分配到的内存空间都赋零值；
- 执行<init>方法，按照程序意愿初始化对象，即执行类的初始化函数。
- 执行完成之后，就得到了一个新的对象。

6详细说说类加载的过程，静态代码块执行在哪个阶段
- 加载–>验证–>准备–>解析–>初始化
- static块的执行发生在“初始化”的阶段。初始化阶段，jvm主要完成对静态变量的初始化，静态块执行等工作。

7业务场景（秒杀防止超卖）
- redis的CAS操作。


10网络了解吗，说说输入网址按下回车后的过程
- 1.协议解析，一般浏览器中都支持多种协议，例如http，https，ftp。

- 2.缓存查询，协议解析之后做缓存查询，看这个URL是否被浏览过。如果浏览过，又不做强制刷新，则不会再次请求。

- 3.浏览器对域名进行解析，将域名转换为IP

- 4.浏览器 请求-处理-响应

- 5.显示

11算法题

两个字符串，按照规则判断相等（重写equals），规则是两个字符串相同字符出现的次数相同，遍判定相等。例（AAB 和 ABA 相等）。

先想了用两个个数组存字符出现次数，然后遍历比较。

面试官想了一下，不要用数组存，时间复杂度允许高一点

两个字符串先用toCharArray()，然后用Arrays.sort()，时间复杂度o(nlogn)


N个人排队，输入M行，每行两个数字（X, Y），代表X比Y高。（X, Y在0~N-1之间）
输出N个人的身高的排列，如果不能排列，则输出false。
当时看到就想到是图的题目，但是自己图这方面做的比较少。输入输出也得自己处理。

写了好久，用回溯做了，面试官说最优解是用入度做，不过回溯做的也可以。
当时确实有点慌，一边想思路一边跟面试官沟通。
面试的时候比较紧张，不过还是用回溯，算是暴力破解了。不过我认为解法可能有点问题。
面试官提了一些可能的情况，比如死循环，不能输出结果。我都解决了这些问题。面试官也认可了我的解法。



1.聊聊sychronized关键字，用法，底层实现，偏向锁，轻量级锁，自旋锁
2.聊聊偏向锁，轻量级锁的原理和过程
3.除了sychronized，还有啥，聊聊ReentrantLock，底层一个继承了AQS的实现类
4.聊聊AQS，volatile修饰的state，加锁过程，公平锁和非公平锁的实现

思考题：三个和尚去打水，等都打完水了才能再次去打水，怎么实现
代码题：两个链表代表的大数进行相加，我用两个栈分别保存链表，然后出栈的时候用头插法生成新链表，感觉时间复杂度没有最优，但是也过了

springmvc的工作流程

- 1、用户发送请求至前端控制器DispatcherServlet。

- 2、DispatcherServlet收到请求调用HandlerMapping处理器映射器。

- 3、处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

- 4、 DispatcherServlet调用HandlerAdapter处理器适配器。

- 5、HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。

- 6、Controller执行完成返回ModelAndView。

- 7、HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。

- 8、DispatcherServlet将ModelAndView传给ViewReslover视图解析器。

- 9、ViewReslover解析后返回具体View.

- 10、DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。 

- 11、DispatcherServlet响应用户。


handlemapping接收的是什么
```
public interface HandlerMapping {
	/**
	 * 获取请求对应的处理类和拦截器
	 */
	HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
}

/**
 * 返回结果：包括具体的Handler，拦截器
 */
public class HandlerExecutionChain {

	// 具体的Handler，不等同于Controller哟。因为HandlerMapping的实现方式有很多，最早的版本，其实handler就是实现了Controller接口的对象，后来的注解版本，其实已经是HandleMethod对象了。不管怎么样，这个handler就是一个可执行请求的对象。
	private final Object handler;
    
    // 拦截器数组，下面还有一个拦截器列表，为什么会有两个，待续……
	private HandlerInterceptor[] interceptors;

	private List<HandlerInterceptor> interceptorList;

	private int interceptorIndex = -1;
    
    // ………………此处省略一批方法
}
```

拦截器和过滤器区别
- 过滤器是在请求进入容器后，但请求进入servlet之前进行预处理的。请求结束返回也是，是在servlet处理完后，返回给前端之前。
- 拦截器能够在方法进入前后，异常抛出前后调用，适用范围更大。

Spring有几种方式定义Bean。


leetcode 41题

start和run方法的区别
- 调用start方法方可启动线程，而run方法只是thread类中的一个普通方法调用，还是在主线程里执行。

redis单线程模型
- 一个bossGroup和一个workerGroup

描述从输入一个url到得到结果的过程

什么时候发生fullgc，minorgc

CMS收集器，为什么要初始标记和重新标记要stop the world？

final、finally、finalize()

threadlocal底层原理

有10亿个电话号码，找出重复的。
编程：将阿拉伯数字转换成中文数字。如（int）123456->十二万三千四百五十六

登录怎么实现权限验证？
- 分布式session

怎么看用expain查sql语句执行计划的结果？
- https://blog.csdn.net/ljl52099/article/details/102079422

分布式session？

zookeeper的应用场景？
zookeeper如何实现分布式锁？

为什么主键要自增，叶子节点key为什么有序？

算法题，打印出一个字符串中所有的回文子串

程序计数器工作原理？作用？不会
JVM调优经历dump之类的操作？无
- 使用printGCDetail命令在控制台打印GC信息。
- 使用jmap命令获取堆快照。

垃圾收集器CMS工作原理? Concurrent体现在哪？用户停顿？垃圾收集为什么要开启多个线程？

索引结构？为什么使用B+索引？
- B树必须用中序遍历的方法按序扫库，而B+树直接从叶子结点挨个扫一遍就完了，B+树支持range-query非常方便，而B树不支持。这是数据库选用B+树的最主要原因。

CAS原理?用CAS实现 两个线程给同一个变量赋值？

AQS等待队列为什么设计成双向链表?
- 在队列同步器中，头节点是成功获取到同步状态的节点，而头节点的线程释放了同步状态后，将会唤醒其他后续节点，后继节点的线程被唤醒后需要检查自己的前驱节点是否是头节点，如果是则尝试获取同步状态。

fork/join?

线程WAITING，BLOCKED状态区别
- blocked 和 waiting 是 Java 线程的两种阻塞状态。

CopyOnWriteList介绍，场景，优缺点

TCP粘包介绍，netty是怎么解决粘包的

什么是回表
mysql为什么推荐主键是自增的

数据量特别大怎么办：分库分表
介绍下分库分表
怎么拆分，拆分的原则

编程题：两个数字相加，数字非常大，字符串传入的

hystrix熔断怎么实现的：不会🙃
redis 哨兵介绍下 master选举过程介绍下

写个题吧，输入一个数字，输出它的中文表示，比如4020，输出四千零二四

leetcode980

查看磁盘，进程的Linux命令？
top
面试常考linux指令
数据库引擎有哪些，区别是什么？
算法题：判断一个链表上的数值是否是回文数？

跳表优点

统计pv



7.3升杯子和5升杯子求4升的水  多方案
8.一副扑克牌放在手上 顺序取 一张放桌子上 一张插入手底 最后桌子上的牌为1-k  求原牌顺序
9.一家965国企 系统出现问题 每三个月挂掉 现已发现是内存泄露导致 让你三天内解决  你会怎么做(项目代码是十年前的那种  难以维护)
10.求二叉树第n层的节点个数




场景设计题：如果让你设计类似于滴滴打车的软件，只考虑后端，那么你会设计哪些模块？

编程题：

给定一个单向链表，里面存放着数字，例如 1 2 3 2 1 ，判断该链表中存放的数字是不是回文数？比如 1 2 3 2 1 就是回文数，而 1 2 3 4 不是。

给定一个数组， 4 5 6 1 2 3 ,是由1 2 3 4 5 6翻转过来的，也可以翻转成 6 1 2 3 4 5 、 3 4 5 6 1 2 ，如何快速在里面查找某个数字的下标，如果找不到，则返回-1

md5算法？


select/poll/epoll
- epoll，通过红黑树组织fd，对于每个fd，到中断处理程序中注册，如果该fd发生了中断，通过回调的方式，内核把数据从网卡中拷贝到内存中，并将socket插入到链表中。
- 具体的参考：https://zhuanlan.zhihu.com/p/93609693

程序计数器工作原理？作用？不会