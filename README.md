# Concurrent
多线程并发知识点总结
JAVA多线程并发

1.1.1. JAVA 并发知识库 



1.1.2. JAVA 线程实现/创建方式 
1.1.2.1. 继承 Thread 类 
Thread 类本质上是实现了 Runnable 接口的一个实例，代表一个线程的实例。启动线程的唯一方法就是通过 Thread 类的 start()实例方法。start()方法是一个 native 方法，它将启动一个新线程，并执行 run()方法。 
public class MyThread extends Thread { 
public void run() { 
System.out.println("MyThread.run()"); 
} 
} 
MyThread myThread1 = new MyThread(); 
myThread1.start(); 
1.1.2.2. 实现 Runnable 接口。 
如果自己的类已经 extends 另一个类，就无法直接 extends Thread，此时，可以实现一个 Runnable 接口。 
public class MyThread extends OtherClass implements Runnable { 
public void run() { 
System.out.println("MyThread.run()"); 
} 
} 
//启动 MyThread，需要首先实例化一个 Thread，并传入自己的 MyThread 实例： 
MyThread myThread = new MyThread(); 
Thread thread = new Thread(myThread); 
thread.start(); 
//事实上，当传入一个 Runnable target 参数给 Thread 后，Thread 的 run()方法就会调用 
target.run() 
public void run() { 
if (target != null) { 
target.run(); 
} 
} 
1.1.2.3. ExecutorService、Callable<Class>、Future 有返回值线程 
有返回值的任务必须实现 Callable 接口，类似的，无返回值的任务必须 Runnable 接口。执行 Callable 任务后，可以获取一个 Future 的对象，在该对象上调用 get 就可以获取到 Callable 任务 返回的 Object 了，再结合线程池接口 ExecutorService 就可以实现传说中有返回结果的多线程 了。 
//创建一个线程池 
ExecutorService pool = Executors.newFixedThreadPool(taskSize); 
// 创建多个有返回值的任务 
List<Future> list = new ArrayList<Future>(); 
for (int i = 0; i < taskSize; i++) { 
Callable c = new MyCallable(i + " "); 
// 执行任务并获取 Future 对象 
Future f = pool.submit(c); 
list.add(f); 
} 
// 关闭线程池 
pool.shutdown(); 
// 获取所有并发任务的运行结果 
for (Future f : list) { 
// 从 Future 对象上获取任务的返回值，并输出到控制台 
System.out.println("res：" + f.get().toString()); 
} 
1.1.2.4. 基于线程池的方式 
线程和数据库连接这些资源都是非常宝贵的资源。那么每次需要的时候创建，不需要的时候销毁，是非常浪费资源的。那么我们就可以使用缓存的策略，也就是使用线程池。 
// 创建线程池 
ExecutorService threadPool = Executors.newFixedThreadPool(10); 
while(true) { 
threadPool.execute(new Runnable() { // 提交多个线程任务，并执行 
@Override 
public void run() { 
System.out.println(Thread.currentThread().getName() + " is running .."); 
try { 
Thread.sleep(3000); 
} catch (InterruptedException e) { 
e.printStackTrace(); 
} 
} 
}); 
} 
} 


1.1.3. 4 种线程池 
Java 里面线程池的顶级接口是 Executor，但是严格意义上讲 Executor 并不是一个线程池，而 只是一个执行线程的工具。真正的线程池接口是 ExecutorService。
1.1.3.1. newCachedThreadPool 
创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。对于执行很多短期异步任务的程序而言，这些线程池通常可提高程序性能。调用 execute 将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。因此，长时间保持空闲的线程池不会使用任何资源。 
1.1.3.2. newFixedThreadPool 
创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。在任意点，在大多数 nThreads 线程会处于处理任务的活动状态。如果在所有线程处于活动状态时提交附加任务， 则在有可用线程之前，附加任务将在队列中等待。如果在关闭前的执行期间由于失败而导致任何线程终止，那么一个新线程将代替它执行后续的任务（如果需要）。在某个线程被显式地关闭之前，池中的线程将一直存在。
1.1.3.3. newScheduledThreadPool 
创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。 
ScheduledExecutorService scheduledThreadPool= Executors.newScheduledThreadPool(3); 
scheduledThreadPool.schedule(newRunnable(){ 
@Override 
public void run() { 
System.out.println("延迟三秒"); 
} 
}, 3, TimeUnit.SECONDS); 
scheduledThreadPool.scheduleAtFixedRate(newRunnable(){ 
@Override 
public void run() { 
System.out.println("延迟 1 秒后每三秒执行一次"); 
} 
},1,3,TimeUnit.SECONDS); 
1.1.3.4. newSingleThreadExecutor 
Executors.newSingleThreadExecutor()返回一个线程池（这个线程池只有一个线程）,这个线程池可以在线程死后（或发生异常时）重新启动一个线程来替代原来的线程继续执行下去！


1.1.4. 线程生命周期(状态) 
当线程被创建并启动以后，它既不是一启动就进入了执行状态，也不是一直处于执行状态。 
在线程的生命周期中，它要经过新建(New)、就绪（Runnable）、运行（Running）、阻塞 (Blocked)和死亡(Dead)5 种状态。尤其是当线程启动以后，它不可能一直"霸占"着 CPU 独自运行，所以 CPU 需要在多条线程之间切换，于是线程状态也会多次在运行、阻塞之间切换 
1.1.4.1. 新建状态（NEW） 
当程序使用 new 关键字创建了一个线程之后，该线程就处于新建状态，此时仅由 JVM 为其分配内存，并初始化其成员变量的值
1.1.4.2. 就绪状态（RUNNABLE）： 
当线程对象调用了 start()方法之后，该线程处于就绪状态。Java 虚拟机会为其创建方法调用栈和程序计数器，等待调度运行。 
1.1.4.3. 运行状态（RUNNING）： 
如果处于就绪状态的线程获得了 CPU，开始执行 run()方法的线程执行体，则该线程处于运行状态。 
1.1.4.4. 阻塞状态（BLOCKED）： 
阻塞状态是指线程因为某种原因放弃了 cpu 使用权，也即让出了 cpu timeslice，暂时停止运行。直到线程进入可运行(runnable)状态，才有机会再次获得 cpu timeslice 转到运行(running)状态。阻塞的情况分三种： 
等待阻塞（o.wait->等待对列）： 
运行(running)的线程执行 o.wait()方法，JVM 会把该线程放入等待队列(waitting queue) 
中。 
同步阻塞(lock->锁池) 
运行(running)的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则 JVM 会把该线程放入锁池(lock pool)中。 
其他阻塞(sleep/join) 
运行(running)的线程执行 Thread.sleep(long ms)或 t.join()方法，或者发出了 I/O 请求时， JVM 会把该线程置为阻塞状态。当 sleep()状态超时、join()等待线程终止或者超时、或者 I/O处理完毕时，线程重新转入可运行(runnable)状态。 
1.1.4.5. 线程死亡（DEAD） 
线程会以下面三种方式结束，结束后就是死亡状态。 
正常结束 
1. run()或 call()方法执行完成，线程正常结束。 
异常结束 
2. 线程抛出一个未捕获的 Exception 或 Error。 
调用 stop 
3. 直接调用该线程的 stop()方法来结束该线程—该方法通常容易导致死锁，不推荐使用。



1.1.5. 终止线程 4 种方式 
1.1.5.1. 正常运行结束 
程序运行结束，线程自动结束。 
1.1.5.2. 使用退出标志退出线程 
一般 run()方法执行完，线程就会正常结束，然而，常常有些线程是伺服线程。它们需要长时间的运行，只有在外部某些条件满足的情况下，才能关闭这些线程。使用一个变量来控制循环，例如： 
最直接的方法就是设一个 boolean 类型的标志，并通过设置这个标志为 true 或 false 来控制 while循环是否退出，代码示例： 
public class ThreadSafe extends Thread { 
public volatile boolean exit = false; 
public void run() { 
while (!exit){ 
//do something 
} 
} 
} 
定义了一个退出标志 exit，当 exit 为 true 时，while 循环退出，exit 的默认值为 false.在定义 exit 时，使用了一个 Java 关键字 volatile，这个关键字的目的是使 exit 同步，也就是说在同一时刻只能由一个线程来修改 exit 的值。 
1.1.5.3. Interrupt 方法结束线程 
使用 interrupt()方法来中断线程有两种情况：
1. 线程处于阻塞状态：如使用了 sleep,同步锁的 wait,socket 中的 receiver,accept 等方法时会使线程处于阻塞状态。当调用线程的 interrupt()方法时，会抛出InterruptException 异常。 阻塞中的那个方法抛出这个异常，通过代码捕获该异常，然后 break 跳出循环状态，从而让我们有机会结束这个线程的执行。通常很多人认为只要调用 interrupt 方法线程就会结束，实际上是错的， 一定要先捕获 InterruptedException 异常之后通过 break 来跳出循环，才能正常结束 run 方法。 
2. 线程未处于阻塞状态：使用 isInterrupted()判断线程的中断标志来退出循环。当使用 
interrupt()方法时，中断标志就会置 true，和使用自定义的标志来控制循环是一样的道理。 
public class ThreadSafe extends Thread { 
public void run() { 
while (!isInterrupted()){ //非阻塞过程中通过判断中断标志来退出 
try{ 
Thread.sleep(5*1000);//阻塞过程捕获中断异常来退出 
}catch(InterruptedException e){ 
e.printStackTrace(); 
break;//捕获到异常之后，执行 break 跳出循环 
} 
} 
} 
} 
1.1.5.4. stop 方法终止线程（线程不安全） 
程序中可以直接使用 thread.stop()来强行终止线程，但是 stop 方法是很危险的，就象突然关闭计算机电源，而不是按正常程序关机一样，可能会产生不可预料的结果，不安全主要是： 
thread.stop()调用之后，创建子线程的线程就会抛出 ThreadDeatherror 的错误，并且会释放子线程所持有的所有锁。一般任何进行加锁的代码块，都是为了保护数据的一致性，如果在调用thread.stop()后导致了该线程所持有的所有锁的突然释放(不可控制)，那么被保护数据就有可能呈现不一致性，其他线程在使用这些被破坏的数据时，有可能导致一些很奇怪的应用程序错误。因此，并不推荐使用 stop 方法来终止线程。 

1.1.6. sleep 与 wait 区别 
1. 对于 sleep()方法，我们首先要知道该方法是属于 Thread 类中的。而 wait()方法，则是属于Object 类中的。
2. sleep()方法导致了程序暂停执行指定的时间，让出 cpu 该其他线程，但是他的监控状态依然保持者，当指定的时间到了又会自动恢复运行状态。 
3. 在调用 sleep()方法的过程中，线程不会释放对象锁。 
4. 而当调用 wait()方法的时候，线程会放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象调用 notify()方法后本线程才进入对象锁定池准备获取对象锁进入运行状态。 
1.1.7. start 与 run 区别 
1. start（）方法来启动线程，真正实现了多线程运行。这时无需等待 run 方法体代码执行完毕，可以直接继续执行下面的代码。 
2. 通过调用 Thread 类的 start()方法来启动一个线程， 这时此线程是处于就绪状态， 并没有运行。 
3. 方法 run()称为线程体，它包含了要执行的这个线程的内容，线程就进入了运行状态，开始运行 run 函数当中的代码。 Run 方法运行结束， 此线程终止。然后 CPU 再调度其它线程。 

1.1.8. JAVA 后台线程 
1. 定义：守护线程--也称“服务线程”，他是后台线程，它有一个特性，即为用户线程 提供公共服务，在没有用户线程可服务时会自动离开。 
2. 优先级：守护线程的优先级比较低，用于为系统中的其它对象和线程提供服务。 
3. 设置：通过 setDaemon(true)来设置线程为“守护线程”；将一个用户线程设置为守护线程的方式是在 线程对象创建 之前 用线程对象的 setDaemon 方法。 
4. 在 Daemon 线程中产生的新线程也是 Daemon 的。 
5. 线程则是 JVM 级别的，以 Tomcat 为例，如果你在 Web 应用中启动一个线程，这个线程的生命周期并不会和 Web 应用程序保持同步。也就是说，即使你停止了 Web 应用，这个线程依旧是活跃的。 
6. example: 垃圾回收线程就是一个经典的守护线程，当我们的程序中不再有任何运行的Thread,程序就不会再产生垃圾，垃圾回收器也就无事可做，所以当垃圾回收线程是 JVM 上仅剩的线程时，垃圾回收线程会自动离开。它始终在低级别的状态中运行，用于实时监控和管理系统中的可回收资源。 
7. 生命周期：守护进程（Daemon）是运行在后台的一种特殊进程。它独立于控制终端并且周期性地执行某种任务或等待处理某些发生的事件。也就是说守护线程不依赖于终端，但是依赖于系统，与系统“同生共死”。当 JVM 中所有的线程都是守护线程的时候，JVM 就可以退出了；如果还有一个或以上的非守护线程则 JVM 不会退出。

