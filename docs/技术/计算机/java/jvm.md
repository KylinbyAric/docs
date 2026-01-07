JVM

-XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=4 -XX:ConcGCThreads=2 -XX:InitiatingHeapOccupancyPercent=45 -XX:-OmitStackTraceInFastThrow
ConcGCThreads和ParallelGCThreads区别
前者是并发标记期间，一般是ParallelGCThreads的四分之一
后者是STW的线程一般等于CPU数。


#参数篇
-Xms1024M  初始堆内存
-Xmx3072M 最大堆内存
-Xss1M	线程堆大小
-XX:Permsize=10M;           初始方法区内存
-XX:MaxPermSize=10M         最大方法区内存
-XX:MaxMetaspaceSize=10m    最大元空间

XX:InitiatingHeapOccupancyPercent=45  当堆内存已使用45%时出发GC
OmitStackTraceInFastThrow 相同异常堆栈只打印一次？？？
java.lang.OutOfMemoryError: GC overhead limit exceeded 超过98的时间用来GC，且只回收了不到2%的堆内存
java.lang.OutOfMemoryError: Java heap space     堆溢出
java.lang.OutOfMemoryError-->Metaspace          元数据区溢出
java.lang.StackOverflowError                    栈溢出

linux 分析工具
Top 
Vmstat  1  3  每秒执行一次采集 总采集3次


jdk自带分析工具

jps  可以查询Java进程号 
jstack 分析堆栈 
jmap分析内存
Jstat -gc  -t. Pid  1000. 3 
Jvisualvm  启动

Top 
Top -Hp pid
jstack pid |grep Oxpid

jmap -histo:live |head -10  

arthas+火焰图 可以分析cpu

jvisualvm+oql 可以分析内存里的对象



jvisualvm 的使用：
1、查询最高的那些对象
2、查询堆栈中的对象名，可以快速定位



- [ ] 如何制造方法区溢出？
将方法区调小，然后不断加载一个很大字符串。
1.7之后是Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
-XX:permsize=10M -XX:MaxPermSize=10M
static String base="askcto";
public static void main(String[] args) {
    List<String> list = new ArrayList<String>();
    for (int i= 0;i<Integer.MAX_VALUE;i++){
        String str = base +base;
        base = str;
        list.add(str.intern());
    }
}
动态加载技术和JSP技术导致的加载的类太多。



理论知识

Java栈：局部变量表 线程私有 StackOverFlow
本地方法栈：native 方法使用的
堆：实例数据和数组
堆区分为
新生代： 8:1:1
eden：新生的对象
survivor：
    其中一个存满则放到old区，否则from 和 to会来回复制。
    from:新生对象经历几次GC，但又还没到old时存放的。
    to:
老年代：old

对象刚生成时是放到eden区，一次young GC(也称minor GC)，会把eden区和from survivor区中不能回收的对象都放到to survivor区。然后调换一下from和to标示。
始终保持to survivor是空的。如果对象GC年龄超过设定值则会放到old gen。如果是大对象超过Region大小50%的，也是会放入到old gen。old区的GC，则称之为
old GC,也称Full GC。

Eden、survivor、old


方法区：类信息、常量、编译后的代码
直接内存

对象：
一、对象的创建：
1、new 通常是去常量池寻找是否存在类的引用，没有则进行加载
2、类加载：
3、分配内存，分配完之后所有值都是初始值，对于JVM是完成了创建，对于java程序没有，因为默认值还没设置。
	指针碰撞：连续规整，中间存放指针作为分界点，
	空闲列表：不连续 标记清除算法的JVM常用
4、执行init方法，赋予默认值。

二、对象内存布局：header、instance data、padding
1、	对象头又分为两部分：一部分是mark word,GC分代、锁、哈希码 第二部分类型指针，表示是什么类型的。
2、实例数据：各个字段的信息，包括父类继承的和本身定义的
3、填充：只为了填充，保证是字节的整数倍

三、访问定位
通过栈上的reference来访问，主流有两种：
1、句柄访问
	堆中划分出句柄池，每个句柄包含实例数据指针(堆)、类型数据指针(方法区)
￼
2、直接访问
	直接存放实例数据指针、类型数据指针
￼


垃圾回收：
一、对象已死？
1、引用计数法
	会有循环引用问题
2、可达性分析
	用一些对象作为GC Root对象，如果能从该对象往下便利，能遍历到则说明可达，说明还存活。不可达则说明死亡
	可用GC Root对象：1、方法区的静态对象 2、常量饮用的对象 3、JVM栈中的本地变量4、本地方法栈的本地变量5、线程
￼
3、再谈引用
	强引用：new
	软饮用：SoftReference 内存溢出前，回收，如果回收之后还是不够，才报内存溢出
	弱饮用：WeakReference 下一次GC回收 ThreadLocal中的ThreadLocalMap中的entry对象就是继承自WeakReference
	虚饮用：？？完全不影响持有对象的生存时间，只是在虚饮用回收时，通知一下持有对象。PhantomReference.
	
4、生存还是死亡？
被标记为不可达对象之后。进行finalize判断，如果是未覆盖或者已经执行，则不需要再执行，直接回收了。如果覆盖且未执行，则放入F-Queue队列执行，如果执行之后重新和GC Root关联上了。则胡汉三又回来了。否则直接回收。

5、回收方法区
主要回收废弃的常量类和无用的类
前者一般是判断是否有该引用，无则回收
后者是全满足三个条件
1、引用无了
2、该对象的ClassLoader被回收
3、Class对象没任何地方饮用，且无法通过反射访问到(不知道怎么判断的？)

二、垃圾回收算法
1、标记-清除
简单、效率不高、碎片很多。老年代
2、复制算法
1:1 只回收一块，存活的复制到另一端。 一般用于新生代
3、标记整理
存活的移动到另一端，一般用于老年代
4、分代算法
新生代：复制算法
老年代：标记清除、标记整理

三、垃圾回收器
1、CMS
标记清除算法
	初始标记：GC Root
	并发标记：
	重新标记：修正并发标记期间发生变动的标记记录
	并发清除：客户线程和垃圾回收线程一起工作
碎片高、浮动垃圾无法处理
2、G1
标记-整理算法
	初始标记：GC Root。STW  ParallelGCThreads
	并发标记： ConcGCThreads。
	最终标记：修正并发标记期间发生变动的记录 STW
	筛选回收：
可预测停顿、空间整理

四、类加载机制
1、加载
2、验证
元数据验证：是不是有父类啊
字节码验证：类型转换是不是有效的啊(子转父可，不可父转子)
3、准备
分配内存和赋予“0”值
4、解析
符号饮用转为直接饮用 
5、初始化 init
赋予java代码中的初始值

类加载器
类的全限定名转为二进制流的过程，称之为类加载。
对于虚拟机：只有两种 启动类加载器(C++实现)，二是其他加载器。
对于开发者：
启动类 ：bootClassPath里的代码
扩张类：第三方jia包
应用程序类：Classpath里的代码，俗称自己的代码
双亲委派：
对于类加载请求，首先先交给父类加载器去加载，只有父类无法加载时，才会尝试加载。

破坏双亲委派：
JNDI：线程上下文加载器。父类请求子类去加载SPI代码。











