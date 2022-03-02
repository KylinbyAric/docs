
###arthas
jad com.***
watch 



```
tomcat:
- [x] 查看版本 usr/local/tomcat/bin/version.sh

<? extends T> 表示包括T和所有继承T的子类
<? supper T> 表示包括T和所有T的父类

集合：
ConcurrentHashMap——HashMap
CopyOnWriteArrayList—ArrayList
CopyOnWriteArraySet—-HashSet
ConcurrentSkipListMap—-TreeMap
ConcurrentSkipListSet—TreeSet
ConcurrentLinkedQueue—Queue
LinkedBlockingQueue、ArrayBlockingQueue、PriorityBlockingQueue

TreeMap和HashMap
如果是需要可排序的map，则可以使用TreeMap.比如一致性hash算法里就会使用treeMap作为负载容器。

hashmap的问题
1、在1.7之前，1.8后修复。链表是头插法，会导致环形链表，最终导致查询死循环。
原因：并发下，T1，T2都可能同时触发rehash。原先是A->B->null。T1在rehash之后变成B->A->null。而T2在扩容时又将A放到表头，从而A->null变成A->B->A….
从而出现了死循环。
2、并发下会出现多个线程put同一个数组元素，造成覆盖问题。即A->B变成B。

BlockingQueue
深拷贝：
会复制一份引用对象
浅拷贝：
引用会指向原来的对象

queue:
Poll. 获取并清除队头 无则返回null
remove  获取并清除队头 无则throw

peek 获取不移除。无则返回null
Element 获取不移除 无则throw

offer 插入 成true否false
add  插入 不成throw

多线程下queue可以采用linkedBlockingQueue,linkedBlockingQueue采用了reentrantLock保证了多线程下的安全性

多线程保证唯一执行，可以采用AtomicBoolean来控制，常用compareAndSet(exact,update),set(update).

Condition是用于多线程之间的通信，用于lock与unlock之间执行。
condition的signal方法类似Object.notify通知所有调用了condition.await()方法的线程。
condition.await方法类似Object.wait,睡眠，然后等待condition的signal，然后接着运行。

类的缓存可以用concurrentHashMap做为缓存容器。

System.exit(0) 0正常退出 1 异常退出

String 
1.6的String引起的内存泄漏问题，这里的内存泄漏是因为部分内存没有被使用，但又没有被释放导致不可使用而造成的内存浪费现象。

1.6的String是由三部分组成1、value[] 2、offerset 3、count。
在substring中字符的截取是复用value[]，只是改变offerset和count。假设旧字符是100字节，新字符是1字符。如果旧字符被释放，但新字符却还在使用，导致实际使用1字节，但实际却占用100字节，99个字节没有被释放而导致浪费。

弱引用：如果只有一个对象只有一个弱引用，那么下次GC不管内存空间是否充足，都会回收他的内存。通常是用于本地缓存，ThreadLocal。
软引用：如果一个对象只有一个软引用，那么当空间不足时才会回收，比弱引用强一点

Preconditions.checkState();
Preconditions.checkNotNull();
如果是局部式的enum直接这样设置就好了
private enum State
{
    LATENT,
    STARTED,
    CLOSED
}


aThread.join()是指当前线程要等待aThread执行完再继续运行

putIfAbsent(K,V)：如果不存在则存入V并返回null,否则不覆盖并返回旧值
put(),直接覆盖旧值，

AtomicInteger timeoutCount = jobTimeoutCountMap.putIfAbsent(jobId, new AtomicInteger(1));


private static final ThreadLocal<PrintRecord> SHINE_MANAGER_HOLDER = ThreadLocal.withInitial(PrintRecord::new);

ThreadLocal+filter形成一个上下文，这样不同的请求，上下文则完全不一样，

public static PrintRecord getInstance() {
    return (PrintRecord) SHINE_MANAGER_HOLDER.get();
}

public void close() {
    SHINE_MANAGER_HOLDER.remove();
}

@sharding的原理
在aop时将决定当前datasource的key替换成所需要的数据源key,从而达到分库的作用。


Thread.interrupt()的做法是设置中断标记位，也就是Thread.isInterrupt()返回为true.如果线程处于sleep、wait、join等状态，则会抛出异常。也就是并不会中断正在运行的线程。
常用的做法是：
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                /*
                 * 如果线程阻塞，将不会去检查中断信号量stop变量，所 以thread.interrupt()
                 * 会使阻塞线程从阻塞的地方抛出异常，让阻塞线程从阻塞状态逃离出来，并
                 * 进行异常块进行 相应的处理
                 */
		doWork();
                Thread.sleep(1000);// 线程阻塞，如果线程收到中断操作信号将抛出异常
            } catch (InterruptedException e) {
                System.out.println("Thread interrupted...");
                /*
                 * 如果线程在调用 Object.wait()方法，或者该类的 join() 、sleep()方法
                 * 过程中受阻，则其中断状态将被清除
                 */
                System.out.println(this.isInterrupted());// false

                //中不中断由自己决定，如果需要真真中断线程，则需要重新设置中断位，如果
                //不需要，则不用调用
                Thread.currentThread().interrupt();
            }
        }
        System.out.println("Thread exiting under request...");
    }


public class MyStack {  
    private List<String> list = new ArrayList<String>();  
  
    public synchronized void push(String value) {  
        synchronized (this) {  
            list.add(value);  
            notify();  
        }  
    }  
  
    public synchronized String pop() throws InterruptedException {  
        synchronized (this) {  
            if (list.size() <= 0) {  
                wait();  
            }  
            return list.remove(list.size() - 1);  
        }  
    }  
}  




StringUtils.join(registryList, ",");


Thread.setDeamon();
设置之后，那么该线程则会守护线程，优先级比较低，如果没有其他用户线程了，则该线程自动退出，不管run有没有完成都会强制退出。常用于监听等服务线程。
通常的守护线程处理方式是
Thread.setDeamon();
Thread.start();
volatile boolean toStop = false;

run(){
	while(!toStop){
		doWork();
	}
}

new HashMap(){{
	put();
}}
某个线程优先级大于线程所在线程组的最大优先级，那么该线程的优先级将会失效，取而代之的是线程组的最大优先级。


Object.wait（）必须是在sync里面，wait是释放对象锁，并且进入睡眠，等待notify。所以wait是必须在有锁的情况下才有效，所以必须是在sync中。

sync加在静态方法里，则锁住的是该方法所在类的Class对象
sync加在常用方法里，则锁住的是该方法所在类的实例对象



LockSupport是可以让线程直接挂起/唤醒,park/unpark成对使用的。尽量不要单独出现。
Java线程的runnable 包括操作系统的ready和running
Thread.sleep()期间是放弃CPU争用权，但不会放弃同步区的锁。如果当前线程用while(1)仅用于等待，那么可以用sleep(0)让出CPU。让其他线程使用CPU。
join的原理是持有b线程的a线程，调用b.wait(),当b terminal之后，会调用b.notifyAll()唤醒a线程。

sleep和wait
sleep只释放cpu资源,不释放锁。所以sleep是容易造成死锁的。

ThreadLocal通常是用于实现线程中的同一静态变量，不同线程不同变量值。对比创建私有变量，达到轻量化。


内存屏障和volatile
被volatile修饰的变量a,普通变量b
一、
那么对于a读操作，JMM就会在这个操作之后插入一个LoadStore屏障和LoadLoad屏障,表示a读操作之后的所有写操作和读操作都不能排在这个a读操作之前。
a读操作语义前的读写操作则可以重排序。比如
b读; //1
b写; //2
a读; //3
也就是可能出现132、213、321、312都是可以的。
但是如果是a读之后的读写操作，则一定是在a读之后执行。比如
a读; //1
b读; //2
b写; //3
这时，一定会1最先执行，然后23可以重排序。

二、那么对于a写操作，JMM就会在这个操作之前插入一个StoreStore屏障,之后插入StoreLoad屏障。表示a写操作之前的写操作都必须在该操作之前，a写操作之后的读操作都必须在该操作之后。
a写操作语义前的读写操作则可以重排序。比如
b写; //1
a写; //2
b读; //3

也就一定是123
但是如果是a读之后的读写操作，则一定是在a读之后执行。比如
a读; //1
b读; //2
b写; //3
这时，一定会1最先执行，然后23可以重排序。


对象锁：本质是个逻辑概念。实质是对象头中的Mark Work的存储信息。
￼
mark word存储的信息变化
偏向锁：存储的是偏向的线程id，也就是经常使用该锁的线程ID，如果线程发现是自己的ID，则无需后面的一些加锁操作，减少同一线程反复获取锁的性能消耗。

如果发现mark word中的线程id不是自己的id，则表明已经有线程在占用了。那么会进行cas操作，替换这个线程id。
成功转换，表示之前的线程id已经不在了，那么锁不升级，认为偏向锁。
失败，则挂起之前占用的线程，设置偏向锁标示为0，锁标示为00，也就是升级锁为轻量级锁。

轻量级锁：
加锁
如果发现是轻量级锁，则尝试cas替换成自己的线程ID。
成功：则获得锁
失败：则自旋，直到获得锁，如果自旋失败，则升级为重量级锁。

释放锁
cas替换mark word。如无其他线程在争用锁，则直接释放成功。如果有其他锁在争用(升级为重量锁了)，mark word 的版本和当前的版本不一致，cas则会失败，但也会释放锁，并且唤醒等待的争用线程。
￼
重量级锁
Contention List：所有请求锁的线程将被首先放置到该竞争队列
Entry List：Contention List中那些有资格成为候选人的线程被移到Entry List
Wait Set：那些调用wait方法被阻塞的线程被放置到Wait Set
OnDeck：任何时刻最多只能有一个线程正在竞争锁，该线程称为OnDeck
Owner：获得锁的线程称为Owner
!Owner：释放锁的线程


锁升级流程：
每一个线程在准备获取共享资源时： 第一步，检查MarkWord里面是不是放的自己的ThreadId ,如果是，表示当前线程是处于 “偏向锁” 。
第二步，如果MarkWord不是自己的ThreadId，锁升级，这时候，用CAS来执行切换，新的线程根据MarkWord里面现有的ThreadId，通知之前线程暂停，之前线程将Markword的内容置为空。
第三步，两个线程都把锁对象的HashCode复制到自己新建的用于存储锁的记录空间，接着开始通过CAS操作， 把锁对象的MarKword的内容修改为自己新建的记录空间的地址的方式竞争MarkWord。
第四步，第三步中成功执行CAS的获得资源，失败的则进入自旋 。
第五步，自旋的线程在自旋过程中，成功获得资源(即之前获的资源的线程执行完成并释放了共享资源)，则整个状态依然处于 轻量级锁的状态，如果自旋失败 。
第六步，进入重量级锁的状态，这个时候，自旋的线程进行阻塞，等待之前线程执行完成并唤醒自己。

个人理解是：
1、如果是偏向锁，则判断是否是自己在持有，是则获得锁并保持偏向锁。如果不是，则cas，成功则获得锁并保持偏向锁。失败，则自旋。升级为轻量级锁。如果偏向锁争用情况很多，那么偏向锁会是累赘，可以关闭-XX:UseBiasedLocking=false。
2、升级之后，通知之前的线程暂停，将mark word的内容置空,然后唤醒之前的线程。然后两个线程cas，把自己的线程id放到mark word。成功的那个线程获得锁，失败的自旋。
3、自旋过程中cas成功，则获得锁，则仍是轻量级锁。自旋失败(非cas失败，通常是指多次cas失败)，则升级为重量级锁。自己进入到竞争队列，然后进行阻塞，等待唤醒。
4、升级为重量级锁之后，原先占用轻量级锁的线程cas时会因为锁升级而失败，接着就释放锁，然后唤醒等待的线程。

三种锁的优缺点：
偏向锁			无需加锁解锁		如果多个线程竞争还是会有额外的锁撤销			适用用多数情况只有一个线程的场景
轻量级锁			不会阻塞			会有自旋消耗CPU							
重量级锁			无自旋CPU消耗	阻塞，响应慢								适用吞吐量大的

synchronized缺点：
1、抢占只能阻塞，不能自动退出，或者超时退出
2、无法知道是否锁住
3、无法中断正在获取锁的线程
4、只有独占模式，没有共享，不能实现读写分离
5、无法中途释放锁，如IO时



Thread.yield()让出自己的CPU时间，重新竞争CPU(可能是自己，也可能是别人)，通常用于自旋等待某个条件，但又让出CPU。

CAS ABA问题：
线程1：获取值100 期望值50
线程2：获取值100 期望值50
线程3：获取值50 	期望值100
正确顺序是1、2、3   正确值是100
实际并发可能出现 132，导致出现错误值50
解决是每次的获取值添加版本号，如AtomicStampedReference利用int标记修改记录，从而避免


AQS:是一个线程同步器，由一个双向队列和一个单向队列（指向下一个waiter）组成。底层是unSafe(实现cas)、LockSupport(用于挂起和恢复线程)。
acquireQueued:
如果前置节点是头节点，则会自旋tryAcquire.成功后将当前节点置为头节点。然后结束
如果前置节点不是头节点，则自旋直到将自身挂起。

变量waitStatus则表示当前被封装成Node结点的等待状态，共有4种取值CANCELLED、SIGNAL、CONDITION、PROPAGATE。
CANCELLED：值为1，在同步队列中等待的线程等待超时或被中断，需要从同步队列中取消该Node的结点，其结点的waitStatus为CANCELLED，即结束状态，进入该状态后的结点将不会再变化。
SIGNAL：值为-1，被标识为该等待唤醒状态的后继结点，当其前继结点的线程释放了同步锁或被取消，将会通知该后继结点的线程执行。说白了，就是处于唤醒状态，只要前继结点释放锁，就会通知标识为SIGNAL状态的后继结点的线程执行。表示是可唤醒的。
CONDITION：值为-2，与Condition相关，该标识的结点处于等待队列中，结点的线程等待在Condition上，当其他线程调用了Condition的signal()方法后，CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁。
PROPAGATE：值为-3，与共享模式相关，在共享模式中，该状态标识结点的线程处于可运行状态。
0状态：值为0，代表初始化状态。
>0则是无效节点。<0是有有效节点
整个流程是先tryAcquire()，走用户自定义的逻辑。成功则获取到了锁。不成功则进入AQS默认逻辑。默认逻辑是先尾插法入队，随后如果是二号，就会自旋tryAcquire，成功则设置自己为head，并且前head的next置为null，help gc。若不是二号，则寻找队伍中上一个signal的node，挂在它后面。然后park就好了。一旦被唤醒，则进入自旋逻辑。

￼

doAcquireShared
与独占模式不同的是，如果二号获得了资源，且有剩余资源则会唤醒下一个，也就是三号，以此类推。然后二号变成1号，3号变成二号




reentrandlock的原理
lock:尝试acquire(cas)。获取成功则继续执行业务。获取失败则进入AQS的队列，然后lockSupport挂起。
unlock:释放资源(cas)。成不成功都会找到AQS中的队列中的下一个有效节点，然后唤醒。
根据公平或者非公平，公平则FIFO。非公平则当前的lock会cas获取一次资源，成功则直接执行业务，否则也进入等待队列。相比于队列中的线程被挂起，当前线程能有一次获取资源的机会，那确实不算是公平。
可重入性是怎么保证的？可重入性是指一个正在持有资源的线程，可以再次获取到资源，其他人不能获取
在try方法上，如果
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
	//如果是当前线程占有，也是可以再次获得锁的。
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}


reentrantLock:
AQS是怎么实现condition唤醒线程的？
condition.signal()会去寻找condition queue里下一个符合唤醒条件的线程，并且把当前线程放入到sync 队列(为什么要放进去呢？是因为await在循环等待该node放入sync queue)。
condition.await()会将当前node加入到wait queue。然后自旋判断当前node是否在sync queue。不在则挂起，在则说明被唤醒/或者有condition.signal则继续执行。

countlatchdown:队列中只会有一个阻塞线程
实例化时将state设置为初始值
await()则是park当前线程
countDown()则是每次将state自减1,如果等于0了，则unpark线程

BlockingQueue:
1、怎么实现阻塞的同步？
用reentrantlock 实现
2、怎么实现为空时获取阻塞以及队列满时插入阻塞？
用reentantLock的condition方式标记消费(notEmpty)和生产线程(notFull)。每次插入，notEmpty.signal(),如果满了，则notFull.await();每次获取，notFull.signal(),为空时，noEmpty.await();类似操作系统中信号量的设置。


sync关键字的不足：
1、只能是排他锁，在只读时性能不高。
2、不够灵活。无法在内部有IO/sleep时释放锁。
3、只能是非公平锁
综上，性能很差。





/**
 * Thread state for a thread which has not yet started.
 */
NEW,

/**
 * Thread state for a runnable thread.  A thread in the runnable
 * state is executing in the Java virtual machine but it may
 * be waiting for other resources from the operating system
 * such as processor.
 */
RUNNABLE,

/**
 * Thread state for a thread blocked waiting for a monitor lock.
 * A thread in the blocked state is waiting for a monitor lock
 * to enter a synchronized block/method or
 * reenter a synchronized block/method after calling
 * {@link Object#wait() Object.wait}.
 */
BLOCKED,

/**
 * Thread state for a waiting thread.
 * A thread is in the waiting state due to calling one of the
 * following methods:
 * <ul>
 *   <li>{@link Object#wait() Object.wait} with no timeout</li>
 *   <li>{@link #join() Thread.join} with no timeout</li>
 *   <li>{@link LockSupport#park() LockSupport.park}</li>
 * </ul>
 *
 * <p>A thread in the waiting state is waiting for another thread to
 * perform a particular action.
 *
 * For example, a thread that has called <tt>Object.wait()</tt>
 * on an object is waiting for another thread to call
 * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
 * that object. A thread that has called <tt>Thread.join()</tt>
 * is waiting for a specified thread to terminate.
 */
WAITING,

/**
 * Thread state for a waiting thread with a specified waiting time.
 * A thread is in the timed waiting state due to calling one of
 * the following methods with a specified positive waiting time:
 * <ul>
 *   <li>{@link #sleep Thread.sleep}</li>
 *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
 *   <li>{@link #join(long) Thread.join} with timeout</li>
 *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
 *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
 * </ul>
 */
TIMED_WAITING,

/**
 * Thread state for a terminated thread.
 * The thread has completed execution.
 */
TERMINATED;


LinkedHashMap:如何能实现LRU。
0、LRU时需要设置 按插入顺序(默认) accessOrder = false | 按访问顺序 accessOrder = true
1、尾插法 tail为最近使用的节点 head是最近最少使用的节点
2、访问过的节点会放入链尾  原理是通过afterNodeAccess方法在put、get、replace等访问方法回调
3、最近最少使用的节点会在被最先删除。
4、底层结构和HashMap一模一样，新增一个双向列表
5、查询节点很快，如果只是双向链表结果，那么查询是O(n)，而如果是hashMap，则快得多；插入也是需要先查询，也是O(n)。综上，用LinkedHashMap性能要快得多。

public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private static final long serialVersionUID = 1L;
    protected int maxElements;

    public LRUCache(int maxSize) {
        super(maxSize, 0.75F, true);
        this.maxElements = maxSize;
    }

    protected boolean removeEldestEntry(Entry<K, V> eldest) {
        return this.size() > this.maxElements;
    }
}
多线程版就新增一个Lock类，在get、set、contain里新加lock.lock

ConcurrentHashMap:
1、1.8 采用局部sync+cas实现并发安全
在put时新加入节点时采用cas减少锁竞争。hash冲突时sync 链表头节点或者红黑树根节点，实现并发。
2、1.7 
总的数组分为多个segment，segment继承ReentrantLock。segment最大为16，也就是最大并发数是16，每个segment之后的数组是可以扩容的。但16个线程仍旧是个限制，比1.8的锁粒度要粗一些。
￼
put的过程：
1、hash算法得出下标
2、如果下标元素为空，则封装成entry或node
3、如果不为空，则看看红黑树还是链表。
	红黑树：封装成红黑树node，遍历插入或更新
	链表：尾插法，遍历更新或插入。插入后长度>=8时转红黑
	插入完成后再判断是否扩容

扩容：
1.7:每个segment都是小型的hashMap.先生成新的2倍容量数组，然后转移数据。每个segment单独判断
1.8:如果正在扩容，则帮助扩容????,将原来的数组分组，分给每一个线程去扩容。



线程池
核心池
最大池
拒绝策略：抛错、当前线程(提交任务的线程，非线程池的线程)执行、直接抛弃、抛弃最老的任务执行
四大线程池：
newFixedThreadPool：固定线程池
newSingleThreadExecutor：单例模式
newScheduledThreadPool：支持定时执行任务 
原理是将原先的task包装成ScheduledFutureTask
在原先run方法的外层添加判断是否应该执行以及执行完是否需要再添加进队列等工作。
newCachedThreadPool：有线程就执行，无线程空闲就创建线程执行，因为可以无限创建线程，有内存泄露风险。 SynchronousQueue


如何实现定时？

如何监控？
通过线程池的一些自带参数：
方法	含义
getActiveCount()	线程池中正在执行任务的线程数量
getCompletedTaskCount()	线程池已完成的任务数量，该值小于等于taskCount
getCorePoolSize()	线程池的核心线程数量
getLargestPoolSize()	线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过，也就是达到了maximumPoolSize
getMaximumPoolSize()	线程池的最大线程数量
getPoolSize()	线程池当前的线程数量
getTaskCount()	线程池已经执行的和未执行的任务总数




 Runtime.getRuntime().availableProcessors()获取当前CPU核数；


T是方法级别的模版类
public <T> Future<T> submit(Runnable task, T result) {
}
```