1.1.9. JAVA 锁 
1.1.9.1. 乐观锁 
乐观锁是一种乐观思想，即认为读多写少，遇到并发写的可能性低，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，采取在写时先读出当前版本号，然后加锁操作（比较跟上一次的版本号，如果一样则更新），如果失败则要重复读-比较-写的操作。 java 中的乐观锁基本都是通过 CAS 操作实现的，CAS 是一种更新的原子操作，比较当前值跟传入值是否一样，一样则更新，否则失败。 
1.1.9.2. 悲观锁 
悲观锁是就是悲观思想，即认为写多，遇到并发写的可能性高，每次去拿数据的时候都认为别人会修改，所以每次在读写数据的时候都会上锁，这样别人想读写这个数据就会 block 直到拿到锁。 java中的悲观锁就是Synchronized,AQS框架下的锁则是先尝试cas乐观锁去获取锁，获取不到，才会转换为悲观锁，如 RetreenLock。 
1.1.9.3. 自旋锁 
自旋锁原理非常简单，如果持有锁的线程能在很短时间内释放锁资源，那么那些等待竞争锁 
的线程就不需要做内核态和用户态之间的切换进入阻塞挂起状态，它们只需要等一等（自旋），等持有锁的线程释放锁后即可立即获取锁，这样就避免用户线程和内核的切换的消耗。 
线程自旋是需要消耗 cup 的，说白了就是让 cup 在做无用功，如果一直获取不到锁，那线程 也不能一直占用 cup 自旋做无用功，所以需要设定一个自旋等待的最大时间。 
如果持有锁的线程执行的时间超过自旋等待的最大时间扔没有释放锁，就会导致其它争用锁 
的线程在最大等待时间内还是获取不到锁，这时争用线程会停止自旋进入阻塞状态。 
自旋锁的优缺点 
自旋锁尽可能的减少线程的阻塞，这对于锁的竞争不激烈，且占用锁时间非常短的代码块来 
说性能能大幅度的提升，因为自旋的消耗会小于线程阻塞挂起再唤醒的操作的消耗，这些操作会导致线程发生两次上下文切换！ 
但是如果锁的竞争激烈，或者持有锁的线程需要长时间占用锁执行同步块，这时候就不适合 
使用自旋锁了，因为自旋锁在获取锁前一直都是占用 cpu 做无用功，占着 XX 不 XX，同时有大量 线程在竞争一个锁，会导致获取锁的时间很长，线程自旋的消耗大于线程阻塞挂起操作的消耗，其它需要 cup 的线程又不能获取到 cpu，造成 cpu 的浪费。所以这种情况下我们要关闭自旋锁； 
自旋锁时间阈值（1.6 引入了适应性自旋锁） 
自旋锁的目的是为了占着 CPU 的资源不释放，等到获取到锁立即进行处理。但是如何去选择自旋的执行时间呢？如果自旋执行时间太长，会有大量的线程处于自旋状态占用 CPU 资源，进而会影响整体系统的性能。因此自旋的周期选的额外重要！JVM 对于自旋周期的选择，jdk1.5 这个限度是一定的写死的，在 1.6 引入了适应性自旋锁，适应 性自旋锁意味着自旋的时间不在是固定的了，而是由前一次在同一个锁上的自旋时间以及锁的拥有者的状态来决定，基本认为一个线程上下文切换的时间是最佳的一个时间，同时 JVM 还针对当 
前 CPU 的负荷情况做了较多的优化，如果平均负载小于 CPUs 则一直自旋，如果有超过(CPUs/2)个线程正在自旋，则后来线程直接阻塞，如果正在自旋的线程发现 Owner 发生了变化则延迟自旋时间（自旋计数）或进入阻塞，如果 CPU 处于节电模式则停止自旋，自旋时间的最坏情况是 CPU的存储延迟（CPU A 存储了一个数据，到 CPU B 得知这个数据直接的时间差），自旋时会适当放弃线程优先级之间的差异。 
自旋锁的开启 
JDK1.6 中-XX:+UseSpinning 开启； 
-XX:PreBlockSpin=10 为自旋次数； 
JDK1.7 后，去掉此参数，由 jvm 控制； 

4.1.9.4. Synchronized 同步锁 
synchronized 它可以把任意一个非 NULL 的对象当作锁。他属于独占式的悲观锁，同时属于可重入锁。 
Synchronized 作用范围 
1. 作用于方法时，锁住的是对象的实例(this)； 
2. 当作用于静态方法时，锁住的是Class实例，又因为Class的相关数据存储在永久带PermGen（jdk1.8 则是 metaspace），永久带是全局共享的，因此静态方法锁相当于类的一个全局锁，会锁所有调用该方法的线程； 
3. synchronized 作用于一个对象实例时，锁住的是所有以该对象为锁的代码块。它有多个队列，当多个线程一起访问某个对象监视器的时候，对象监视器会将这些线程存储在不同的容器中。 
Synchronized 核心组件 
1) Wait Set：哪些调用 wait 方法被阻塞的线程被放置在这里； 
2) Contention List：竞争队列，所有请求锁的线程首先被放在这个竞争队列中； 
3) Entry List：Contention List 中那些有资格成为候选资源的线程被移动到 Entry List 中； 
4) OnDeck：任意时刻，最多只有一个线程正在竞争锁资源，该线程被成为 OnDeck； 
5) Owner：当前已经获取到所资源的线程被称为 Owner； 
6) !Owner：当前释放锁的线程。 
Synchronized 实现

1. JVM 每次从队列的尾部取出一个数据用于锁竞争候选者（OnDeck），但是并发情况下， 
ContentionList 会被大量的并发线程进行 CAS 访问，为了降低对尾部元素的竞争，JVM 会将一部分线程移动到 EntryList 中作为候选竞争线程。 
2. Owner 线程会在 unlock 时，将 ContentionList 中的部分线程迁移到 EntryList 中，并指定EntryList 中的某个线程为 OnDeck 线程（一般是最先进去的那个线程）。 
3. Owner 线程并不直接把锁传递给 OnDeck 线程，而是把锁竞争的权利交给 OnDeck， 
OnDeck 需要重新竞争锁。这样虽然牺牲了一些公平性，但是能极大的提升系统的吞吐量，在JVM 中，也把这种选择行为称之为“竞争切换”。 
4. OnDeck 线程获取到锁资源后会变为 Owner 线程，而没有得到锁资源的仍然停留在 EntryList中。如果 Owner 线程被 wait 方法阻塞，则转移到 WaitSet 队列中，直到某个时刻通过 notify或者 notifyAll 唤醒，会重新进去 EntryList 中。 
5. 处于 ContentionList、EntryList、WaitSet 中的线程都处于阻塞状态，该阻塞是由操作系统来完成的（Linux 内核下采用 pthread_mutex_lock 内核函数实现的）。 
6. Synchronized 是非公平锁。 Synchronized 在线程进入 ContentionList 时，等待的线程会先尝试自旋获取锁，如果获取不到就进入 ContentionList，这明显对于已经进入队列的线程是不公平的，还有一个不公平的事情就是自旋获取锁的线程还可能直接抢占 OnDeck 线程的锁资源。 
7. 每个对象都有个 monitor 对象，加锁就是在竞争 monitor 对象，代码块加锁是在前后分别加上 monitorenter 和 monitorexit 指令来实现的，方法加锁是通过一个标记位来判断的 
8. synchronized 是一个重量级操作，需要调用操作系统相关接口，性能是低效的，有可能给线程加锁消耗的时间比有用操作消耗的时间更多。 
9. Java1.6，synchronized 进行了很多的优化，有适应自旋、锁消除、锁粗化、轻量级锁及偏向锁等，效率有了本质上的提高。在之后推出的 Java1.7 与 1.8 中，均对该关键字的实现机理做了优化。引入了偏向锁和轻量级锁。都是在对象头中有标记位，不需要经过操作系统加锁。 
10. 锁可以从偏向锁升级到轻量级锁，再升级到重量级锁。这种升级过程叫做锁膨胀； 
11. JDK 1.6 中默认是开启偏向锁和轻量级锁，可以通过-XX:-UseBiasedLocking 来禁用偏向锁。

1.1.9.5. ReentrantLock 
ReentantLock 继承接口 Lock 并实现了接口中定义的方法，他是一种可重入锁，除了能完成 synchronized 所能完成的所有工作外，还提供了诸如可响应中断锁、可轮询锁请求、定时锁等避免多线程死锁的方法。 
Lock 接口的主要方法 
1. void lock(): 执行此方法时, 如果锁处于空闲状态, 当前线程将获取到锁. 相反, 如果锁已经被其他线程持有, 将禁用当前线程, 直到当前线程获取到锁. 
2. boolean tryLock()：如果锁可用, 则获取锁, 并立即返回 true, 否则返回 false. 该方法和 lock()的区别在于, tryLock()只是"试图"获取锁, 如果锁不可用, 不会导致当前线程被禁用,当前线程仍然继续往下执行代码. 而 lock()方法则是一定要获取到锁, 如果锁不可用, 就一直等待, 在未获得锁之前,当前线程并不继续向下执行. 
3. void unlock()：执行此方法时, 当前线程将释放持有的锁. 锁只能由持有者释放, 如果线程并不持有锁, 却执行该方法, 可能导致异常的发生. 
4. Condition newCondition()：条件对象，获取等待通知组件。该组件和当前的锁绑定， 
当前线程只有获取了锁，才能调用该组件的 await()方法，而调用后，当前线程将缩放锁。 
5. getHoldCount() ：查询当前线程保持此锁的次数，也就是执行此线程执行 lock 方法的次数。 
6. getQueueLength（）：返回正等待获取此锁的线程估计数，比如启动 10 个线程，1 个 
线程获得锁，此时返回的是 9 
7. getWaitQueueLength：（Condition condition）返回等待与此锁相关的给定条件的线 
程估计数。比如 10 个线程，用同一个 condition 对象，并且此时这 10 个线程都执行了 
condition 对象的 await 方法，那么此时执行此方法返回 10 
8. hasWaiters(Condition condition)：查询是否有线程等待与此锁有关的给定条件 
(condition)，对于指定 contidion 对象，有多少线程执行了 condition.await 方法 
9. hasQueuedThread(Thread thread)：查询给定线程是否等待获取此锁 
10. hasQueuedThreads()：是否有线程等待此锁 
11. isFair()：该锁是否公平锁 
12. isHeldByCurrentThread()： 当前线程是否保持锁锁定，线程的执行 lock 方法的前后分别是 false 和 true 
13. isLock()：此锁是否有任意线程占用 
14. lockInterruptibly（）：如果当前线程未被中断，获取锁 
15. tryLock（）：尝试获得锁，仅在调用时锁未被线程占用，获得锁 
16. tryLock(long timeout TimeUnit unit)：如果锁在给定等待时间内没有被另一个线程保持，则获取该锁。 
非公平锁 
JVM 按随机、就近原则分配锁的机制则称为不公平锁，ReentrantLock 在构造函数中提供了是否公平锁的初始化方式，默认为非公平锁。非公平锁实际执行的效率要远远超出公平锁，除非程序有特殊需要，否则最常用非公平锁的分配机制。13/04/2018 Page 67 of 283 
公平锁 
公平锁指的是锁的分配机制是公平的，通常先对锁提出获取请求的线程会先被分配到锁， 
ReentrantLock 在构造函数中提供了是否公平锁的初始化方式来定义公平锁。 
ReentrantLock 与 synchronized 
1. ReentrantLock 通过方法 lock()与 unlock()来进行加锁与解锁操作，与 synchronized 会被 JVM 自动解锁机制不同，ReentrantLock 加锁后需要手动进行解锁。为了避免程序出现异常而无法正常解锁的情况，使用 ReentrantLock 必须在 finally 控制块中进行解锁操作。 
2. ReentrantLock 相比 synchronized 的优势是可中断、公平锁、多个锁。这种情况下需要使用 ReentrantLock。 
ReentrantLock 实现 
public class MyService { 
private Lock lock = new ReentrantLock(); 
//Lock lock=new ReentrantLock(true);//公平锁 
//Lock lock=new ReentrantLock(false);//非公平锁 
private Condition condition=lock.newCondition();//创建 Condition 
public void testMethod() { 
try { 
lock.lock();//lock 加锁 
//1：wait 方法等待： 
//System.out.println("开始 wait"); 
condition.await(); 
//通过创建 Condition 对象来使线程 wait，必须先执行 lock.lock 方法获得锁 
//:2：signal 方法唤醒 
condition.signal();//condition 对象的 signal 方法可以唤醒 wait 线程 
for (int i = 0; i < 5; i++) { 
System.out.println("ThreadName=" + Thread.currentThread().getName()+ (" " + (i + 1))); 
} 
} catch (InterruptedException e) { 
e.printStackTrace(); 
} 
finally{ 
lock.unlock(); 
} 
} 
} 
Condition 类和 Object 类锁方法区别区别 
1. Condition 类的 awiat 方法和 Object 类的 wait 方法等效 
2. Condition 类的 signal 方法和 Object 类的 notify 方法等效 
3. Condition 类的 signalAll 方法和 Object 类的 notifyAll 方法等效 
4. ReentrantLock 类可以唤醒指定条件的线程，而 object 的唤醒是随机的 
tryLock 和 lock 和 lockInterruptibly 的区别 
1. tryLock 能获得锁就返回 true，不能就立即返回 false，tryLock(long timeout,TimeUnit unit)，可以增加时间限制，如果超过该时间段还没获得锁，返回 false 
2. lock 能获得锁就返回 true，不能的话一直等待获得锁 
3. lock 和 lockInterruptibly，如果两个线程分别执行这两个方法，但此时中断这两个线程，lock 不会抛出异常，而 lockInterruptibly 会抛出异常。

