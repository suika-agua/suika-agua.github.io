# Java并发

## 线程池

### 事先把多个对象放到一个容器中，使用时直接去池中拿进程。节省开辟进程时间，提高代码执行效率。

### 共四种

- 可缓存线程池

	- newCachedThreadPool
	- 线程数无限制，最大支持Integer.MAX_VALUE
	- 有空闲线程复用空闲线程，无空闲线程新建线程
	- 一定程度减少频繁创建/销毁进程，减少系统开销

- 定长线程池

	- newFixedThreadPool
	- 可控制线程最大并发数，超出线程在队列中等待

- 单线程化线程池

	- newSingleThreadPool
	- 有且仅有一个工作线程执行任务
	- 所有任务按照指定顺序执行，按照队列规则

- 周期性线程池

	- newScheduledThreadPool
	- 支持定时以指定周期循环执行任务

### 线程池原理

- 由阻塞队列和hashset构成
- 核心线程->阻塞队列->非核心线程 ->handler拒绝提交
- 线程复用*

	- 子主题 1

- 信号量

	- 用于保证两个或多个关键代码段不被并发调用

### 工作队列

- ArrayBlockingQueue：基于数组结构的有界阻塞队列，FIFO
- LinkedBlockingQueue：基于链表结构阻塞队列，FIFO，吞吐量高于ArrayBlockingQueue（newFixedThreadPool&newSingleThreadExecutor）
- SynchronousQueue：不存储元素的阻塞队列，插入操作必须等另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量高于2（newCachedThreadPool）
- PriorityBlockingQueue：一个具有优先级的无限阻塞队列

### 无界队列和有界队列

- 无界队列相较有界队列，任务和处理速度差异很大时，队列会保持快速增长，直到耗尽内存

### 多线程中的安全队列一般通过什么实现

- 阻塞队列，eg：BlockingQueue
- 非阻塞队列，eg：ConcurrentLinkedQueue
- BlockingQueue实现阻塞功能，调用put(e) take()方法*，ConcurrentLinkedQueue是基于链接节点、无界的、线程安全的非阻塞队列

### 线程池实际调用*

## Synchronized、volatile、Lock相关

### Synchronized&Lock

- synchronized是Java关键字，在jvm层面上；lock是一个接口
- synchronized会自动释放锁；lock需手动释放（写于try-catch的finally中）
- synchronized无法中断等待锁；lock可以中断
- lock可以提高多个线程行进行读/写操作的效率
- 竞争资源激烈时，lock的性能明显优于synchronize

### synchronize锁的机制

- 自旋锁

	- 占着cpu不放，等待获取锁机会（时间长会影响整体性能，时间短达不到延迟阻塞的目的）

- 偏向锁

	- 一旦第一次获得监视对象，之后让监视对象偏向这个线程。之后可避免CAF操作？--放置变量，若为true则无需走加锁解锁流程

- 轻量级锁

	- 偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁竞争用的时候，偏见->轻量级

- 重量级锁

	- 又叫对象监视器，除了具备Mutex互斥的功能，还实现semaphore（信号量）功能。

- 详情图？

	- 没看懂

### 类锁&方法锁

- synchronized修饰静态方法获取类锁；synchronized修饰普通方法或代码块获取对象锁
- 两者不冲突

### 重入锁&非重入锁

- 重入锁：又称递归锁，以线程为单位，当一个线程获取对象锁后，该线程可以再获取该对象上的锁
- 非重入锁：不可发生递归调用

### wait、sleep的区别和notify运行过程

- wait&sleep

	- sleep是Thread的静态方法，可以在任何地方调用
	- wait是Object的成员方法，只能在Synchronized代码块中调用
	- sleep不会释放共享资源锁，wait会释放共享资源锁——sleep用于暂停执行，wait用于线程交互
	- 调用wait后，需要别的线程执行notify才能重新获取cpu

- notify

	- 线程A调用wait()后，线程A让出锁，加入锁对象等待队列；线程B获取锁后，调用notify通知锁对象等待队列，线程A从等待队列进入阻塞队列。直至线程B释放锁后，线程A竞争到锁后继续执行

### volatile原理

- 特征

	- 只用于修饰变量，试用修饰可能被多线程同时访问的变量
	- 保证有序性（禁止指令重排序）、可见性；相当于轻量级Synchronized
	- 变量位于主内存中，而变量在线程的工作内存中有拷贝，线程直接操作的是拷贝文件
	- 被修饰变量改变后会立刻同步到主内存，保持变量的可见性

- 双重检查单例中的应用

	- volatile可用于禁止指令重排序

- 意义

	- 防止cpu指令重排序

		- 指令重排序会在被volatile修饰的变量赋值操作前添加一个内存屏障，指令重排序时不能把后面指令移到内存屏障之前位置

	- 保证可见性

		- 被volatile修饰的变量在工作内存修改后会被强制写会主内存，其他线程在使用时也会强制从主内存刷新

	- 多线程安全

		- 必须要考虑操作是否具有原子性
		- 不需加锁保证原子性情况

			- 要么结果不依赖当前值，要么操作是原子性的
			- 变量不需要与其他状态变量共同参与不变约束（开关变量）

### synchronized和volatile关键字作用和区别

- volatile本质为jvm告诉当前变量在寄存器（工作内存）中的值是不确定的，需要从主内存中读取；Synchronized则是锁定当前线程可以访问该变量，其他线程被阻塞住
- 区别

	- volatile-变量；synchronized-变量、方法、类
	- volatile-仅实现变量修改可见性，不保证原子性；synchronized-变量修改可见性和原子性
	- volatile不会造成任何线程的阻塞
	- volatile标记的变量不会被编译器优化（指把变量读取到寄存器中）

