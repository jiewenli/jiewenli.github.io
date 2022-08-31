# JAVA并发


# JAVA并发
- ## 基础知识
    1. ### 并发编程的优缺点
        1. 为什么要用到并发（优点）
            - 充分利用多核CPU的运算能力
            - 方便业务拆分
        2. 并发编程的缺点
            - 频繁的上下文切换
            - 线程安全（常见的避免死锁的方式）
        3. 易混淆的概念
            - 同步 VS 异步
            - 并发 VS 并行
            - 阻塞 VS 非阻塞
            - 临界区资源
    2. ### 线程的状态和基本操作
        1. 如何新建线程
            - 继承Thread类
            - 实现Runnable接口
            - 实现Callable接口
            - ***几种新建线程方式的比较***
        2. 线程状态的转换
            - NEW
            - RUNNABLE
            - WAITING
            - TIMED_WAITING
            - TERMINATED
            - BLOCKED
        3. 线程的基本操作
            - interrupt
                - 抛出InterruptedException异常时，会清除中断标志位
                - ***interrupt(), interrupted(), isInterrupt()***
            - sleep
                - ***sleep与wait的区别***
            - join
            - yield
                - yield仅仅只会把时间片让给同优先级的线程，而sleep没有这个要求
        4. ***守护线程Daemon***
- ## 并发理论 JMM
    1. ### JMM内存模型
        1. 哪些是共享数据
            - 实例域
            - 静态域
            - 数组
        2. 抽象结构
            - 线程将数据拷贝到工作内存，再刷新到主存。
            - 各个线程通过主存中的数据来完成隐式通信
    2. ### 重排序
        1. 什么是重排序
            - 为了提高执行性能，编译器和处理器会对指令进行重排序
            - 针对编译器重排序，编译器重排序规则会禁止一些特定类型的编译器重排序
            - 针对处理器重排序，编译器会在生成指令的时候插入内存屏障来禁止特定类型的处理器重排序
        2. 数据依赖性
        3. as-if-serial
            - 遵守as-if-serial语义的编译器，runtime和处理器共同为编写单线程程序的程序员创建了一个幻觉：单线程程序是按程序的顺序来执行的
    3. ### happens-before规则
        1. 定义
            - 如果A happens-before B，则A操作的结果对操作B可见，且A操作在操作B之前执行
            - 如果指令重排序之后的结果，与按照happens-before关系执行的结果一致，则指令可以重排序
        2. 理解
            - 站在程序员角度：为编程人员提供了一个类似强内存的内存结构，方便编程
            - 站在编译器和处理器厂商：在不影响正确结果的前提下，可以让编译器和处理器厂商尽情优化
        3. 具体规则
            - 程序顺序规则
            - volatile变量规则
            - 监视器锁规则
            - 传递性
            - start规则
            - join规则
            - 线程中断规则
            - 对象finnalize规则