1.1.9.6. Semaphore 信号量 
Semaphore 是一种基于计数的信号量。它可以设定一个阈值，基于此，多个线程竞争获取许可信号，做完自己的申请后归还，超过阈值后，线程申请许可信号将会被阻塞。Semaphore 可以用来构建一些对象池，资源池之类的，比如数据库连接池 
实现互斥锁（计数器为 1） 
我们也可以创建计数为 1 的 Semaphore，将其作为一种类似互斥锁的机制，这也叫二元信号量，表示两种互斥状态。 
代码实现 
它的用法如下： 
// 创建一个计数阈值为 5 的信号量对象 
// 只能 5 个线程同时访问 
Semaphore semp = new Semaphore(5); 
try { // 申请许可 
semp.acquire(); 
try { 
// 业务逻辑
} catch (Exception e) { 
} finally { 
// 释放许可 
semp.release(); 
} 
} catch (InterruptedException e) { 
} 
Semaphore 与 ReentrantLock 
Semaphore 基本能完成 ReentrantLock 的所有工作，使用方法也与之类似，通过 acquire()与release()方法来获得和释放临界资源。经实测，Semaphone.acquire()方法默认为可响应中断锁，与 ReentrantLock.lockInterruptibly()作用效果一致，也就是说在等待临界资源的过程中可以被Thread.interrupt()方法中断。此外，Semaphore 也实现了可轮询的锁请求与定时锁的功能，除了方法名 tryAcquire 与 tryLock不同，其使用方法与 ReentrantLock 几乎一致。Semaphore 也提供了公平与非公平锁的机制，也可在构造函数中进行设定。Semaphore 的锁释放操作也由手动进行，因此与 ReentrantLock 一样，为避免线程因抛出异常而无法正常释放锁的情况发生，释放锁的操作也必须在 finally 代码块中完成。

1.1.9.7. AtomicInteger 
首先说明，此处 AtomicInteger ，一个提供原子操作的 Integer 的类，常见的还有 
AtomicBoolean、AtomicInteger、AtomicLong、AtomicReference 等，他们的实现原理相同，区别在与运算对象类型的不同。令人兴奋地，还可以通过 AtomicReference<V>将一个对象的所有操作转化成原子操作。 
我们知道，在多线程程序中，诸如++i 或 i++等运算不具有原子性，是不安全的线程操作之一。 
通常我们会使用 synchronized 将该操作变成一个原子操作，但 JVM 为此类操作特意提供了一些同步类，使得使用更方便，且使程序运行效率变得更高。通过相关资料显示，通常AtomicInteger的性能是 ReentantLock 的好几倍。 
1.1.9.8. 可重入锁（递归锁） 
本文里面讲的是广义上的可重入锁，而不是单指 JAVA 下的 ReentrantLock。可重入锁，也叫做递归锁，指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响。在 JAVA 环境下 ReentrantLock 和 synchronized 都是 可重入锁。
1.1.9.9. 公平锁与非公平锁 
公平锁（Fair） 
加锁前检查是否有排队等待的线程，优先排队等待的线程，先来先得 
非公平锁（Nonfair） 
加锁时不考虑排队等待问题，直接尝试获取锁，获取不到自动到队尾等待 
1. 非公平锁性能比公平锁高 5~10 倍，因为公平锁需要在多核的情况下维护一个队列 
2. Java 中的 synchronized 是非公平锁，ReentrantLock 默认的 lock()方法采用的是非公平锁。 
1.1.9.10. ReadWriteLock 读写锁 
为了提高性能，Java 提供了读写锁，在读的地方使用读锁，在写的地方使用写锁，灵活控制，如果没有写锁的情况下，读是无阻塞的,在一定程度上提高了程序的执行效率。读写锁分为读锁和写锁，多个读锁不互斥，读锁与写锁互斥，这是由 jvm 自己控制的，你只要上好相应的锁即可。 
读锁 
如果你的代码只读数据，可以很多人同时读，但不能同时写，那就上读锁 
写锁 
如果你的代码修改数据，只能有一个人在写，且不能同时读取，那就上写锁。总之，读的时候上读锁，写的时候上写锁！ 
Java 中 读 写 锁 有 个 接 口 java.util.concurrent.locks.ReadWriteLock ， 也 有 具 体 的 实 现ReentrantReadWriteLock。 
1.1.9.11. 共享锁和独占锁 
java 并发包提供的加锁模式分为独占锁和共享锁。 
独占锁 
独占锁模式下，每次只能有一个线程能持有锁，ReentrantLock 就是以独占方式实现的互斥锁。 
独占锁是一种悲观保守的加锁策略，它避免了读/读冲突，如果某个只读线程获取锁，则其他读线程都只能等待，这种情况下就限制了不必要的并发性，因为读操作并不会影响数据的一致性。 
共享锁 
共享锁则允许多个线程同时获取锁，并发访问 共享资源，如：ReadWriteLock。共享锁则是一种乐观锁，它放宽了加锁策略，允许多个执行读操作的线程同时访问共享资源。 
1. AQS 的内部类 Node 定义了两个常量 SHARED 和 EXCLUSIVE，他们分别标识 AQS 队列中等待线程的锁获取模式。 
2. java 的并发包中提供了 ReadWriteLock，读-写锁。它允许一个资源可以被多个读操作访问，或者被一个 写操作访问，但两者不能同时进行。13/04/2018 Page 71 of 283 
1.1.9.12. 重量级锁（Mutex Lock） 
Synchronized 是通过对象内部的一个叫做监视器锁（monitor）来实现的。但是监视器锁本质又是依赖于底层的操作系统的 Mutex Lock 来实现的。而操作系统实现线程之间的切换这就需要从用户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，这就是为什么Synchronized 效率低的原因。因此，这种依赖于操作系统 Mutex Lock 所实现的锁我们称之为”重量级锁”。JDK 中对 Synchronized 做的种种优化，其核心都是为了减少这种重量级锁的使用。 
JDK1.6 以后，为了减少获得锁和释放锁所带来的性能消耗，提高性能，引入了“轻量级锁”和偏向锁”。 
1.1.9.13. 轻量级锁 
锁的状态总共有四种：无锁状态、偏向锁、轻量级锁和重量级锁。 
锁升级 
随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁（但是锁的升级是单向的也就是说只能从低到高升级，不会出现锁的降级）。“轻量级”是相对于使用操作系统互斥量来实现的传统锁而言的。但是，首先需要强调一点的是，轻量级锁并不是用来代替重量级锁的，它的本意是在没有多线程竞争的前提下，减少传统的重量 级锁使用产生的性能消耗。在解释轻量级锁的执行过程之前，先明白一点，轻量级锁所适应的场 景是线程交替执行同步块的情况，如果存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁。 
1.1.9.14. 偏向锁 
Hotspot 的作者经过以往的研究发现大多数情况下锁不仅不存在多线程竞争，而且总是由同一线程多次获得。偏向锁的目的是在某个线程获得锁之后，消除这个线程锁重入（CAS）的开销，看起来让这个线程得到了偏护。引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，因为轻量级锁的获取及释放依赖多次 CAS 原子指令，而偏向锁只需要在置换ThreadID 的时候依赖一次 CAS 原子指令（由于一旦出现多线程竞争的情况就必须撤销偏向锁，所以偏向锁的撤销操作的性能损耗必须小于节省下来的 CAS 原子指令的性能消耗）。上面说过，轻量级锁是为了在线程交替执行同步块时提高性能，而偏向锁则是在只有一个线程执行同步块时进一步提高性能。 
1.1.9.15. 分段锁 
分段锁也并非一种实际的锁，而是一种思想 ConcurrentHashMap 是学习分段锁的最好实践 
4.1.9.16. 锁优化
减少锁持有时间 
只用在有线程安全要求的程序上加锁 
减小锁粒度 
将大对象（这个对象可能会被很多线程访问），拆成小对象，大大增加并行度，降低锁竞争。 
降低了锁的竞争，偏向锁，轻量级锁成功率才会提高。最最典型的减小锁粒度的案例就是 
ConcurrentHashMap。 
锁分离 
最常见的锁分离就是读写锁 ReadWriteLock，根据功能进行分离成读锁和写锁，这样读读不互斥，读写互斥，写写互斥，即保证了线程安全，又提高了性能，具体也请查看[高并发 Java 五] 
JDK 并发包 1。读写分离思想可以延伸，只要操作互不影响，锁就可以分离。比如 
LinkedBlockingQueue 从头部取出，从尾部放数据 
锁粗化 
通常情况下，为了保证多线程间的有效并发，会要求每个线程持有锁的时间尽量短，即在使用完公共资源后，应该立即释放锁。但是，凡事都有一个度，如果对同一个锁不停的进行请求、同步和释放，其本身也会消耗系统宝贵的资源，反而不利于性能的优化 。 
锁消除 
锁消除是在编译器级别的事情。在即时编译器时，如果发现不可能被共享的对象，则可以消除这些对象的锁操作，多数是因为程序员编码不规范引起。 


1.1.10. 线程基本方法 
线程相关的基本方法有 wait，notify，notifyAll，sleep，join，yield 等。

1.1.10.1. 线程等待（wait） 
调用该方法的线程进入 WAITING 状态，只有等待另外线程的通知或被中断才会返回，需要注意的是调用 wait()方法后，会释放对象的锁。因此，wait 方法一般用在同步方法或同步代码块中。 
1.1.10.2. 线程睡眠（sleep） 
sleep 导致当前线程休眠，与 wait 方法不同的是 sleep 不会释放当前占有的锁,sleep(long)会导致线程进入 TIMED-WATING 状态，而 wait()方法会导致当前线程进入 WATING 状态 
1.1.10.3. 线程让步（yield） 
yield 会使当前线程让出 CPU 执行时间片，与其他线程一起重新竞争 CPU 时间片。一般情况下，优先级高的线程有更大的可能性成功竞争得到 CPU 时间片，但这又不是绝对的，有的操作系统对线程优先级并不敏感。 
1.1.10.4. 线程中断（interrupt） 
中断一个线程，其本意是给这个线程一个通知信号，会影响这个线程内部的一个中断标识位。这个线程本身并不会因此而改变状态(如阻塞，终止等)。 
1. 调用 interrupt()方法并不会中断一个正在运行的线程。也就是说处于 Running 状态的程并不会因为被中断而被终止，仅仅改变了内部维护的中断标识位而已。 
2. 若调用 sleep()而使线程处于 TIMED-WATING 状态，这时调用 interrupt()方法，会抛出InterruptedException,从而使线程提前结束 TIMED-WATING 状态。
3. 许多声明抛出 InterruptedException 的方法(如 Thread.sleep(long mills 方法))，抛出异常前，都会清除中断标识位，所以抛出异常后，调用 isInterrupted()方法将会返回 false。 
4. 中断状态是线程固有的一个标识位，可以通过此标识位安全的终止线程。比如,你想终止 
一个线程 thread 的时候，可以调用 thread.interrupt()方法，在线程的 run 方法内部可以 根据 thread.isInterrupted()的值来优雅的终止线程。 
1.1.10.5. Join 等待其他线程终止 
join() 方法，等待其他线程终止，在当前线程中调用一个线程的 join() 方法，则当前线程转为阻塞状态，回到另一个线程结束，当前线程再由阻塞状态变为就绪状态，等待 cpu 的宠幸。 
1.1.10.6. 为什么要用 join()方法？ 
很多情况下，主线程生成并启动了子线程，需要用到子线程返回的结果，也就是需要主线程需要在子线程结束后再结束，这时候就要用到 join() 方法。 
System.out.println(Thread.currentThread().getName() + "线程运行开始!"); 
Thread6 thread1 = new Thread6(); 
thread1.setName("线程 B"); 
thread1.join(); 
System.out.println("这时 thread1 执行完毕之后才能执行主线程"); 
1.1.10.7. 线程唤醒（notify） 
Object 类中的 notify() 方法，唤醒在此对象监视器上等待的单个线程，如果所有线程都在此对象上等待，则会选择唤醒其中一个线程，选择是任意的，并在对实现做出决定时发生，线程通过调用其中一个 wait() 方法，在对象的监视器上等待，直到当前的线程放弃此对象上的锁定，才能继续执行被唤醒的线程，被唤醒的线程将以常规方式与在该对象上主动同步的其他所有线程进行竞争。类似的方法还有 notifyAll() ，唤醒再次监视器上等待的所有线程。 
1.1.10.8. 其他方法： 
1. sleep()：强迫一个线程睡眠Ｎ毫秒。 
2. isAlive()： 判断一个线程是否存活。 
3. join()： 等待线程终止。 
4. activeCount()： 程序中活跃的线程数。 
5. enumerate()： 枚举程序中的线程。 
6. currentThread()： 得到当前线程。 
7. isDaemon()： 一个线程是否为守护线程。 
8. setDaemon()： 设置一个线程为守护线程。(用户线程和守护线程的区别在于，是否等待主线程依赖于主线程结束而结束) 
9. setName()： 为线程设置一个名称。 
10. wait()： 强迫一个线程等待。
11. notify()： 通知一个线程继续运行。 
12. setPriority()： 设置一个线程的优先级。 
13. getPriority():：获得一个线程的优先级。


1.1.11. 线程上下文切换 
巧妙地利用了时间片轮转的方式, CPU 给每个任务都服务一定的时间，然后把当前任务的状态保存下来，在加载下一任务的状态后，继续服务下一任务，任务的状态保存及再加载, 这段过程就叫做上下文切换。时间片轮转的方式使多个任务在同一颗 CPU 上执行变成了可能。 