###ThreadLocal
ThreadLocalMap是由一个Entry数组组成的，Entry是一个<ThreadLocal实例弱引用，value>的结构。
每一个Tread都会持有ThreadLocal.ThreadLocalMap实例，用于存放变量副本。
1、每一个ThreadLocal实例都会持有一个唯一自增的ID。
2、set的原理是将当前线程的ThreadLocalMap取出(没有则帮他new一下)，以ThreadLocal实例作为key存放进去。
3、get的原理是取出当前线程的ThreadLocalMap取出，然后以ThreadLocal实例作为key去hash取出。

问题：
1、为什么Entry的key是thread弱引用?
如果是强引用，那么如果没有手动删除，Thread.ThreadLocalMap将继续持有ThreadLocad的强引用，导致ThreadLocal不会被回收，造成内存泄漏。
而弱引用则会回收。
2、ThreadLocal是怎么产生内存泄漏的？
当ThreadLocal被回收时，Entry<ThreadLocal,value>会变成Entry<null,value>。也就是Thread.ThreadLocalMap还是持有value的强引用。导致
value无法被回收。造成内存泄漏。
解决办法就是在每次调用完get\set之后，都要调用remove()。remove会移除key和value。这样就不会有value的强引用，也就是不会导致内存泄漏。
小结：根源是1、没有remove导致value有强引用 2、value的生命周期和Thread绑定，Thread不GC，value就不会被GC。

