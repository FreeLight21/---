### 01、线程池

1. 线程池的好处：-》池化技术
   * 减少资源的浪费，可以重复利用已有的线程，不用频繁的创建线程和销毁线程造成消耗
   * 提高响应速度，当任务到达时，任务不用等候线程的创建，就可以立即执行
   * 提高客观理性：线程是有限的资源，频繁的创建线程会影响到系统的稳定性，线程池可以分配，管理，调优，监控线程。

2. Executor框架

   * 比Thread.start()启动线程更方便，执行任务需实现Runnable 和 Callable接口

   * ThreadPoolExecutor, ScheduledThreadExcutor-->ExecutorService->Executor(核心接口)

   * Future接口或者Future接口的实现类FutureTask,可以返回任务执行的结果。

   * 阿里巴巴推荐使用<mark>ThreadPoolExecutor</mark>

     ```java
     ThreadPoolExecutor executor = new ThreadPoolExecutor();
     1.CorePoolSize,核心线程池数量
     2.MaxmumSize,最大线程池数量，核心线程已经满了， 任务队列也装不下任务了，此时就会使用最大线程池数量。
     3.keepAliveTime, 当线程数大于核心线程数的时候，此时没有任务提交，核心线程之外的空闲线程在keepAlive时间之后才会被销毁。
     4.ThreadFactory, 默认，用来创建线程
     5.RejectExecutionHandler拒绝策略，abortPolicy，抛出异常，RejectExecutorException,     
     ```

   * ThreadPoolExecutor的构造函数有好几个，这是最长的参数的一个构造器，其他都是它的变种。
   
3. <font color = green>创建线程的两种方式：</font>
   
   * ThreadPoolExecutor构造函数，推荐
   
   * 使用工具类Executors
   
     * FixedThreadPool
     * singlePoolExecutor
     * cachedThreadPool
   
     >这三种线程池的底层都是使用的ThreadPoolExecutor创建而成，FixedThreadPool,以及singlePoolExecutor的workingQueue使用的是无界队列，Intger.MAX_VALUE,会造成OOM
   
   ### 02、 CAS(CompareAndSwap)
   
   1. 无锁机制，首先从内存区中位置获得值的大小，和我们预期expect的值比较，如果没有发生了改变，就update更新为新值，将新值写入到内存区当中。如果发生改变了，两者值不相等，就反复重试，通过这种机制保证原子性。
   2. 锁与共享变量
      * 乐观锁，认为线程操作共享变量的过程中，没有其他的线程会修改共享变量的值，对结果持乐观态度。CAS就是乐观锁，无锁，
      * 悲观锁， synchronized，同步机制，就是悲观的，认为线程冲突一定会发生，所以在同一时刻，只能允许一个线程占有资源。
   3. CAS无锁，violatile关键字，让共享变量可见，这样其他的线程也能看到共享变量的实时变化。
   4. CAS缺陷：ABA问题，将内存位置的值修改为B，然后再修改为A，整个过程，值没有变化，而实际上中间已经修改了一次，--》加上版本号：1A2B3A。
   
   
   
   
   