1.1.11.1. 进程 
（有时候也称做任务）是指一个程序运行的实例。在 Linux 系统中，线程就是能并行运行并且与他们的父进程（创建他们的进程）共享同一地址空间（一段内存区域）和其他资源的轻量级的进程。 
1.1.11.2. 上下文 
是指某一时间点 CPU 寄存器和程序计数器的内容。 
1.1.11.3. 寄存器 
是 CPU 内部的数量较少但是速度很快的内存（与之对应的是 CPU 外部相对较慢的 RAM 主内存）。寄存器通过对常用值（通常是运算的中间值）的快速访问来提高计算机程序运行的速度。 
1.1.11.4. 程序计数器 
是一个专用的寄存器，用于表明指令序列中 CPU 正在执行的位置，存的值为正在执行的指令的位置或者下一个将要被执行的指令的位置，具体依赖于特定的系统。 
1.1.11.5. PCB-“切换桢” 
上下文切换可以认为是内核（操作系统的核心）在 CPU 上对于进程（包括线程）进行切换，上下文切换过程中的信息是保存在进程控制块（PCB, process control block）中的。PCB 还经常被称作“切换桢”（switchframe）。信息会一直保存到 CPU 的内存中，直到他们被再次使用。
1.1.11.6. 上下文切换的活动： 
1. 挂起一个进程，将这个进程在 CPU 中的状态（上下文）存储于内存中的某处。 
2. 在内存中检索下一个进程的上下文并将其在 CPU 的寄存器中恢复。 
3. 跳转到程序计数器所指向的位置（即跳转到进程被中断时的代码行），以恢复该进程在程序中。 
1.1.11.7. 引起线程上下文切换的原因 
1. 当前执行任务的时间片用完之后，系统 CPU 正常调度下一个任务； 
2. 当前执行任务碰到 IO 阻塞，调度器将此任务挂起，继续下一任务； 
3. 多个任务抢占锁资源，当前任务没有抢到锁资源，被调度器挂起，继续下一任务； 
4. 用户代码挂起当前任务，让出 CPU 时间； 
5. 硬件中断； 
1.1.12. 同步锁与死锁 
1.1.12.1. 同步锁 
当多个线程同时访问同一个数据时，很容易出现问题。为了避免这种情况出现，我们要保证线程同步互斥，就是指并发执行的多个线程，在同一时间内只允许一个线程访问共享数据。 Java 中可以使用 synchronized 关键字来取得一个对象的同步锁。 
1.1.12.2. 死锁 
何为死锁，就是多个线程同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。 
1.1.13. 线程池原理 
线程池做的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量超出数量的线程排队等候，等其它线程执行完毕再从队列中取出任务来执行。他的主要特点为：线程复用；控制最大并发数；管理线程。 
1.1.13.1. 线程复用 
每一个 Thread 的类都有一个 start 方法。 当调用 start 启动线程时 Java 虚拟机会调用该类的 run方法。 那么该类的 run() 方法中就是调用了 Runnable 对象的 run() 方法。 我们可以继承重写Thread 类，在其 start 方法中添加不断循环调用传递过来的 Runnable 对象。 这就是线程池的实现原理。循环方法中不断获取 Runnable 是用 Queue 实现的，在获取下一个 Runnable 之前可以是阻塞的。 
1.1.13.2. 线程池的组成 
一般的线程池主要分为以下 4 个组成部分：
1. 线程池管理器：用于创建并管理线程池 
2. 工作线程：线程池中的线程 
3. 任务接口：每个任务必须实现的接口，用于工作线程调度其运行 
4. 任务队列：用于存放待处理的任务，提供一种缓冲机制Java 中的线程池是通过Executor 框架实现的，该框架中用到了 Executor，Executors，ExecutorServiceThreadPoolExecutor ，Callable 和 Future、FutureTask 这几个类。 
ThreadPoolExecutor 的构造方法如下： 
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize, long keepAliveTime, 
TimeUnit unit, BlockingQueue<Runnable> workQueue) { 
this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue, 
Executors.defaultThreadFactory(), defaultHandler); 
} 
1. corePoolSize：指定了线程池中的线程数量。 
2. maximumPoolSize：指定了线程池中的最大线程数量。 
3. keepAliveTime：当前线程池数量超过 corePoolSize 时，多余的空闲线程的存活时间，即多次时间内会被销毁。 
4. unit：keepAliveTime 的单位。 
5. workQueue：任务队列，被提交但尚未被执行的任务。 
6. threadFactory：线程工厂，用于创建线程，一般用默认的即可。 
7. handler：拒绝策略，当任务太多来不及处理，如何拒绝任务。
1.1.13.3. 拒绝策略 
线程池中的线程已经用完了，无法继续为新任务服务，同时，等待队列也已经排满了，再也 
塞不下新任务了。这时候我们就需要拒绝策略机制合理的处理这个问题。 
JDK 内置的拒绝策略如下： 
1. AbortPolicy ： 直接抛出异常，阻止系统正常运行。 
2. CallerRunsPolicy ： 只要线程池未关闭，该策略直接在调用者线程中，运行当前被丢弃的任务。显然这样做不会真的丢弃任务，但是，任务提交线程的性能极有可能会急剧下降。 
3. DiscardOldestPolicy ： 丢弃最老的一个请求，也就是即将被执行的一个任务，并尝试再次提交当前任务。 
4. DiscardPolicy ： 该策略默默地丢弃无法处理的任务，不予任何处理。如果允许任务丢 
失，这是最好的一种方案。以上内置拒绝策略均实现了 RejectedExecutionHandler 接口，若以上策略仍无法满足实际需要，完全可以自己扩展 RejectedExecutionHandler 接口。 
1.1.13.4. Java 线程池工作过程 
1. 线程池刚创建时，里面没有一个线程。任务队列是作为参数传进来的。不过，就算队列里面有任务，线程池也不会马上执行它们。 
2. 当调用 execute() 方法添加一个任务时，线程池会做如下判断： 
a) 如果正在运行的线程数量小于 corePoolSize，那么马上创建线程运行这个任务； 
b) 如果正在运行的线程数量大于或等于 corePoolSize，那么将这个任务放入队列； 
c) 如果这时候队列满了，而且正在运行的线程数量小于 maximumPoolSize，那么还是要 
创建非核心线程立刻运行这个任务； 
d) 如果队列满了，而且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池 
会抛出异常 RejectExecutionException。 
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。 
4. 当一个线程无事可做，超过一定的时间（keepAliveTime）时，线程池会判断，如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小。

1.1.14. JAVA 阻塞队列原理
1.1.14.2. Java 中的阻塞队列 
1. ArrayBlockingQueue ：由数组结构组成的有界阻塞队列。 
2. LinkedBlockingQueue ：由链表结构组成的有界阻塞队列。 
3. PriorityBlockingQueue ：支持优先级排序的无界阻塞队列。 
4. DelayQueue：使用优先级队列实现的无界阻塞队列。 
5. SynchronousQueue：不存储元素的阻塞队列。 
6. LinkedTransferQueue：由链表结构组成的无界阻塞队列。 
7. LinkedBlockingDeque：由链表结构组成的双向阻塞队列
1.1.14.3. ArrayBlockingQueue（公平、非公平） 
用数组实现的有界阻塞队列。此队列按照先进先出（FIFO）的原则对元素进行排序。默认情况下不保证访问者公平的访问队列，所谓公平访问队列是指阻塞的所有生产者线程或消费者线程，当队列可用时，可以按照阻塞的先后顺序访问队列，即先阻塞的生产者线程，可以先往队列里插入元素，先阻塞的消费者线程，可以先从队列里获取元素。通常情况下为了保证公平性会降低吞吐量。我们可以使用以下代码创建一个公平的阻塞队列： 
ArrayBlockingQueue fairQueue = new ArrayBlockingQueue(1000,true); 
1.1.14.4. LinkedBlockingQueue（两个独立锁提高并发） 
基于链表的阻塞队列，同 ArrayListBlockingQueue 类似，此队列按照先进先出（FIFO）的原则对元素进行排序。而 LinkedBlockingQueue 之所以能够高效的处理并发数据，还因为其对于生产者端和消费者端分别采用了独立的锁来控制数据同步，这也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能。 
LinkedBlockingQueue 会默认一个类似无限大小的容量（Integer.MAX_VALUE）。 
1.1.14.5. PriorityBlockingQueue（compareTo 排序实现优先） 
是一个支持优先级的无界队列。默认情况下元素采取自然顺序升序排列。可以自定义实现 
compareTo()方法来指定元素进行排序规则，或者初始化 PriorityBlockingQueue 时，指定构造参数 Comparator 来对元素进行排序。需要注意的是不能保证同优先级元素的顺序。 
1.1.14.6. DelayQueue（缓存失效、定时任务 ） 
是一个支持延时获取元素的无界阻塞队列。队列使用 PriorityQueue 来实现。队列中的元素必须实现 Delayed 接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素。我们可以将 DelayQueue 运用在以下应用场景： 
1. 缓存系统的设计：可以用 DelayQueue 保存缓存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从 DelayQueue 中获取元素时，表示缓存有效期到了。
2. 定时任务调度：使用 DelayQueue 保存当天将会执行的任务和执行时间，一旦从 
DelayQueue 中获取到任务就开始执行，从比如 TimerQueue 就是使用 DelayQueue 实现的。 
1.1.14.7. SynchronousQueue（不存储数据、可用于传递数据） 
是一个不存储元素的阻塞队列。每一个 put 操作必须等待一个 take 操作，否则不能继续添加元素。SynchronousQueue 可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合于传递性场景,比如在一个线程中使用的数据，传递给另 外 一 个 线 程 使 用 ， SynchronousQueue 的 吞 吐 量 高 于 LinkedBlockingQueue 和ArrayBlockingQueue。 
1.1.14.8. LinkedTransferQueue 
是 一 个 由 链 表 结 构 组 成 的 无 界 阻 塞 TransferQueue 队 列 。 相 对 于 其 他 阻 塞 队 列 ，LinkedTransferQueue 多了 tryTransfer 和 transfer 方法。 
1. transfer 方法：如果当前有消费者正在等待接收元素（消费者使用 take()方法或带时间限制的poll()方法时），transfer 方法可以把生产者传入的元素立刻 transfer（传输）给消费者。如果没有消费者在等待接收元素，transfer 方法会将元素存放在队列的 tail 节点，并等到该元素被消费者消费了才返回。 
2. tryTransfer 方法。则是用来试探下生产者传入的元素是否能直接传给消费者。如果没有消费者等待接收元素，则返回 false。和 transfer 方法的区别是 tryTransfer 方法无论消费者是否接收，方法立即返回。而 transfer 方法是必须等到消费者消费了才返回。 
对于带有时间限制的 tryTransfer(E e, long timeout, TimeUnit unit)方法，则是试图把生产者传入的元素直接传给消费者，但是如果没有消费者消费该元素则等待指定的时间再返回，如果超时还没消费元素，则返回 false，如果在超时时间内消费了元素，则返回 true。 
1.1.14.9. LinkedBlockingDeque 
是一个由链表结构组成的双向阻塞队列。所谓双向队列指的你可以从队列的两端插入和移出元素。双端队列因为多了一个操作队列的入口，在多线程同时入队时，也就减少了一半的竞争。相比其他的阻塞队列，LinkedBlockingDeque 多了 addFirst，addLast，offerFirst，offerLast，peekFirst，peekLast 等方法，以 First 单词结尾的方法，表示插入，获取（peek）或移除双端队列的第一个元素。以 Last 单词结尾的方法，表示插入，获取或移除双端队列的最后一个元素。另外插入方法 add 等同于 addLast，移除方法 remove 等效于removeFirst。但是 take 方法却等同于 takeFirst，不知道是不是 Jdk 的 bug，使用时还是用带有 First 和 Last 后缀的方法更清楚。在初始化 LinkedBlockingDeque 时可以设置容量防止其过渡膨胀。另外双向阻塞队列可以运用在“工作窃取”模式中。