###TreeMap
TreeMap是一颗红黑树，每一个Entry都持有left、right、parent引用。
getCeilingEntry(key)方法：从root开始向下遍历，找到比key小的上一个值。原理对比二叉树的遍历。
getFloorEntry(key)方法：从root遍历，找到比key大的下一个值。
getFirstEntry():从root开始一直向左遍历，找到最左的节点，返回。
getLastEntry():从root开始一直向右遍历，找到最右的节点，返回。
fixAfterDeletion(E) 每次delete之后都会进行红黑树的调整。
fixAfterInsertion(E) 每次put之后都会进行红黑树的调整。


###雪花算法
核心思想是使用64bit的long类型数字作为全局唯一id。
第一部分：最高位 0 无意义 只是为了输出正数
第二部分： 41bit 表示时间戳  当前时间-起始时间 单位毫秒
第三部分： 5bit 表示机房id  32个
第四部分： 5bit表示机器ID    32个 即支持 32 * 32 = 1024个节点
第五部分： 12bit 表示毫秒内的顺序号。 4096个

优点:
1、一段时间内--69年生成的id是唯一的
2、获取是纯内存操作，性能高
3、生成的id自增，存到数据库中时，索引效率高

缺点：
1、依赖系统时间，如果系统时间被回拨了，就会重复生成
改进：记录上次的时间戳，如果下次生成的时间戳小于上次，可以抛异常提示

