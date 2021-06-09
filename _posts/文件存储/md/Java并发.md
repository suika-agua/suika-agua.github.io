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

## 第 3 部分

*XMind - Evaluation Version*