1.1.15. CyclicBarrier、CountDownLatch、Semaphore 的用法 
1.1.15.1. CountDownLatch（线程计数器 ） 
CountDownLatch 类位于 java.util.concurrent 包下，利用它可以实现类似计数器的功能。比如有一个任务 A，它要等待其他 4 个任务执行完毕之后才能执行，此时就可以利用 CountDownLatch来实现这种功能了。 
final CountDownLatch latch = new CountDownLatch(2); 
new Thread(){public void run() { 
System.out.println("子线程"+Thread.currentThread().getName()+"正在执行"); 
Thread.sleep(3000); 
System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕"); 
latch.countDown(); 
};}.start(); 
new Thread(){ public void run() { 
System.out.println("子线程"+Thread.currentThread().getName()+"正在执行"); 
Thread.sleep(3000); 
System.out.println("子线程"+Thread.currentThread().getName()+"执行完毕"); 
latch.countDown(); 
};}.start(); 
System.out.println("等待 2 个子线程执行完毕..."); 
latch.await(); 
System.out.println("2 个子线程已经执行完毕"); 
System.out.println("继续执行主线程"); 
} 
1.1.15.2. CyclicBarrier（回环栅栏-等待至 barrier 状态再全部同时执行） 
字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier 可以被重用。我们暂且把这个状态就叫做barrier，当调用 await()方法之后，线程就处于 barrier 了。 
CyclicBarrier 中最重要的方法就是 await 方法，它有 2 个重载版本： 
1. public int await()：用来挂起当前线程，直至所有线程都到达 barrier 状态再同时执行后续任务； 
2. public int await(long timeout, TimeUnit unit)：让这些线程等待至一定的时间，如果还有线程没有到达 barrier 状态就直接让到达 barrier 的线程执行后续任务。
具体使用如下，另外 CyclicBarrier 是可以重用的。 
public static void main(String[] args) { 
int N = 4; 
CyclicBarrier barrier = new CyclicBarrier(N); 
for(int i=0;i<N;i++) 
new Writer(barrier).start(); 
} 
static class Writer extends Thread{ 
private CyclicBarrier cyclicBarrier; 
public Writer(CyclicBarrier cyclicBarrier) { 
this.cyclicBarrier = cyclicBarrier; 
} 
@Override 
public void run() { 
try { 
Thread.sleep(5000); //以睡眠来模拟线程需要预定写入数据操作 
System.out.println("线程"+Thread.currentThread().getName()+"写入数据完 
毕，等待其他线程写入完毕"); 
cyclicBarrier.await(); 
} catch (InterruptedException e) { 
e.printStackTrace(); 
}catch(BrokenBarrierException e){ 
e.printStackTrace(); 
} 
System.out.println("所有线程写入完毕，继续处理其他任务，比如数据操作"); 
} 
} 
1.1.15.3. Semaphore（信号量-控制同时访问的线程个数） 
Semaphore 翻译成字面意思为 信号量，Semaphore 可以控制同时访问的线程个数，通过acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。 
Semaphore 类中比较重要的几个方法： 
1. public void acquire(): 用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。 
2. public void acquire(int permits):获取 permits 个许可 
3. public void release() { } :释放许可。注意，在释放许可之前，必须先获获得许可。 
4. public void release(int permits) { }:释放 permits 个许可 
上面 4 个方法都会被阻塞，如果想立即得到执行结果，可以使用下面几个方法
1. public boolean tryAcquire():尝试获取一个许可，若获取成功，则立即返回 true，若获取失败，则立即返回 false 
2. public boolean tryAcquire(long timeout, TimeUnit unit):尝试获取一个许可，若在指定的时间内获取成功，则立即返回 true，否则则立即返回 false 
3. public boolean tryAcquire(int permits):尝试获取 permits 个许可，若获取成功，则立即返回 true，若获取失败，则立即返回 false 
4. public boolean tryAcquire(int permits, long timeout, TimeUnit unit): 尝试获取 permits个许可，若在指定的时间内获取成功，则立即返回 true，否则则立即返回 false 
5. 还可以通过 availablePermits()方法得到可用的许可数目。 
例子：若一个工厂有 5 台机器，但是有 8 个工人，一台机器同时只能被一个工人使用，只有使用完了，其他工人才能继续使用。那么我们就可以通过 Semaphore 来实现： 
int N = 8; //工人数 
Semaphore semaphore = new Semaphore(5); //机器数目 
for(int i=0;i<N;i++) 
new Worker(i,semaphore).start(); 
} 
static class Worker extends Thread{ 
private int num; 
private Semaphore semaphore; 
public Worker(int num,Semaphore semaphore){ 
this.num = num; 
this.semaphore = semaphore; 
} 
@Override 
public void run() { 
try { 
semaphore.acquire(); 
System.out.println("工人"+this.num+"占用一个机器在生产..."); 
Thread.sleep(2000); 
System.out.println("工人"+this.num+"释放出机器"); 
semaphore.release(); 
} catch (InterruptedException e) { 
e.printStackTrace(); 
} 
} 
 CountDownLatch 和 CyclicBarrier 都能够实现线程之间的等待，只不过它们侧重点不同；CountDownLatch 一般用于某个线程 A 等待若干个其他线程执行完任务之后，它才执行；而 CyclicBarrier 一般用于一组线程互相等待至某个状态，然后这一组线程再同时 
执行；另外，CountDownLatch 是不能够重用的，而 CyclicBarrier 是可以重用的。 
 Semaphore 其实和锁有点类似，它一般用于控制对某组资源的访问权限。

1.1.16. volatile 关键字的作用（变量可见性、禁止重排序） 
Java 语言提供了一种稍弱的同步机制，即 volatile 变量，用来确保将变量的更新操作通知到其他线程。volatile 变量具备两种特性，volatile 变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取 volatile 类型的变量时总会返回最新写入的值。 
变量可见性 
其一是保证该变量对所有线程可见，这里的可见性指的是当一个线程修改了变量的值，那么新的值对于其他线程是可以立即获取的。 
禁止重排序 
volatile 禁止了指令重排。 
比 sychronized 更轻量级的同步锁 
在访问 volatile 变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此 volatile 变量是一种比 sychronized 关键字更轻量级的同步机制。volatile 适合这种场景：一个变量被多个线程共享，线程直接给这个变量赋值。 


1.1.19. synchronized 和 ReentrantLock 的区别 
1.1.19.1. 两者的共同点： 
1. 都是用来协调多线程对共享对象、变量的访问 
2. 都是可重入锁，同一线程可以多次获得同一个锁 
3. 都保证了可见性和互斥性13/04/2018 Page 92 of 283 
1.1.19.2. 两者的不同点： 
1. ReentrantLock 显示的获得、释放锁，synchronized 隐式获得释放锁 
2. ReentrantLock 可响应中断、可轮回，synchronized 是不可以响应中断的，为处理锁的不可用性提供了更高的灵活性 
3. ReentrantLock 是 API 级别的，synchronized 是 JVM 级别的 
4. ReentrantLock 可以实现公平锁 
5. ReentrantLock 通过 Condition 可以绑定多个条件 
6. 底层实现不一样， synchronized 是同步阻塞，使用的是悲观并发策略，lock 是同步非阻塞，采用的是乐观并发策略 
7. Lock 是一个接口，而 synchronized 是 Java 中的关键字，synchronized 是内置的语言实现。 
8. synchronized 在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生； 
而 Lock 在发生异常时，如果没有主动通过 unLock()去释放锁，则很可能造成死锁现象， 
因此使用 Lock 时需要在 finally 块中释放锁。 
9. Lock 可以让等待锁的线程响应中断，而 synchronized 却不行，使用 synchronized 时，等待的线程会一直等待下去，不能够响应中断。 
10. 通过 Lock 可以知道有没有成功获取锁，而 synchronized 却无法办到。 
11. Lock 可以提高多个线程进行读操作的效率，既就是实现读写锁等。

1.1.21. Java 中用到的线程调度 
1.1.21.1. 抢占式调度： 
抢占式调度指的是每条线程执行的时间、线程的切换都由系统控制，系统控制指的是在系统某种运行机制下，可能每条线程都分同样的执行时间片，也可能是某些线程执行的时间片较长，甚至某些线程得不到执行的时间片。在这种机制下，一个线程的堵塞不会导致整个进程堵塞。 
1.1.21.2. 协同式调度： 
协同式调度指某一线程执行完后主动通知系统切换到另一线程上执行，这种模式就像接力赛一样，一个人跑完自己的路程就把接力棒交接给下一个人，下个人继续往下跑。线程的执行时间由线程本身控制，线程切换可以预知，不存在多线程同步问题，但它有一个致命弱点：如果一个线程编写有问题，运行到一半就一直堵塞，那么可能导致整个系统崩溃。

1.1.21.3. JVM 的线程调度实现（抢占式调度） 
java 使用的线程调使用抢占式调度，Java 中线程会按优先级分配 CPU 时间片运行，且优先级越高越优先执行，但优先级高并不代表能独自占用执行时间片，可能是优先级高得到越多的执行时间片，反之，优先级低的分到的执行时间少但不会分配不到执行时间。 
1.1.21.4. 线程让出 cpu 的情况： 
1. 当前运行线程主动放弃 CPU，JVM 暂时放弃 CPU 操作（基于时间片轮转调度的 JVM 操作系统不会让线程永久放弃 CPU，或者说放弃本次时间片的执行权），例如调用 yield()方法。 
2. 当前运行线程因为某些原因进入阻塞状态，例如阻塞在 I/O 上。 
3. 当前运行线程结束，即运行完 run()方法里面的任务。


1.1.22. 进程调度算法 
1.1.22.1. 优先调度算法 
1. 先来先服务调度算法（FCFS） 
当在作业调度中采用该算法时，每次调度都是从后备作业队列中选择一个或多个最先进入该队列的作业，将它们调入内存，为它们分配资源、创建进程，然后放入就绪队列。在进程调度中采用 FCFS 算法时，则每次调度是从就绪队列中选择一个最先进入该队列的进程，为之分配处理机，使之投入运行。该进程一直运行到完成或发生某事件而阻塞后才放弃处理机，特点是：算法比较简单，可以实现基本上的公平。 
2. 短作业(进程)优先调度算法 
短作业优先(SJF)的调度算法是从后备队列中选择一个或若干个估计运行时间最短的作业，将它们调入内存运行。而短进程优先(SPF)调度算法则是从就绪队列中选出一个估计运行时间最短的进程，将处理机分配给它，使它立即执行并一直执行到完成，或发生某事件而被阻塞放弃处理机时再重新调度。该算法未照顾紧迫型作业。

1.1.23. 什么是 CAS（比较并交换-乐观锁机制-锁自旋） 
1.1.23.1. 概念及特性 
CAS（Compare And Swap/Set）比较并交换，CAS 算法的过程是这样：它包含 3 个参数 CAS(V,E,N)。V 表示要更新的变量(内存值)，E 表示预期值(旧的)，N 表示新值。当且仅当 V 值等于 E 值时，才会将 V 的值设为 N，如果 V 值和 E 值不同，则说明已经有其他线程做了更新，则当前线程什么都不做。最后，CAS 返回当前 V 的真实值。 
CAS 操作是抱着乐观的态度进行的(乐观锁)，它总是认为自己可以成功完成操作。当多个线程同时使用 CAS 操作一个变量时，只有一个会胜出，并成功更新，其余均会失败。失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。基于这样的原理，CAS 操作即使没有锁，也可以发现其他线程对当前线程的干扰，并进行恰当的处理。 
1.1.23.2. 原子包 java.util.concurrent.atomic（锁自旋） 
JDK1.5 的原子包：java.util.concurrent.atomic 这个包里面提供了一组原子类。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由 JVM 从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。相对于对于 synchronized 这种阻塞算法，CAS 是非阻塞算法的一种常见实现。由于一般 CPU 切换时间比 CPU 指令集操作更加长， 所以 J.U.C 在性能上有了很大的提升。
1.1.23.3. ABA 问题 
CAS 会导致“ABA 问题”。CAS 算法实现一个重要前提需要取出内存中某时刻的数据，而在下时刻比较并替换，那么在这个时间差类会导致数据的变化。比如说一个线程 one 从内存位置 V 中取出 A，这时候另一个线程 two 也从内存中取出 A，并且two 进行了一些操作变成了 B，然后 two 又将 V 位置的数据变成 A，这时候线程 one 进行 CAS 操 
作发现内存中仍然是 A，然后 one 操作成功。尽管线程 one 的 CAS 操作成功，但是不代表这个过程就是没有问题的。 
部分乐观锁的实现是通过版本号（version）的方式来解决 ABA 问题，乐观锁每次在执行数据的修 改操作时，都会带上一个版本号，一旦版本号和数据的版本号一致就可以执行修改操作并对版本号执行+1 操作，否则就执行失败。因为每次操作的版本号都会随之增加，所以不会出现 ABA 问题，因为版本号只会增加不会减少。


1.1.24. 什么是 AQS（抽象的队列同步器） 
AbstractQueuedSynchronizer 类如其名，抽象的队列式的同步器，AQS 定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的 
ReentrantLock/Semaphore/CountDownLatch。它维护了一个 volatile int state（代表共享资源）和一个 FIFO 线程等待队列（多线程争用资源被阻塞时会进入此队列）。这里 volatile 是核心关键词，具体 volatile 的语义，在此不述。state 的访问方式有三种: 
getState() 
setState() 
compareAndSetState() 
AQS 定义两种资源共享方式 
Exclusive 独占资源-ReentrantLock 
Exclusive（独占，只有一个线程能执行，如 ReentrantLock） 
Share 共享资源-Semaphore/CountDownLatch 
Share（共享，多个线程可同时执行，如 Semaphore/CountDownLatch）。 
AQS 只是一个框架，具体资源的获取/释放方式交由自定义同步器去实现，AQS 这里只定义了一个接口，具体资源的获取交由自定义同步器去实现了（通过 state 的 get/set/CAS)之所以没有定义成abstract ，是 因 为独 占模 式 下 只 用实tryAcquire-tryRelease ，而 共享 模 式 下 只用 实 现tryAcquireShared-tryReleaseShared。如果都定义成abstract，那么每个模式也要去实现另一模式下的接口。不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/ 唤醒出队等），AQS 已经在顶层实现好了。自定义同步器实现时主要实现以下几种方法： 
1．isHeldExclusively()：该线程是否正在独占资源。只有用到 condition 才需要去实现它。 
2．tryAcquire(int)：独占方式。尝试获取资源，成功则返回 true，失败则返回 false。 
3．tryRelease(int)：独占方式。尝试释放资源，成功则返回 true，失败则返回 false。 
4．tryAcquireShared(int)：共享方式。尝试获取资源。负数表示失败；0 表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。 
5．tryReleaseShared(int)：共享方式。尝试释放资源，如果释放后允许唤醒后续等待结点返回true，否则返回 false。
同步器的实现是 ABS 核心（state 资源状态计数） 
同步器的实现是 ABS 核心，以 ReentrantLock 为例，state 初始化为 0，表示未锁定状态。A 线程 lock()时，会调用 tryAcquire()独占该锁并将 state+1。此后，其他线程再 tryAcquire()时就会失败，直到 A 线程 unlock()到 state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（state 会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证 state 是能回到零态的。 
以 CountDownLatch 以例，任务分为 N 个子线程去执行，state 也初始化为 N（注意 N 要与线程个数一致）。这 N 个子线程是并行执行的，每个子线程执行完后 countDown()一次，state会 CAS 减 1。等到所有子线程都执行完后(即 state=0)，会 unpark()主调用线程，然后主调用线程就会从 await()函数返回，继续后余动作。 
ReentrantReadWriteLock 实现独占和共享两种方式 
一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现 tryAcquire
tryRelease、tryAcquireShared-tryReleaseShared 中的一种即可。但 AQS 也支持自定义同步器同时实现独占和共享两种方式，如 ReentrantReadWriteLock。