- ## 并发关键字
    1. ### ***synchronized***
        1. 如何使用？
            - 实例方法（锁的是实例对象）
            - 静态方法（锁的是类对象）
            - 代码块根据配置，锁的是实例对象也可以是类对象
        2. moniter机制
            - 字节码中会添加monitorenter和monitorexit指令
            - 锁的重入性：同一个锁程，不需要再次申请获取锁
        3. synchronized的happens-before关系
        4. synchronized的内存语义
            - 共享变量会刷新到主存中，线程每次会从主存中读取最新的值到自身的工作内存中
        5. ***锁优化***
            - 锁状态
                - 无锁状态
                - 偏向锁
                - 轻量级锁
                - 重量级锁
            - CAS操作
                - 是一种乐观锁策略
                - 利用现代处理器的CMPXCHG
                - 存在ABA的问题；自旋时间可能过长的问题
            - JAVA对象头
                - 对象的hashcode
                - 对象的分代年龄
                - 是否是偏向锁的标志位
                - 锁标志位
        6. ***锁升级策略***
            - 轻量级锁
                - 加锁：在对象头和栈帧的锁记录中，添加自身的线程ID
                - 锁撤销：在全局安全点上进行
            - 轻量级锁
                - 加锁：Displace mark word，对象头mark word通过CAS指向栈中锁记录
                - 锁撤销：如果CAS替换回对象头失败，则升级成重量级锁
            - 重量级锁
            - ***各种锁的比较***
        - [Java锁与线程的那些事](https://tech.youzan.com/javasuo-yu-xian-cheng-de-na-xie-shi/)
        - [Java6及以上版本对synchronized的优化](https://www.cnblogs.com/wuqinglong/p/9945618.html)
    2. ### ***volatile***
        1. 实现原理
            - 写volatile变量在编译时添加Lock指令
            - 缓存一致性：每个处理器会通过总线嗅探出自己的工作内存中数据是否发生变化
        2. happens-before关系的推导
        3. 内存语义
            - 写volatile变量会重新刷新到主存中，其他线程读volatile变量会重新从主存中读取最新值
        4. 内存语义的实现
            - 通过在特定位置处插入内存屏障来防止重排序
    3. ### final
        1. 如何使用
            - 变量
            - 基本类型
                - 类变量（static变量）：只能在声明时赋值或者
                - 在静态代码块中赋值
                - 实例变量：声明时赋值，构造器以及非静态代码块中赋值
                - 局部变量：有且仅有一次赋值机会
            - 引用类型
                - final修饰的引用类型只保证引用的对象地址不变，其对象
                的属性是可以改变的
                方法
                - 被final修饰的方法不能被子类重写，但是可以被重载
                类
                - 被final修饰的类，不能被子类继承
        2. final的重排序规则
            - final域为基本类型
            - 禁止对final于的写重排序到构造函数之外
            - 禁止读对象的引用和读该对象包含的final域重排序
            - final域为引用类型
            - 对一个final修饰的对象的成员域的写入，与随后在构造函数之外
            - 把这个被构造的对象的引用赋给一个引用变量，这两个操作是不能被重排序的
        3. final实现原理
            - 插入StoreStore和LoadLoad内存屏障
        4. finla引用不能从构造函数函数中“溢出”(this逃逸)
    4. ### 三大性质
        - 原子性
            - synchronzied
        - 可见性
            - synchronized
            - volatile
        - 有序性
            - synchronized
            - volatile
- ## Lock体系
    1. ### Lock与synchronized的比较
        - Lock提供了基于API的可操作性，提供能可响应中断式获取锁，超时获取锁以及非阻塞式获取锁的特性
        - synchronized执行完同步块以及遇到异常会自动释放锁，而Lock要显示的调用unlock方法释放锁
    2. ### ***AQS***
        1. 设计意图（模板方法设计模式）
            - AQS提供给同步组件实现者，为其屏蔽了同步状态的管理，线程排队等底层操作实现者只需要通过AQS提供的模板方法实现同步组件的语义即可
            - lock（同步组件）是面向使用者的，定义了接口，隐藏了实现细节
        2. 如何使用AQS实现自定义同步组件
            - 重写protected方法，告诉AQS如何判断当前同步状态获取是否成功或者失败
            - 同步组件调用AQS的模板方法，实现同步语义。而提供的模板方法又会调用被重写的方法
            - 实现自定义同步组件时，推荐采用继承AQS的静态内部类
        3. 可重写的方法
        4. ***AQS提供的模板方法***
    3. ### AQS源码解析
        1. AQS同步队列的数据结构
            - 带头结点的双向链表实现的队列
        2. ***独占式锁***
            - 同步状态获取成功则退出；失败则通过addWaiter方法将当前线程
            - 封装成节点加入同步队列，acquireQueued方法使得当前线程等待获取同步状态
            - 如果获取同步状态并且是同步队列中的头结点，则表明获取锁成功，并唤醒后继结点
            - 可响应中断式获取锁以及超时获取锁特性的实现原理
        3. ***共享式锁***
            - 锁获取原理
            - 锁释放原理
            - 可响应中断式获取锁以及可超时获取锁特性的实现原理
        4. ***ReentrantLock***
            1. 重入锁的实现原理
            2. 公平锁的实现原理
            3. 非公平锁的实现原理
            4. 公平锁和非公平锁的比较
        5. ***ReentrantReadWriteLock***
            1. 如何表示读写状态的
            低16位用来表示写状态，高16位用来表示读状态
            2. WriteLock的获取和释放
            当ReadLock已经被其他线程获取或者WriteLock被其他线程获取，当前线程
            获取WriteLock失败；否则获取成功。支持重入性
            WriteLock释放时将写状态通过CAS操作减一
            3. ReadLock的获取和释放
            当WriteLock已经被其他线程获取的话，ReadLock获取失败；否则获取成功。支持重入性
            通过CAS操作将读状态减一
            4. 锁降级策略
            按照WriteLock.lock()-->ReadLock.lock()-->WriteLock.unlock()的顺序，WriteLock会降级为ReadLock
            5. 生成Condition等待队列
            WriteLock可以通过newCondition方法生成Condition等待队列，
            而ReadLock无法生成Conditon等待队列
            6. 应用场景
            适用于读多写少的应用场景，比如缓存设计上
        6. ***Condition机制***
            1. 与Object的wait/notify机制相比
            具有的特性
            Condition能够支持不响应中断，而Object不支持
            Lock能够支持多个Condition等待队列，而Object只能支持一个
            Condition能够支持设置超时时间的await，而Object不能
            2. 与Object的wait/notify相对应的方法
            针对Object的wait方法：await, awaitNanos,...
            针对Object的notify/notifyAll方法：signal，signalAll方法
            3. 底层数据结构
            复用AQS的Node类，由不带头结点的链表实现的队列
            4. awiat实现原理
            将调用await方法的线程封装成Node，尾插入到同步队列中，并通过LockSupport.park方法
            将当前线程置于WAITING状态，直至其他线程通过signal/signalAll方法将其移入到同步队列中，
            使其有机会在同步队列中通过自旋获取到Lock，从而当前线程才能从await方法处退出
            5. signal/signalAll实现原理
            将等待队列的队头结点移入到同步队列中
            6. await和signal/signalAll的结合使用
        7. ### ***LockSupport***
    - [Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)
    - [Java并发包基石-AQS详解](https://www.cnblogs.com/chengxiao/archive/2017/07/24/7141160.html)
- ## 并发容器
    1. ### concurrentHashMap
        1. 关键属性
            - table：元素为Node类的哈希桶数组
            - nextTable：扩容时的新数组
            - sizeCtl：控制数组的大小
            - Unsafe u：提供对哈希桶数组元素的CAS操作
        2. 重要内部类
            - Node：实现Map.entry接口，存放key，value
            - TreeNode：继承Node，会被封装成TreeBin
            - TreeBin：进一步封装TreeNode，链表过长时转换成红黑树时使用
            - ForwardingNode：扩容时出现的特殊结点
        3. 涉及到的CAS操作
            - tabAt：查询哈希桶数组的元素
            - casTabAt：设置哈希桶数组中索引为i的元素
            - setTabAt：设置哈希桶数组中索引为i的元素
        4. 构造方法
            - 数组长度总是会保证为2的幂次方
        5. put执行流程
            1. 如果当前数组还未初始化，先进行初始化
            2. spread方法重哈希（高16位和低16位异或操作），将哈希值与数组长度与运算，确定待插入结点的索引为i
            3. 当前哈希桶中i处为null，直接插入
            4. i处结点不为null的话并且结点hash>0，说明i处为链表头结点。遍历链表，遇到与key相同的结点则覆盖其value，
            如果遍历完没有找到，则尾插入新结点
            5. i处结点不为null的话并且结点状态为MOVED，则说明在扩容，帮助扩容
            6. i处结点不为null的话并且结点位TreeBin，则使用红黑树的方式插入结点
            7. 插入新结点后，检查链表长度是否大于8，若大于，则转换成红黑树
            8. 检测数组长度，若超过临界值，则扩容
        6. get执行流程
        7. 扩容机制
        8. 用于统计size的方法的执行流程
        9. 1.8版本的ConcurrentHashMap与之前版本的比较
            - 减小锁粒度 
            - 采用了synchronized而不是lock，大量使用CAS操作
    2. ### CopyOnWriteArrayList
        1. 实现原理
            - 利用了读写分离的思想；当写线程写入数据的时候会复制新建一个新容器，当数据更新完成后，再将旧容器引用指向新容器。读线程感知数据更新是延时的，也就是说COW是牺牲了数据实时性而保证数据最终一致性
            - 由于写线程写数据是在新容器写入的，因此读线程不会被阻塞
        2. COW和ReentrantReadWriteLock的区别
            - 相同点
                1. 两者都采用了读写分离的思想，并且读和
                读线程之间都不会被阻塞
            - 不同点
                - 当写线程在写数据时，ReadWriteLock会阻塞读线程，而
                由于COW采用了延时更新的策略，COW并不会阻塞读线程
                - ReadWriteLock保证了数据实时性而COW保证数据最终一致性
        3. 应用场景
            - 适用于读多写少的场景，比如系统的黑名单，白名单设置
        4. 为什么具有弱一致性
            - COW的实现是采用数组，而数组的引用是volatile修饰，但是数组的元素并不是
        volatile的。因此数据更新只有当volatile引用指向新数组时才会生效
        5. COW的缺点
            - 由于在写数据时，会复制，因此可能会出现内存使用瞬间增加，
            导致minor GC和major GC
            - 只具有数据最终一致性，对数据实时性要求高的场景不合适
    3. ### ThreadLocal
        1. 实现思想
        采用“空间换时间的”思想，每个线程拥有变量副本，达到
        隔离线程的目的，线程间不受影响解决线程安全的问题
        操作系统
        2. set方法原理
        数据存放在由当前线程Thread维护的ThreadLocalMap中，数据结构为
        当前ThreadLocal实例为key，值为value的键值对
        3. get方法原理
        以当前ThreadLocal为键，从当前线程Thread维护的ThreadLocalMap中获取value
        4. remove方法原理
        从当前线程Thread维护的ThreadLocalMap中删除以当前ThreadLocal实例为键的键值对
        5. ThreadLocalMap
            - 底层数据结构
                键为ThreadLocal实例，值为value的Entry数组
                数组大小为2的幂次方
                键ThreadLocal为弱引用
            - set方法原理
                1. 计算ThreadLocal的hashcode
                总是加上0x61c88647，这是“Fibonacci Hashing”
                2. 计算待插入的索引为i
                采用与运算
                3. 如何解决hash冲突
                    - 当索引为i处有Entry的话（hash冲突），就采用线性探测，进行环形搜素
                4. 加载因子
                    - ThreadLocalMap初始大小为16，加载因子为2/3
                5. 扩容resize
                    - 容量为原数组大小的两倍
                    - getEntry方法原理
                        - 根据ThreadLocal的hashcode进行定位，如果所定位的Entry的key与所查找的key相同则直接返回，否则，环形向后继续探测。
                    - remove原理
                        - 先找到对应的entry，然后让它的Key为null，之后再对其进行清理
                6. ThreadLocal内存泄漏
                    - 造成内存泄露的原因
                        - 由于ThreadLocal在Entry中是弱引用，当外部ThreadLocal实例被置为null后，根据可达性分析，堆中ThreadLocal不可达，会被GC掉，因此就存在key为null的entry。无法通过key为null去访问entry，因此，就会存在threadRef->currentThread->threadLocalMap->entry->valueRef->valueMemory引用链造成valueMemory不会被GC掉，造成内存泄漏
                    - 怎样来解决内存泄漏
                        - 关键方法cleanSomeSlots，expungeStaleEntry，replaceStaleEntry
                        - 在ThreadLocal的set，getEntry以及remove方法中都利用以上三个关键方法对潜在的内存泄漏进行处理
                    - 为什么要使用弱引用
                        - 如果使用强引用的话，即使显示对ThreadLocal的实例置为null的话，由于Thread，ThreadLocal以及
                        ThreadLocalMap引用链关系，ThreadLocal也不会被GC掉，反而会程序员带来困扰；
                        - 使用弱引用，尽管存在ThreadLocal内存泄漏的危险，但实际上已经对其进行了处理
                7. ThreadLocal的最佳实践
                    - 使用完ThreadLocal后要remove掉
                8. 应用场景
                    - hibernate管理seesion，每个线程维护其自身的session，彼此不干扰
                    - 用于解决对象不能被多个线程共享的问题
    4. ### BlockingQueue
        1. BlockingQueue的基本操作
        2. 常用的BlockingQueue
            - ArrayBlockingQueue：由数组实现的有界阻塞队列
            - LinkedBlockingQueue：由链表实现的有界阻塞队列，可指定长度，如果没有指定则为Integer.MAX_VALUE
            - PriorityBlockingQueue：支持优先级的无界阻塞队列
            - SynchronousQueue：不存储任何元素阻塞队列
            - LinkedTransferQueue：由链表实现的无界阻塞队列
            - LinkedBlockingDeque：由链表实现的无界阻塞队列
            - DelayQueue：存在实现了Delayed接口的数据的无界阻塞队列
        3. ArrayBlockingQueue与LinkedBlockingQueue实现原理
            - 会有notFull和notEmpty两个等待队列，分别存放
            被阻塞的插入数据线程以及被阻塞的消费数据的线程
            - ArrayBlockingQueue只有一个lock，而LinkedBlockingQueue有两个lock，因此LinkedBlockingQueue的并发度更高，吞吐量更大
            - 通过put和take方法了解生产者-消费者的正确写法
    5. ### ConcurrentLinkedQueue
        1. 实现原理
        主要采用CAS操作以保证线程安全，并且采用了延时更新的策略，提高吞吐量
        2. 数据结构
        由Node构成的链式队列
        3. 核心方法
        offer方法实现原理
        poll方法实现原理
        4. HOPS延迟更新的设计意图
        尽可能的减少CAS操作，以提升吞吐量和执行效率
- ## 线程池（Executor体系）
    1. ### ThreadPoolExecutor
        1. 为什么要使用线程池
            - 降低资源损耗
            - 提升系统响应速度
            - 提高线程的可管理性
        2. 执行流程
            - 核心线程corePool，阻塞队列workQueue以及最大线程池maxPool三级缓存的工作方式
        3. 构造器各个参数的意义
            - coolPoolSize：核心线程池的大小
            - maximumPoolSize：线程池最大容量
            - keepAliveTime：空闲线程可存活时间
            - unit：keepAliveTime的时间单位
            - workQueue：存放任务的阻塞队列
            - threadFactory：生产线程的工厂类
            - handler：饱和丢弃策略。共四种：
                - AbortPolicy，
                - CallerRunsPolicy，
                - DiscardPolicy，
                - DiscardOldestPolicy
        4. 如何关闭线程池
            - shutdown：正在执行任务的线程，将任务执行完。空闲线程以中断的方式关闭
            - shutdownNow：停止所有线程，包括正在执行任务的线程。返回未执行的任务列表
            - showdown将线程池状态设置为SHUTDOWN，而shutdownNow将线程池状态设置为STOP
            - isTerminated来检查线程池是否已经关闭
        5. 如何配置线程池
            - CPU密集型：Ncpu+1
            - IO密集型：2Ncpu
            - 任务按照IO密集型和CPU密集型进行拆分
    2. ### ScheduledThreadPoolExecutor
        1. UML（类结构）
        继承了ThreadPoolExecutor，并实现了ScheduledExecutorService
        2. 常用方法
            - 可 延时执行任务：schedule(.....)
            - 可周期执行任务：scheduledAtFixedRate(...)和scheduledWithFixedDelay
            - scheduledAtFixedRate和scheduledWithFixedDelay的区别：...AtFixedRate不要求任务结束了才开始统计延时时间，而....WithFixedDelay要求从任务结束开始统计延时时间
        3. ScheduledFutureTask
            - 可周期执行的异步任务，每一次执行完后会重新设置任务下一次执行的任务，
        并且会添加到阻塞队列中
        4. DelayedWorkQueue
            - 按优先级排序的有界阻塞队列，底层数据结构是堆
    3. ### FutureTask
        1. FutureTask的几种状态
            - 未启动（还未执行run方法），已启动(已执行run方法），已结束（正常结束，被取消，出现异常）
        2. get()
             - 未启动和已启动状态，get方法会阻塞当前线程直到异步任务执行结束
        3. cancel()
             - 未启动状态时，调用cancel方法后该异步任务永远不会再执行
             - 已启动状态，调用cancel方法后根据参数是否中断当前执行任务的线程
             - 已结束状态，调用cancel方法时会返回false
        4. 应用场景
             - 当一个线程需要等到另一个任务执行结束后才能继续进行时，可以使用futureTask
        5. 实现了Runnable接口
             - futureTask同样可以交由executor执行，获取直接调用run方法执行
- ## 原子操作类
    1. ### 实现原理
        - 借住与Unsafe类的CAS操作，达到并发安全的目的
    2. ### 原子更新基本类型
        - AtomicInteger， AtomicLong，AtomicBoolean
    3. ### 原子更新数组类型
        - AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray
    4. ### 原子更新引用类型
        - AtomicReference，AtomicReferenceFieldUpdater，AtomicMarkableReference
    5. ### 原子更新字段类型
        - AtomicIntegerFieldUpdater，AtomicLongUpdater，AtomicStampedReference
- ## 并发工具
    1. ### 倒计时器CountDownLatch
        - 当CountDownLatch维护的计数器减至为0的时候，调用await方法的线程才会继续往下执行，否则会阻塞等待
        - 适用于一个线程需要等待其他多个线程执行结果的应用场景操作系统
    2. ### 循环栅栏CyclicBarrier
        -当一组线程都达到了“临界点”时，所有的线程才能继续往前执行，否则阻塞等待
    3. ### CountDownLatch和CyclicBarrier的比较
        1. CyclicBarrier能够复用，而CountDownLatch维护的倒计数器不能复用
        2. CyclicBarrier会在await处阻塞等待，而CountDownLatch在await出不会阻塞等待
        3. CyclicBarrier提供了例如isBroken，getNumerWaiting等方法能够
        查询当前状态，而CountDownLatch提供的方法较少
    4. ### 资源访问控制Semaphore
        - 适用于对特定资源需要控制能够并发访问资源的线程个数。需要先执行acquire方法获取许可证，如果获取成功后线程才能往下继续执行，否则只能阻塞等待；使用完后需要
        - 用release方法归还许可
    5. ### 数据交换Exchanger
        - 为两个线程提供了一个同步点，当两个线程都达到了同步点之后就可以使用exchange方法，互相交换数据；如果一个线程先达到了同步点，会在同步点阻塞等待直到另外一个线程也到达同步点
- ## 并发实践
    - ### 生产者-消费者问题
        1. 使用Object的wait/notifyAll方式实现
            - 使用Object的消息通知机制可能存在的问题
                - notify过早，wait线程无法再获取到通知以至于一直阻塞等待。解决办法：添加状态标志
                - wait条件变化。解决方法：使用while进行wait条件的判断，而不是在if中进行判断
                - “假死”状态：使用notifyAll而不是notify
            - 标准范式
                - 永远在while中对wait条件进行判断，而不是在if中进行判断
                - 使用notifyAll进行通知，而不要使用notify进行通知
        2. 使用lock的condition的await/signalAll方式实现
        3. 使用blockingQueue方式实现
            - 由于BlockingQueue有可阻塞的插入和删除数据的put和take方法，因此，在实现上比使用Object和lock的方式更加简洁
- ## 相关阅读
    - [tobebetterjavaer并发编程](https://tobebetterjavaer.com/thread/wangzhe-thread.html)
    - [Java并发专题](https://github.com/CL0610/Java-concurrency)


--- 

<仅供个人收藏学习>