### ReentrantLock的内部实现*

- 处理逻辑 
- NonFairSync(非公平可重入锁)
- FairSync(公平可重入锁)

### ReentranLock、synchronized、volatile 比较*

## 其他

### 多线程使用场景

- 为不阻塞主线程：耗时操作不在UI线程做
- 为不必等待，可同时处理多条数据：主线程监听用户请求，子线程处理用户请求，以获得大吞吐量。
- 不定期处理优先级低的服务（JVM垃圾回收）
- 为耗时但不消耗CPU操作时间的任务开启进程，提升工作效率（拆分磁盘IO为读取数据与处理数据两个进程，利用cpu等待磁盘IO时间）

### CopyOnWriteArrayList

- copy-on-write

	- 对一块内存进行修改时，现将内存拷贝一份，写完后再将原来指向内存指针指向新内存，回收就内存

- CopyOnWriteArrayList运用copy-on-write原理，修改数据时先copy一个容器，修改好时将新容器引用地址赋给旧容器。但其他线程读取该数组是，依旧是读取旧数据
- 优点

	- 可保证并发数据一致性
	- 解决了集合所线程遍历迭代问题

- 缺点

	- 内存占用大
	- 不能保证数据的实时一致性

- 使用场景

	- 读多写少（白名单，黑名单，商品类目访问更新）
	- 集合不大
	- 实时性要求不高

### ConcurrentHashMap

- 解決了访问HashTable的线程都必须竞争同一把锁的问题。将数据氛围一段一段的存储，每一段数据一把锁，当多线程访问容器里不同数据段数据时现成不存在锁竞争。
- concurrentHashMap（并发数，segment数），可在初始化时设置为其他值，但不可扩容。
- ensureSegment(初始化槽)：初始化时会初始化第一个槽，其他槽在插入第一个值时初始化
- Java8有做改动

### 线程死锁的4个条件

- 互斥条件：一个资源每次只能被一个线程使用
- 请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放
- 不剥夺条件：线程已获得的资源，在未使用完之前，不能进行剥夺
- 循环等待条件：若干线程之间形成一种头尾相接的循环等待关系

### CAS介绍*

- 子主题 1

### 进程和线程之间的区别

- *一个程序至少有一个进程，一个进程至少有一个线程
- 划分尺度：线程划分尺度小于进程，使得多线程程序并发性高
- 内存空间：进程在执行过程中拥有独立的内存单元；一个进程中多个线程共享内存
- 执行过程：每个独立的进程有一个程序运行的入口、顺序执行序列和程序的出口；线程不能够独立执行，必须以存在应用程序中，由程序进行控制
- 逻辑角度：多线程意义在于一个应用程序中，多个执行部分可以同时执行；但未被看作是独立的应用来实现（类似）进程的调度管理以及资源分配
- 进程是资源分配调度的一个独立单位，线程是进程的一个实体，是cpu调度的基本单位。
- 进程与线程关系/线程与线程关系：一个线程可以创建和撤销另一个线程；同一个进程中的多个线程之间可以并发执行
- 进程有独立地址空间，一个进程崩溃后，保护模式下不会对其他进程产生影响，而线程只是一个进程中的不同执行路径。线程没有单独地址空间，一个线程死掉等于整个进程死掉。则多进程程序强壮于多线程程序
- 进程切换耗费资源大
- 待整理

### 什么导致线程阻塞

- 阻塞：暂停一个线程的执行以等待某个条件发生
- sleep()方法：用于等待某个资源就绪的情形，测试发现条件不满足后，让线程阻塞
- suspend()和resume()：suspend()阻塞线程 resume()恢复
- yield()：使线程放弃当前分得的cpu时间
- wait()和notify()：wait阻塞notify恢复

	- 与其他方法区别：wait阻塞时要释放所有占用着的锁，调用notify时随机释放一个由于调用wait方法阻塞的线程

- wait、notify、notifyAll

	- 锁池：执行对象synchronized方法会进入对象锁池
	- 等待池：某线程调用wait方法，线程释放该对象的锁，等待线程不会竞争该对象的锁
	- notify随机唤醒等待池中一个线程，notifyall唤醒等待池中所有线程，均进入锁池

### 线程的生命周期

- new->runnable->waiting->time_waiting->blocked->terminated//画图解释

### 乐观锁与悲观锁

- 悲观锁：线程得到锁，其他线程挂起等待，适用写入频繁（synchronized）

	- 假设拿数据时别人会修改

- 乐观锁：假设没有冲突，不加锁，更新判断是否过期，不过期不更新，适用读取频繁

	- 假设拿数据时别人不会修改，不上锁，版本号机制和CSA算法判断是否过期

### run和start区别

### 多线程断点续传（

### 怎样安全停止一个线程任务

- 终止线程

	- 使用violate boolean变量推出标志，使线程正常退出
	- interrupt方法中断线程（线程不一定中止）
	- stop方法强行中止线程，不安全的原因：调用后创建子线程的线程会跑出ThreadDeathError错误，会释放子线程所有锁

- 终止线程池

	- shutdown关闭线程池：线程池不会立刻退出，知道添加到线程池中的任务都处理完成才会推出
	- shutdownnow关闭线程池并中断任务：试图停止所有进程，但线程中有sleep wait condition 定时锁等应用则无法中断线程，则仍要等正在执行任务执行完成才能推出

### 栈如何转换为堆

- 堆内存用于创建对象和数组，由GC垃圾回收机制来管理
- 堆用于存放对象，栈用于执行程序
- 栈中变量指向堆中变量，即Java指针

### 多线程特性

- 原子性、可见性、有序性 
- happens-before原则（

*XMind - Evaluation Version*