例题
1、并行和并发有什么区别？
1. 并行是指两个或者多个事件在同一时刻发生；而并发是指两个或多个事件在	同一时间间隔发生；
2. 并行是在不同实体上的多个事件，并发是在同一实体上的多个事件；
3. 在一台处理器上“同时”处理多个任务，在多台处理器上同时处理多个任务。	如 Hadoop 分布式集群。所以并发编程的目标是充分的利用处理器的每一个	核，以达到最高的处理性能。
2、线程和进程的区别？
进程：是程序运行和资源分配的基本单位，一个程序至少有一个进程，一个进程至少有一个线程。进程在执行过程中拥有独立的内存单元，而多个线程共享内存资源，减少切换次数，从而效率更高。
线程：是进程的一个实体，是 cpu 调度和分派的基本单位，是比程序更小的能独立运行的基本单位。同一进程中的多个线程之间可以并发执行。
3、守护线程是什么？
守护线程（即 Daemon thread），是个服务线程，准确地来说就是服务其他的线程。
4、创建线程的几种方式？
1.继承 Thread 类创建线程；
2.实现 Runnable 接口创建线程；
3.通过 Callable 和 Future 创建线程；
4. 通过线程池创建线程。
5、Runnable 和 Callable 有什么区别？
1. Runnable 接口中的 run() 方法的返回值是 void，它做的事情只是纯粹地去执行 run() 方法中的代码而已；
2. Callable 接口中的 call() 方法是有返回值的，是一个泛型，和 Future、FutureTask 配合可以用来获取异步执行的结果。
6、线程状态及转换？
Thread 的源码中定义了6种状态：new（新建）、runnnable（可运行）、blocked（阻塞）、waiting（等待）、time waiting （定时等待）和 terminated（终止）。

线程状态转换如下图所示：


7、sleep() 和 wait() 的区别？
1. sleep() 方法正在执行的线程主动让出 cpu（然后 cpu 就可以去执行其他任务），在 sleep 指定时间后 cpu 再回到该线程继续往下执行（注意：sleep 方法只让出了 cpu，而并不会释放同步资源锁）；而 wait()  方法则是指当前线程让自己暂时退让出同步资源锁，以便其他正在等待该资源的线程得到该资源进而运行，只有调用了 notify() 方法，之前调用 wait() 的线程才会解除 wait 状态，可以去参与竞争同步资源锁，进而得到执行。（注意：notify 的作用相当于叫醒睡着的人，而并不会给他分配任务，就是说 notify 只是让之前调用 wait 的线程有权利重新参与线程的调度）；
2.sleep() 方法可以在任何地方使用，而 wait() 方法则只能在同步方法或同步块中使用；
3.sleep() 是线程类（Thread）的方法，调用会暂停此线程指定的时间，但监控依然保持，不会释放对象锁，到时间自动恢复；wait() 是 Object 的方法，调用会放弃对象锁，进入等待队列，待调用 notify()/notifyAll() 唤醒指定的线程或者所有线程，才会进入锁池，不再次获得对象锁才会进入运行状态。
8、线程的 run() 和 start() 有什么区别？
1. 每个线程都是通过某个特定 Thread 对象所对应的方法 run() 来完成其操作的，方法 run() 称为线程体。通过调用 Thread 类的 start() 方法来启动一个线程；
2.start() 方法来启动一个线程，真正实现了多线程运行。这时无需等待 run() 方法体代码执行完毕，可以直接继续执行下面的代码；这时此线程是处于就绪状态，并没有运行。然后通过此 Thread 类调用方法 run() 来完成其运行状态，这里方法 run() 称为线程体，它包含了要执行的这个线程的内容，run() 方法运行结束，此线程终止。然后 cpu 再调度其它线程；
3.run() 方法是在本线程里的，只是线程里的一个函数，而不是多线程的。如果直接调用 run()，其实就相当于是调用了一个普通函数而已，直接待用 run() 方法必须等待 run() 方法执行完毕才能执行下面的代码，所以执行路径还是只有一条，根本就没有线程的特征，所以在多线程执行时要使用 start() 方法而不是 run() 方法。

9、在 Java 程序中怎么保证多线程的运行安全？
线程安全在三个方面体现：
原子性：提供互斥访问，同一时刻只能有一个线程对数据进行操作，（atomic，synchronized）；
可见性：一个线程对主内存的修改可以及时地被其他线程看到，（synchronized、volatile）；
有序性：一个线程观察其他线程中的指令执行顺序，由于指令重排序，该观察结果一般杂乱无序，（happens-before 原则）。
10、Java 线程同步的几种方法？
1. 使用 Synchronized 关键字；
2.wait 和 notify；
3.使用特殊域变量 volatile 实现线程同步；
4.使用可重入锁实现线程同步；
5.使用阻塞队列实现线程同步；
6. 使用信号量 Semaphore。

11、Thread.interrupt() 方法的工作原理是什么？
	在 Java 中，线程的中断 interrupt 只是改变了线程的中断状态，至于这个中断状态改变后带来的结果，那是无法确定的，有时它更是让停止中的线程继续执行的唯一手段。不但不是让线程停止运行，反而是继续执行线程的手段。
在一个线程对象上调用 interrupt() 方法，真正有影响的是 wait、join、sleep 方法，当然这 3 个方法包括它们的重载方法。请注意：上面这三个方法都会抛出 InterruptedException。
1.对于 wait 中的等待 notify、notifyAll 唤醒的线程，其实这个线程已经“暂停”执行，因为它正在某一对象的休息室中，这时如果它的中断状态被改变，那么它就会抛出异常。这个 InterruptedException 异常不是线程抛出的，而是 wait 方法，也就是对象的 wait 方法内部会不断检查在此对象上休息的线程的状态，如果发现哪个线程的状态被置为已中断，则会抛出 InterruptedException，意思就是这个线程不能再等待了，其意义就等同于唤醒它了，然后执行 catch 中的代码。
2.对于 sleep 中的线程，如果你调用了 Thread.sleep(一年)；现在你后悔了，想让它早些醒过来，调用 interrupt() 方法就是唯一手段，只有改变它的中断状态，让它从 sleep 中将控制权转到处理异常的 catch 语句中，然后再由 catch 中的处理转换到正常的逻辑。同样，对于 join 中的线程你也可以这样处理。
12、谈谈对 ThreadLocal 的理解？
1. Java 的 Web 项目大部分都是基于 Tomcat。每次访问都是一个新的线程，每一个线程都独享一个 ThreadLocal，我们可以在接收请求的时候 set 特定内容，在需要的时候 get 这个值。
2. ThreadLocal 提供 get 和 set 方法，为每一个使用这个变量的线程都保存有一份独立的副本。
1. get() 方法是用来获取 ThreadLocal 在当前线程中保存的变量副本；
2. set() 用来设置当前线程中变量的副本；
3. remove() 用来移除当前线程中变量的副本；
4. initialValue() 是一个 protected 方法，一般是用来在使用时进行重写的，如果在没有 set 的时候就调用 get，会调用 initialValue 方法初始化内容。
13、在哪些场景下会使用到 ThreadLocal？
在调用 API 接口的时候传递了一些公共参数，这些公共参数携带了一些设备信息（是安卓还是 ios），服务端接口根据不同的信息组装不同的格式数据返回给客户端。假定服务器端需要通过设备类型（device）来下发下载地址，当然接口也有同样的其他逻辑，我们只要在返回数据的时候判断好是什么类型的客户端就好了。上面这种场景就可以将传进来的参数 device 设置到 ThreadLocal 中。用的时候取出来就行。避免了参数的层层传递。
14、说一说自己对于 synchronized 关键字的了解？
synchronized关键字解决的是多个线程之间访问资源的同步性，synchronized 关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。
另外，在 Java 早期版本中，synchronized 属于重量级锁，效率低下，因为监视器锁（monitor）是依赖于底层的操作系统的 Mutex Lock 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是为什么早期的 synchronized 效率低的原因。庆幸的是在 JDK6 之后 Java 官方对从 JVM 层面对synchronized 较大优化，所以现在的 synchronized 锁效率也优化得很不错了。JDK6 对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。
15、讲一下 synchronized 关键字的底层原理？
synchronized 关键字底层原理属于 JVM 层面。
synchronized 同步语句块的情况

通过 JDK 自带的 javap 命令查看 SynchronizedDemo 类的相关字节码信息：首先切换到类的对应目录执行 javac SynchronizedDemo.java 命令生成编译后的 .class 文件，然后执行 javap -c -s -v -l SynchronizedDemo.class。



从上面我们可以看出：synchronized 同步语句块的实现使用的是 monitorenter 和 monitorexit 指令，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指明同步代码块的结束位置。
当执行 monitorenter 指令时，线程试图获取锁也就是获取 monitor的持有权。monitor 对象存在于每个 Java 对象的对象头中，synchronized 锁便是通过这种方式获取锁的，也是为什么 Java 中任意对象可以作为锁的原因。当计数器为 0 则可以成功获取，获取后将锁计数器设为 1 也就是加 1。相应的在执行 monitorexit 指令后，将锁计数器设为 0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。

synchronized 修饰方法的的情况


synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。
16、如何在项目中使用 synchronized 的？
synchronized 关键字最主要的三种使用方式：
1. 修饰实例方法：作用于当前对象实例加锁，进入同步代码前要获得当前对象实例的锁；
2. 修饰静态方法：作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁 。也就是给当前类加锁，会作用于类的所有对象实例，因为静态成员不属于任何一个实例对象，是类成员（static 表明这是该类的一个静态资源，不管 new了多少个对象，只有一份，所以对该类的所有对象都加了锁）。所以如果一个线程 A 调用一个实例对象的非静态 synchronized 方法，而线程 B 需要调用这个实例对象所属类的静态 synchronized 方法，是允许的，不会发生互斥现象，因为访问静态 synchronized 方法占用的锁是当前类的锁，而访问非静态 synchronized 方法占用的锁是当前实例对象锁；
3. 修饰代码块：指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。和 synchronized 方法一样，synchronized(this) 代码块也是锁定当前对象的。synchronized 关键字加到 static 静态方法和 synchronized(class) 代码块上都是是给 Class 类上锁。这里再提一下：synchronized 关键字加到非 static 静态方法上是给对象实例上锁。另外需要注意的是：尽量不要使用 synchronized(String a) 因为 JVM 中，字符串常量池具有缓冲功能。
补充：双重校验锁实现单例模式
问到 synchronized 的使用，很有可能让你用 synchronized 实现个单例模式。这里补充下使用 synchronized 双重校验锁的方法实现单例模式：

另外，需要注意 uniqueInstance 采用 volatile 关键字修饰也是很有必要。采用 volatile 关键字修饰也是很有必要的， uniqueInstance = new Singleton(); 这段代码其实是分为三步执行：
1.为 uniqueInstance 分配内存空间
2.初始化 uniqueInstance
3.将 uniqueInstance 指向分配的内存地址
但是由于 JVM 具有指令重排的特性，执行顺序有可能变成 1 -> 3 -> 2。指令重排在单线程环境下不会出现问题，但是在多线程环境下会导致一个线程获得还没有初始化的实例。例如，线程 T1 执行了 1 和 3，此时 T2 调用 getUniqueInstance() 后发现 uniqueInstance 不为空，因此返回 uniqueInstance，但此时 uniqueInstance 还未被初始化。