2、毫秒内如果需要生成超过4096个，则无法支持。一般不太可能。

3、有id间隙，一段时间没生成，那么这段时间的id不能再生成了。


```java

/**
 * Twitter_Snowflake<br>
 * SnowFlake的结构如下(每部分用-分开):<br>
 * 0 - 0000000000 0000000000 0000000000 0000000000 0 - 00000 - 00000 - 000000000000 <br>
 * 1位标识，由于long基本类型在Java中是带符号的，最高位是符号位，正数是0，负数是1，所以id一般是正数，最高位是0<br>
 * 41位时间截(毫秒级)，注意，41位时间截不是存储当前时间的时间截，而是存储时间截的差值（当前时间截 - 开始时间截)
 * 得到的值），这里的的开始时间截，一般是我们的id生成器开始使用的时间，由我们程序来指定的（如下下面程序IdWorker类的startTime属性）。41位的时间截，可以使用69年，年T = (1L << 41) / (1000L * 60 * 60 * 24 * 365) = 69<br>
 * 10位的数据机器位，可以部署在1024个节点，包括5位datacenterId和5位workerId<br>
 * 12位序列，毫秒内的计数，12位的计数顺序号支持每个节点每毫秒(同一机器，同一时间截)产生4096个ID序号<br>
 * 加起来刚好64位，为一个Long型。<br>
 * SnowFlake的优点是，整体上按照时间自增排序，并且整个分布式系统内不会产生ID碰撞(由数据中心ID和机器ID作区分)，并且效率较高，经测试，SnowFlake每秒能够产生26万ID左右。
 */
public class SnowflakeIdWorker implements Serializable {
    private static final long serialVersionUID = 1L;

    // ==============================Fields===========================================
    /**
     * 开始时间截 (2017-11-23,1511366400000L 15位) 2019-06-12 14:41:52
     */
    private final long twepoch = 1560321712000L;

    /**
     * 机器id所占的位数
     */
    private final long workerIdBits = 4L;

    /**
     * 数据标识id所占的位数
     */
    private final long datacenterIdBits = 4L;

    /**
     * 支持的最大机器id，结果是31 (这个移位算法可以很快的计算出几位二进制数所能表示的最大十进制数)
     */
    private final long maxWorkerId = -1L ^ (-1L << workerIdBits);

    /**
     * 支持的最大数据标识id，结果是31
     */
    private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);

    /**
     * 序列在id中占的位数
     */
    private final long sequenceBits = 9L;

    /**
     * 机器ID向左移12位
     */
    private final long workerIdShift = sequenceBits;

    /**
     * 数据标识id向左移17位(12+5)
     */
    private final long datacenterIdShift = sequenceBits + workerIdBits;

    /**
     * 时间截向左移22位(5+5+12)
     */
    private final long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;

    /**
     * 生成序列的掩码，这里为4095 (0b111111111111=0xfff=4095)
     */
    private final long sequenceMask = -1L ^ (-1L << sequenceBits);

    /**
     * 工作机器ID(0~31)
     */
    private long workerId;

    /**
     * 数据中心ID(0~31)
     */
    private long datacenterId;

    /**
     * 毫秒内序列(0~4095)
     */
    private long sequence = 0L;

    /**
     * 上次生成ID的时间截
     */
    private long lastTimestamp = -1L;

    //==============================Constructors=====================================

    /**
     * 构造函数
     *
     * @param workerId     工作ID (0~31)
     * @param datacenterId 数据中心ID (0~31)
     */
    public SnowflakeIdWorker(long workerId, long datacenterId) {
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        this.workerId = workerId;
        this.datacenterId = datacenterId;
    }

    // ==============================Methods==========================================

    /**
     * 获得下一个ID (该方法是线程安全的)
     *
     * @return SnowflakeId
     */
    public synchronized long nextId() {
        long timestamp = timeGen();

        //如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(
                    String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }

        //如果是同一时间生成的，则进行毫秒内序列
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            //毫秒内序列溢出
            if (sequence == 0) {
                //阻塞到下一个毫秒,获得新的时间戳
                timestamp = tilNextMillis(lastTimestamp);
            }
        }
        //时间戳改变，毫秒内序列重置
        else {
            sequence = 0L;
        }

        //上次生成ID的时间截
        lastTimestamp = timestamp;

        //移位并通过或运算拼到一起组成64位的ID
        return ((timestamp - twepoch) << timestampLeftShift) //
                | (datacenterId << datacenterIdShift) //
                | (workerId << workerIdShift) //
                | sequence;
    }

    /**
     * 阻塞到下一个毫秒，直到获得新的时间戳
     *
     * @param lastTimestamp 上次生成ID的时间截
     * @return 当前时间戳
     */
    protected long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }

    /**
     * 返回以毫秒为单位的当前时间
     *
     * @return 当前时间(毫秒)
     */
    protected long timeGen() {
        return System.currentTimeMillis();
    }

    //==============================Test=============================================

    /**
     * 测试
     */
    public static void main(String[] args) throws InterruptedException {
        SnowflakeIdWorker idWorker = new SnowflakeIdWorker(0, 0);
        for (int i = 0; i < 5000; i++) {
            long id = idWorker.nextId();
            System.out.println(id + ":" + (id + "").length());
        }
    }
}
```

###StopWatch 一个粗糙的性能统计工具,可以很好得用作采集性能。

stream.filter是过滤出符合条件的元素
stream.map是映射每一个元素中为R
```java
List<String> list1 = list.stream().map(a -> a.a).collect(Collectors.toList());
```
list->map
```java

Map<String, List<Student>> stuMap = stuList.stream().filter((Student s) -> s.getHeight() > 160) .
collect(Collectors.groupingBy(Student ::getSex)); 
```

###异常怎么处理？
受查异常：编译的时候就会检查你是否处理的异常，比如一个函数抛出受查异常，而上层调用函数没有处理的话，编译的时候则会报错
非受查异常：运行时才抛的异常。

是否要丢弃，看调用方是否在意这个异常
是否直接上抛，看更上层是否能理解这个异常，否则就得封装成上层理解的异常。都在直接抛出去，上层也不知道如何处理啊。