使用 volatile 可以禁止 JVM 的指令重排，保证在多线程环境下也能正常运行。
17、说说 JDK1.6 之后的 synchronized 关键字底层做了哪些优化，可以详细介绍一下这些优化吗？
说明：这道题答案有点长，但是回答的详细面试会很加分。
JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。
锁主要存在四种状态，依次是：无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态，它们会随着竞争的激烈而逐渐升级。注意锁可以升级不可降级，这种策略是为了提高获得锁和释放锁的效率。
偏向锁
引入偏向锁的目的和引入轻量级锁的目的很像，它们都是为了没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗。但是不同是：轻量级锁在无竞争的情况下使用 CAS 操作去代替使用互斥量。而偏向锁在无竞争的情况下会把整个同步都消除掉。
偏向锁的“偏”就是偏心的偏，它的意思是会偏向于第一个获得它的线程，如果在接下来的执行中，该锁没有被其他线程获取，那么持有偏向锁的线程就不需要进行同步。
但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，因此这种场合下不应该使用偏向锁，否则会得不偿失，需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁。
轻量级锁
倘若偏向锁失败，虚拟机并不会立即升级为重量级锁，它还会尝试使用一种称为轻量级锁的优化手段(JDK1.6 之后加入的)。轻量级锁不是为了代替重量级锁，它的本意是在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗，因为使用轻量级锁时，不需要申请互斥量。另外，轻量级锁的加锁和解锁都用到了 CAS 操作。
轻量级锁能够提升程序同步性能的依据是“对于绝大部分锁，在整个同步周期内都是不存在竞争的”，这是一个经验数据。如果没有竞争，轻量级锁使用 CAS 操作避免了使用互斥操作的开销。但如果存在锁竞争，除了互斥量开销外，还会额外发生 CAS 操作，因此在有锁竞争的情况下，轻量级锁比传统的重量级锁更慢！如果锁竞争激烈，那么轻量级将很快膨胀为重量级锁！
自旋锁和自适应自旋
轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。
互斥同步对性能最大的影响就是阻塞的实现，因为挂起线程/恢复线程的操作都需要转入内核态中完成（用户态转换到内核态会耗费时间）。
一般线程持有锁的时间都不是太长，所以仅仅为了这一点时间去挂起线程/恢复线程是得不偿失的。所以，虚拟机的开发团队就这样去考虑：“我们能不能让后面来的请求获取锁的线程等待一会而不被挂起呢？看看持有锁的线程是否很快就会释放锁”。为了让一个线程等待，我们只需要让线程执行一个忙循环（自旋），这项技术就叫做自旋。
百度百科对自旋锁的解释：
何谓自旋锁？它是为实现保护共享资源而提出一种锁机制。其实，自旋锁与互斥锁比较类似，它们都是为了解决对某项资源的互斥使用。无论是互斥锁，还是自旋锁，在任何时刻，最多只能有一个保持者，也就说，在任何时刻最多只能有一个执行单元获得锁。但是两者在调度机制上略有不同。对于互斥锁，如果资源已经被占用，资源申请者只能进入睡眠状态。但是自旋锁不会引起调用者睡眠，如果自旋锁已经被别的执行单元保持，调用者就一直循环在那里看是否该自旋锁的保持者已经释放了锁，"自旋"一词就是因此而得名。
自旋锁在 JDK1.6 之前其实就已经引入了，不过是默认关闭的，需要通过 --XX:+UseSpinning 参数来开启。JDK1.6 及 1.6 之后，就改为默认开启的了。需要注意的是：自旋等待不能完全替代阻塞，因为它还是要占用处理器时间。如果锁被占用的时间短，那么效果当然就很好了。反之，自旋等待的时间必须要有限度。如果自旋超过了限定次数任然没有获得锁，就应该挂起线程。自旋次数的默认值是 10 次，用户可以修改 --XX:PreBlockSpin 来更改。另外，在 JDK1.6 中引入了自适应的自旋锁。自适应的自旋锁带来的改进就是：自旋的时间不在固定了，而是和前一次同一个锁上的自旋时间以及锁的拥有者的状态来决定，虚拟机变得越来越“聪明”了。
锁消除
锁消除理解起来很简单，它指的就是虚拟机即使编译器在运行时，如果检测到那些共享数据不可能存在竞争，那么就执行锁消除。锁消除可以节省毫无意义的请求锁的时间。
锁粗化
原则上，我们在编写代码的时候，总是推荐将同步块的作用范围限制得尽量小。只在共享数据的实际作用域才进行同步，这样是为了使得需要同步的操作数量尽可能变小，如果存在锁竞争，那等待线程也能尽快拿到锁。大部分情况下，上面的原则都是没有问题的，但是如果一系列的连续操作都对同一个对象反复加锁和解锁，那么会带来很多不必要的性能消耗。
18、谈谈 synchronized 和 ReenTrantLock 的区别？
1. synchronized 是和 for、while 一样的关键字，ReentrantLock 是类，这是二者的本质区别。既然 ReentrantLock 是类，那么它就提供了比 synchronized 更多更灵活的特性：等待可中断、可实现公平锁、可实现选择性通知（锁可以绑定多个条件）、性能已不是选择标准。
2. synchronized 依赖于 JVM 而 ReenTrantLock 依赖于 API。synchronized 是依赖于 JVM 实现的，JDK1.6 为 synchronized 关键字进行了很多优化，但是这些优化都是在虚拟机层面实现的，并没有直接暴露给我们。ReenTrantLock 是 JDK 层面实现的（也就是 API 层面，需要 lock() 和 unlock 方法配合 try/finally 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的。
19、synchronized 和 volatile 的区别是什么？
1. volatile 本质是在告诉 JVM当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized 则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
2. volatile 仅能使用在变量级别；synchronized 则可以使用在变量、方法、和类级别的。
3. volatile 仅能实现变量的修改可见性，不能保证原子性；而 synchronized 则可以保证变量的修改可见性和原子性。
4. volatile 不会造成线程的阻塞；synchronized 可能会造成线程的阻塞。
5. volatile 标记的变量不会被编译器优化；synchronized 标记的变量可以被编译器优化。
20、谈一下你对 volatile 关键字的理解？
volatile 关键字是用来保证有序性和可见性的。这跟 Java 内存模型有关。我们所写的代码，不一定是按照我们自己书写的顺序来执行的，编译器会做重排序，CPU 也会做重排序的，这样做是为了减少流水线阻塞，提高 CPU 的执行效率。这就需要有一定的顺序和规则来保证，不然程序员自己写的代码都不知道对不对了，所以有 happens-before 规则，其中有条就是 volatile 变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作、有序性实现的是通过插入内存屏障来保证的。
被 volatile 修饰的共享变量，就具有了以下两点特性：
1 . 保证了不同线程对该变量操作的内存可见性;
2 . 禁止指令重排序。


21、说下对 ReentrantReadWriteLock 的理解？
ReentrantReadWriteLock 允许多个读线程同时访问，但是不允许写线程和读线程、写线程和写线程同时访问。读写锁内部维护了两个锁：一个是用于读操作的 ReadLock，一个是用于写操作的 WriteLock。读写锁 ReentrantReadWriteLock 可以保证多个线程可以同时读，所以在读操作远大于写操作的时候，读写锁就非常有用了。
ReentrantReadWriteLock 基于 AQS 实现，它的自定义同步器（继承 AQS）需要在同步状态 state 上维护多个读线程和一个写线程，该状态的设计成为实现读写锁的关键。ReentrantReadWriteLock 很好的利用了高低位。来实现一个整型控制两种状态的功能，读写锁将变量切分成了两个部分，高 16 位表示读，低 16 位表示写。
ReentrantReadWriteLock 的特点：
1.写锁可以降级为读锁，但是读锁不能升级为写锁；
2.不管是 ReadLock 还是 WriteLock 都支持 Interrupt，语义与 ReentrantLock 一致；
3.WriteLock 支持 Condition 并且与 ReentrantLock 语义一致，而 ReadLock 则不能使用 Condition，否则抛出 UnsupportedOperationException 异常；
4. 默认构造方法为非公平模式 ，开发者也可以通过指定 fair 为 true 设置为公平模式 。
升降级
1.读锁里面加写锁，会导致死锁；
2.写锁里面是可以加读锁的，这就是锁的降级。


22、说下对悲观锁和乐观锁的理解？
悲观锁
总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁（共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程）。传统的关系型数据库里边就用到了很多这种锁机制，比如：行锁、表锁、读锁、写锁等，都是在做操作之前先上锁。Java 中 synchronized 和 ReentrantLock 等独占锁就是悲观锁思想的实现。
乐观锁
总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制和 CAS 算法实现。乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库提供的类似于 write_condition 机制，其实都是提供的乐观锁。在 Java 中 java.util.concurrent.atomic 包下面的原子变量类就是使用了乐观锁的一种实现方式 CAS 实现的。
两种锁的使用场景
从上面对两种锁的介绍，我们知道两种锁各有优缺点，不可认为一种好于另一种，像乐观锁适用于写比较少的情况下（多读场景），即冲突真的很少发生的时候，这样可以省去了锁的开销，加大了系统的整个吞吐量。但如果是多写的情况，一般会经常产生冲突，这就会导致上层应用会不断的进行 retry，这样反倒是降低了性能，所以一般多写的场景下用悲观锁就比较合适。
23、乐观锁常见的两种实现方式是什么？
乐观锁一般会使用版本号机制或者 CAS 算法实现。
版本号机制
    一般是在数据表中加上一个数据版本号 version 字段，表示数据被修改的次数，当数据被修改时，version 值会加 1。当线程 A 要更新数据值时，在读取数据的同时也会读取 version 值，在提交更新时，若刚才读取到的 version 值为当前数据库中的 version 值相等时才更新，否则重试更新操作，直到更新成功。
CAS 算法
即 compare and swap（比较与交换），是一种有名的无锁算法。无锁编程，即不使用锁的情况下实现多线程之间的变量同步，也就是在没有线程被阻塞的情况下实现变量的同步，所以也叫非阻塞同步（Non-blocking Synchronization）。CAS 算法涉及到三个操作数：
1、需要读写的内存值 V
		 2、进行比较的值 A
3、拟写入的新值 B
当且仅当 V 的值等于 A 时，CAS 通过原子方式用新值 B 来更新 V 的值，否则不会执行任何操作（比较和替换是一个原子操作）。一般情况下是一个自旋操作，即不断的重试。
24、乐观锁的缺点有哪些？
1. ABA 问题
如果一个变量 V 初次读取的时候是 A 值，并且在准备赋值的时候检查到它仍然是 A 值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回 A，那 CAS 操作就会误认为它从来没有被修改过。这个问题被称为 CAS 操作的  "ABA" 问题。    
JDK 1.5 以后的AtomicStampedReference 类就提供了此种能力，其中的 compareAndSet 方法就是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。
2. 循环时间长开销大
自旋 CAS（也就是不成功就一直循环执行直到成功）如果长时间不成功，会给 CPU 带来非常大的执行开销。如果 JVM 能支持处理器提供的 pause 指令那么效率会有一定的提升，pause 指令有两个作用，第一：它可以延迟流水线执行指令（de-pipeline），使 CPU 不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二：它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起 CPU 流水线被清空（CPU pipeline flush），从而提高 CPU 的执行效率。
3. 只能保证一个共享变量的原子操作
CAS 只对单个共享变量有效，当操作涉及跨多个共享变量时 CAS 无效。 但是从 JDK 1.5 开始，提供了 AtomicReference 类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行 CAS 操作。所以我们可以使用锁或者利用 AtomicReference 类把多个共享变量合并成一个共享变量来操作。
25、CAS 和 synchronized 的使用场景？
简单的来说 CAS 适用于写比较少的情况下（多读场景，冲突一般较少），synchronized 适用于写比较多的情况下（多写场景，冲突一般较多）。
1.对于资源竞争较少（线程冲突较轻）的情况，使用 synchronized 同步锁进行线程阻塞和唤醒切换以及用户态内核态间的切换操作额外浪费消耗 cpu 资源；而 CAS 基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少，因此可以获得更高的性能。
2. 对于资源竞争严重（线程冲突严重）的情况，CAS 自旋的概率会比较大，从而浪费更多的 CPU 资源，效率低于 synchronized。
26、简单说下对 Java 中的原子类的理解？
这里 Atomic 是指一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。所以，所谓原子类说简单点就是具有原子操作特征的类。并发包 java.util.concurrent 的原子类都存放在 java.util.concurrent.atomic 下。根据操作的数据类型，可以将 JUC 包中的原子类分为 4 类：
1. 基本类型
使用原子的方式更新基本类型：
AtomicInteger：整型原子类
AtomicLong：长整型原子类
AtomicBoolean ：布尔型原子类
2. 数组类型
使用原子的方式更新数组里的某个元素：
AtomicIntegerArray：整型数组原子类
AtomicLongArray：长整型数组原子类
AtomicReferenceArray ：引用类型数组原子类
3. 引用类型
AtomicReference：引用类型原子类
AtomicStampedReference：原子更新引用类型里的字段原子类AtomicMarkableReference ：原子更新带有标记位的引用类型
4. 对象的属性修改类型
AtomicIntegerFieldUpdater：原子更新整型字段的更新器
AtomicLongFieldUpdater：原子更新长整型字段的更新器AtomicStampedReference ：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号，可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。

27、atomic 的原理是什么？
Atomic 包中的类基本的特性就是在多线程环境下，当有多个线程同时对单个（包括基本类型及引用类型）变量进行操作时，具有排他性，即当多个线程同时对该变量的值进行更新时，仅有一个线程能成功，而未成功的线程可以向自旋锁一样，继续尝试，一直等到执行成功。
Atomic 系列的类中的核心方法都会调用 unsafe 类中的几个本地方法。我们需要先知道一个东西就是 Unsafe 类，全名为：sun.misc.Unsafe，这个类包含了大量的对 C 代码的操作，包括很多直接内存分配以及原子操作的调用，而它之所以标记为非安全的，是告诉你这个里面大量的方法调用都会存在安全隐患，需要小心使用，否则会导致严重的后果，例如在通过 unsafe 分配内存的时候，如果自己指定某些区域可能会导致一些类似 C++ 一样的指针越界到其他进程的问题。
28、说下对同步器 AQS 的理解？
AQS 的全称为：AbstractQueuedSynchronizer，这个类在 java.util.concurrent.locks 包下面。AQS 是一个用来构建锁和同步器的框架，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如：我们提到的 ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask 等等皆是基于 AQS 的。当然，我们自己也能利用 AQS 非常轻松容易地构造出符合我们自己需求的同步器。

29、AQS 的原理是什么？
AQS 核心思想是：如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

CLH队列：
CLH(Craig, Landin, and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点（Node）来实现锁的分配。
AQS 使用一个 int 成员变量 (state) 来表示同步状态，通过内置的 FIFO 队列来完成获取资源线程的排队工作。AQS 使用 CAS 对该同步状态进行原子操作实现对其值的修改。


30.AQS 对资源的共享模式有哪些？
1. Exclusive（独占）：只有一个线程能执行，如：ReentrantLock，又可分为公平锁和非公平锁：
2. Share（共享）：多个线程可同时执行，如：CountDownLatch、Semaphore、CountDownLatch、 CyclicBarrier、ReadWriteLock。


31、AQS 底层使用了模板方法模式，你能说出几个需要重写的方法吗？
使用者继承 AbstractQueuedSynchronizer 并重写指定的方法。将 AQS 组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。
1.isHeldExclusively() ：该线程是否正在独占资源。只有用到 condition 才需要去实现它。
2.tryAcquire(int) ：独占方式。尝试获取资源，成功则返回 true，失败则返回 false。
3.tryRelease(int) ：独占方式。尝试释放资源，成功则返回 true，失败则返回 false。
4.tryAcquireShared(int) ：共享方式。尝试获取资源。负数表示失败；0 表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。5. tryReleaseShared(int) ：共享方式。尝试释放资源，成功则返回 true，失败则返回 false。

32、说下对信号量 Semaphore 的理解？
synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore (信号量)可以指定多个线程同时访问某个资源。
执行 acquire 方法阻塞，直到有一个许可证可以获得然后拿走一个许可证；每个 release 方法增加一个许可证，这可能会释放一个阻塞的 acquire 方法。然而，其实并没有实际的许可证这个对象，Semaphore 只是维持了一个可获得许可证的数量。Semaphore 经常用于限制获取某种资源的线程数量。当然一次也可以一次拿取和释放多个许可证，不过一般没有必要这样做。除了 acquire方法（阻塞）之外，另一个比较常用的与之对应的方法是 tryAcquire 方法，该方法如果获取不到许可就立即返回 false。
33、CountDownLatch 和 CyclicBarrier 有什么区别？
CountDownLatch 是计数器，只能使用一次，而 CyclicBarrier 的计数器提供 reset 功能，可以多次使用。
对于 CountDownLatch 来说，重点是“一个线程（多个线程）等待”，而其他的 N 个线程在完成“某件事情”之后，可以终止，也可以等待。而对于 CyclicBarrier，重点是多个线程，在任意一个线程没有完成，所有的线程都必须等待。
CountDownLatch 是计数器，线程完成一个记录一个，只不过计数不是递增而是递减，而 CyclicBarrier 更像是一个阀门，需要所有线程都到达，阀门才能打开，然后继续执行。
应用场景：
CountDownLatch 应用场景：
1.某一线程在开始运行前等待 n 个线程执行完毕：启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。
2.实现多个线程开始执行任务的最大并行性。注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。
3.死锁检测：一个非常方便的使用场景是，你可以使用 n 个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。
CyclicBarrier  应用场景：
CyclicBarrier 可以用于多线程计算数据，最后合并计算结果的应用场景。比如：我们用一个 Excel 保存了用户所有银行流水，每个 Sheet 保存一个帐户近一年的每笔银行流水，现在需要统计用户的日均银行流水，先用多线程处理每个 sheet 里的银行流水，都执行完之后，得到每个 sheet 的日均银行流水，最后，再用 barrierAction 用这些线程的计算结果，计算出整个 Excel 的日均银行流水。
34、说下对线程池的理解？为什么要使用线程池？
线程池提供了一种限制和管理资源（包括执行一个任务）的方式。每个线程池还维护一些基本统计信息，例如：已完成任务的数量。

使用线程池的好处

1. 降低资源消耗：通过重复利用已创建的线程降低线程创建和销毁造成的消耗;
2. 提高响应速度：当任务到达时，任务可以不需要的等到线程创建就能立即执行;
3. 提高线程的可管理性：线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。
35、创建线程池的参数有哪些？
1. corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，如果当前 poolSize < corePoolSize 时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，等到需要执行的任务数大于线程池基本大小时就不再创建。如果调用了线程池的prestartAllCoreThreads() 方法，线程池会提前创建并启动所有基本线程。
2. maximumPoolSize（线程池最大数量）：线程池允许创建的最大线程数。如果队列满了，并且已创建的线程数小于最大线程数，则线程池会再创建新的线程执行任务。值得注意的是，如果使用了无界的任务队列这个参数就没什么效果。
3. keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。所以，如果任务很多，并且每个任务执行的时间比较短，可以调大时间，提高线程的利用率。
4. TimeUnit（线程活动保持时间的单位）：可选的单位有天（DAYS）、小时（HOURS）、分钟（MINUTES）、毫秒（MILLISECONDS）、微秒（MICROSECONDS，千分之一毫秒）和纳秒（NANOSECONDS，千分之一微秒）。
5. workQueue（任务队列）：用于保存等待执行的任务的阻塞队列。
可以选择以下几个阻塞队列：

1.ArrayBlockingQueue：是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。
2. LinkedBlockingQueue：一个基于链表结构的阻塞队列，此队列按 FIFO 排序元素，吞吐量通常要高于 ArrayBlockingQueue。静态工厂方法 Executors.newFixedThreadPool() 使用了这个队列。
3. SynchronousQueue：一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于 LinkedBlockingQueue，静态工厂方法 Executors.newCachedThreadPool 使用了这个队列。
4.PriorityBlockingQueue：一个具有优先级的无限阻塞队列。
6. threadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。
7. RejectExecutionHandler（饱和策略）：队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是 AbortPolicy，表示无法处理新任务时抛出异常。
饱和策略：
在 JDK1.5 中 Java 线程池框架提供了以下 4 种策略：
1. AbortPolicy：直接抛出异常。
2.CallerRunsPolicy：只用调用者所在线程来运行任务。
3. DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。4. DiscardPolicy：不处理，丢弃掉。当然，也可以根据应用场景需要来实现RejectedExecutionHandler 接口自定义策略。如记录日志或持久化存储不能处理的任务。
36、如何创建线程池？
方式一：通过 ThreadPoolExecutor 的构造方法实现：

方式二：通过 Executor 框架的工具类 Executors 来实现：
我们可以创建三种类型的 ThreadPoolExecutor：
1.FixedThreadPool：该方法返回一个固定线程数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务。
2. SingleThreadExecutor：方法返回一个只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先进先出的顺序执行队列中的任务。
3. CachedThreadPool：该方法返回一个可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。
注意：
阿里巴巴Java开发手册》中强制线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
Executors 创建线程池对象的弊端如下：
FixedThreadPool 和 SingleThreadExecutor ：允许请求的队列长度为 Integer.MAX_VALUE，可能堆积大量的请求，从而导致 OOM。CachedThreadPool 和 ScheduledThreadPool ： 允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建大量线程，从而导致 OOM。

37、线程池中的的线程数一般怎么设置？需要考虑哪些问题？
主要考虑下面几个方面：
1. 线程池中线程执行任务的性质：
计算密集型的任务比较占 cpu，所以一般线程数设置的大小 等于或者略微大于 cpu 的核数；但 IO 型任务主要时间消耗在 IO 等待上，cpu 压力并不大，所以线程数一般设置较大。
2. cpu 使用率：
当线程数设置较大时，会有如下几个问题：第一，线程的初始化，切换，销毁等操作会消耗不小的 cpu 资源，使得 cpu 利用率一直维持在较高水平。第二，线程数较大时，任务会短时间迅速执行，任务的集中执行也会给 cpu 造成较大的压力。第三， 任务的集中支持，会让 cpu 的使用率呈现锯齿状，即短时间内 cpu 飙高，然后迅速下降至闲置状态，cpu 使用的不合理，应该减小线程数，让任务在队列等待，使得 cpu 的使用率应该持续稳定在一个合理，平均的数值范围。所以 cpu 在够用时，不宜过大，不是越大越好。可以通过上线后，观察机器的 cpu 使用率和 cpu 负载两个参数来判断线程数是否合理。
3. 内存使用率：
线程数过多和队列的大小都会影响此项数据，队列的大小应该通过前期计算线程池任务的条数，来合理的设置队列的大小，不宜过小，让其不会溢出，因为溢出会走拒绝策略，多少会影响性能，也会增加复杂度。
4. 下游系统抗并发能力：
多线程给下游系统造成的并发等于你设置的线程数，例如：如果是多线程访问数据库，你就考虑数据库的连接池大小设置，数据库并发太多影响其  QPS，会把数据库打挂等问题。如果访问的是下游系统的接口，你就得考虑下游系统是否能抗的住这么多并发量，不能把下游系统打挂了。

38、执行 execute() 方法和 submit() 方法的区别是什么呢？
1. execute() 方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否；
2.submit() 方法用于提交需要返回值的任务。线程池会返回一个 Future 类型的对象，通过这个 Future 对象可以判断任务是否执行成功，并且可以通过 Future 的 get() 方法来获取返回值，get() 方法会阻塞当前线程直到任务完成，而使用 get(long timeout，TimeUnit unit) 方法则会阻塞当前线程一段时间后立即返回，这时候有可能任务没有执行完。

39、说下对 Fork/Join 并行计算框架的理解？
Fork/Join 并行计算框架主要解决的是分治任务。分治的核心思想是“分而治之”：将一个大的任务拆分成小的子任务的结果聚合起来从而得到最终结果。
Fork/Join 并行计算框架的核心组件是 ForkJoinPool。ForkJoinPool 支持任务窃取机制，能够让所有的线程的工作量基本均衡，不会出现有的线程很忙，而有的线程很闲的情况，所以性能很好。
ForkJoinPool 中的任务队列采用的是双端队列，工作线程正常获取任务和“窃取任务”分别是从任务队列不同的端消费，这样能避免很多不必要的数据竞争。

40、JDK 中提供了哪些并发容器？
JDK 提供的这些容器大部分在 java.util.concurrent 包中。
1. ConcurrentHashMap：线程安全的 HashMap；
2. CopyOnWriteArrayList：线程安全的 List，在读多写少的场合性能非常好，远远好于 Vector；
3. ConcurrentLinkedQueue：高效的并发队列，使用链表实现。可以看做一个线程安全的 LinkedList，这是一个非阻塞队列；
4. BlockingQueue：这是一个接口，JDK 内部通过链表、数组等方式实现了这个接口。表示阻塞队列，非常适合用于作为数据共享的通道；
5. ConcurrentSkipListMap：跳表的实现。这是一个 Map，使用跳表的数据结构进行快速查找。
41、谈谈对 CopyOnWriteArrayList 的理解？
在很多应用场景中，读操作可能会远远大于写操作。由于读操作根本不会修改原有的数据，因此对于每次读取都进行加锁其实是一种资源浪费。我们应该允许多个线程同时访问 List 的内部数据，毕竟读取操作是安全的。
CopyOnWriteArrayList 类的所有可变操作（add，set 等等）都是通过创建底层数组的新副本来实现的。当 List 需要被修改的时候，我们并不需要修改原有内容，而是对原有数据进行一次复制，将修改的内容写入副本。写完之后，再将修改完的副本替换原来的数据，这样就可以保证写操作不会影响读操作了。
从 CopyOnWriteArrayList 的名字就能看出 CopyOnWriteArrayList 是满足 CopyOnWrite 的 ArrayList，所谓 CopyOnWrite 也就是说：在计算机，如果你想要对一块内存进行修改时，我们不在原有内存块中进行写操作，而是将内存拷贝一份，在新的内存中进行写操作，写完之后，就将指向原来内存指针指向新的内存，原来的内存就可以被回收掉了。
CopyOnWriteArrayList 读取操作没有任何同步控制和锁操作，理由就是内部数组 array 不会发生修改，只会被另外一个 array 替换，因此可以保证数据安全。
CopyOnWriteArrayList 写入操作 add() 方法在添加集合的时候加了锁，保证了同步，避免了多线程写的时候会 copy 出多个副本出来。
42、谈谈对 BlockingQueue 的理解？分别有哪些实现类？
阻塞队列（BlockingQueue）被广泛使用在“生产者-消费者”问题中，其原因是 BlockingQueue 提供了可阻塞的插入和移除的方法。当队列容器已满，生产者线程会被阻塞，直到队列未满；当队列容器为空时，消费者线程会被阻塞，直至队列非空时为止。
BlockingQueue 是一个接口，继承自 Queue，所以其实现类也可以作为 Queue 的实现来使用，而 Queue 又继承自 Collection 接口。下面是 BlockingQueue 的相关实现类：

43、谈谈对 ConcurrentSkipListMap 的理解？
对于一个单链表，即使链表是有序的，如果我们想要在其中查找某个数据，也只能从头到尾遍历链表，这样效率自然就会很低，跳表就不一样了。跳表是一种可以用来快速查找的数据结构，有点类似于平衡树。它们都可以对元素进行快速的查找。
但一个重要的区别是：对平衡树的插入和删除往往很可能导致平衡树进行一次全局的调整。而对跳表的插入和删除只需要对整个数据结构的局部进行操作即可。这样带来的好处是：在高并发的情况下，你会需要一个全局锁来保证整个平衡树的线程安全。而对于跳表，你只需要部分锁即可。这样，在高并发环境下，你就可以拥有更好的性能。而就查询的性能而言，跳表的时间复杂度也是 O(logn) 。跳表的本质是同时维护了多个链表，并且链表是分层的。
