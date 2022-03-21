## 八股文

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

   ### 03、Java虚拟机

   1. JDK->JRE->JVM三者的关系，前者包含后者

   2. java虚拟机的结构：classfiles-》classloader类加载子系统-》运行时数据区running data area-》执行引擎（解释器，即时编译器，垃圾回收器）-》本地方法接口

   3. 类加载器： 加载-链接-初始化三步，负责字节码的加载工作

      * 加载阶段：引导类加载，扩展类加载，系统类加载器
      * 链接阶段：验证verify，准备prepare，解析resolve

   4. 解释器、即时编译器JIT,

   5. 比较流行的三个虚拟机：hotspotVM, JRockit, J9, JRockit是号称世界上最快的虚拟机。

   6. 双亲委派机制：加载请求到达类加载器的时候，不会自己先去加载，首先向上委托给父类的类加载，父类加载器处理不了，再有子类加载器处理。系统类加载器向上委托-》扩展类加载器-》引导类加载器

   7. 栈帧：局部变量表，操作栈，方法返回地址，动态链接

   8. 堆： 新生代（eden,s0,s1）-》老年代,MetaSpace元数据区包含常量池，方法元信息，类信息

   9. 为什么要有双亲委派机制：当自定义一个String.class类，在这种机制下 bootstrap.classloader已经提前加载过这些类了，保护程序安全，防止核心 API被纂改。

   10. verify,验证：主要使字节码，文件格式，元数据验证-》prepare准备，为类变量分配内存空间，并默认初始化 。-》resolve解析，将符合引用变为直接引用的过程，<mark>符合引用：用一组符合来描述所引用的目标。 </mark>直接引用：直接指向目标的指针，定位到目标的句柄。

   11. 程序计数器：用来保存下一条字节码指令的地址，内存空间很小的一块地址，是运行速度最快的存储区域。CPU要来回的不断切换线程，切换回来之后，就是程序计数器来明确吓一跳要执行的指令。

   12. 虚拟机栈：本地变量表(方法的局部变量)、操作数栈，方法返回地址，链接地址；栈是不存在垃圾回收的。 虚拟机栈的大小是动态的，也可以是固定不变的。它的大小直接决定了函数调用的最大可达深度。

       >通过参数-Xss来调整最大栈空间

   13. 本地变量表，内部是一个index：0~n-1的数组，最小单位是slot, 32位以下的数据类型是占一个slot, 64位的数据类型占两个slot(例如：long、double); slot可以重复利用，当局部变量的过了变量作用域，栈为了节省内存空间，会重复利用这些过期局部变量的slot.

   14. 类变量初始有两次时机：1.linking，prepare阶段为类变量分配空间并赋默认值。2. init初始化阶段，为类变量赋予程序员定义的值。而局部变量是不存在系统初始化的，这意味着局部变量则必须人为的初始化。

   15. 操作数栈：用来保存计算过程中的中间结果。

   ![image-20200706094109629](D:/Study Document/Java Learning/Java Notes/JVM/LearningNotes/JVM/1_内存与垃圾回收篇/5_虚拟机栈/images/image-20200706094109629.png)

   bipush，压栈到操作数栈中，istore_1,将操作数栈顶部的元素15，存储到局部变量表中索引为1的位置，局部变量表索引为0的位置保存this的指针。

   16. 动态链接：包含的是指向运行时常量池的引用，它的内存地址。运行时常量池包含了符合引用和字面量，动态链接的作用就是将符合引号转换为调用方法的直接引用。

   17. 动态类型or静态类型语言：<mark>类型检查是在编译期还是运行期</mark>, 

   18. 链接：静态链接or动态链接

       * 静态链接：在编译期间就能够确定方法的版本，而且运行期间保持不变，这种情况下直接调用方法的符合引用转换为直接引用，对应的是早期绑定。
       * 动态链接：在编译期间被调用的方法无法确定下来，只能在运行期间，才可以将用方法的符合引用转换为直接引用，晚期绑定。

   19. 方法返回值：returnvalue, 保存的是调用了该方法B的上一级方法A的pc寄存器的值，将改地址交给执行引擎，就能执行A方法的下一条指令。<mark>当方法正常退出的时候，返回地址是调用者的pc寄存器的值；而通过异常退出的，返回地址是要通过异常表来确定</mark>

   20. 虚拟机栈：管理java方法、本地方法栈：用来管理本地方法的调用。

   21. 一个JVM实例只存在一个堆内存，且是共享的。-Xms（最小内存）-Xmx(最大内存)， 在这里可以划分出TLAB（Thread Local Allocation Buffer）线程私有的缓冲区。

   22. 堆：几乎java所有的对象的实例以及数组都会在运行时在这里划分。这里垃圾gc最主要的区域。

       >java7: 堆分为：新生代--》Eden ,Suriror(s0,s1)，老年代，永久代（方法区）
       >
       >java8: 堆分为：新生代，老年代，元空间
       >
       >为什么要将永久代变为元空间？
       >
       >* 为永久代设置空间大小很难确定。某些场景下，如果动态类加载过多，容易产生Perm区的oom.
       >* 对永久代调优时很困难的

   23. Young新生代：eden, from , to 的比例是8：1：1， 新生代：老年代的比例--》1：2； 

   24. 概括下堆中对象的分配过程：

       >新建对象首先Eden,当eden满之后，就会触发minor GC ,同时将eden园区中幸存的对象转移到s0区， 然后eden区就能接着存放新建的对象，如果再次触发垃圾回收，so区的幸存对象会被转移到s1区，接着再次进行垃圾回收，s1->s0区，当满足默认的15次GC转移之后，就会就如养老区，当养老区内存不足，就会触发Major GC，当剩余的空间仍然无法保存下对象，就会OOM异常

   25. eden满了之后会触发minor gc， 但是suriror满了之后，不会触发minor gc.

   26. 从内存模型的角度，而非垃圾回收的角度，可以将Eden区分为多个线程的TLAB，每一个线程一个TLAB。

   27. 如何判断对象是否死亡？

       * 可达性分析
       * 引用计数法：无法解决循环引用的问题，每个地方引用了，引用就加1，引用失效了就减1，直到等于0，没有引用存在了。

   28. 方法区保存的内容：类型信息，域信息，方法信息， JIT代码缓存，静态变量，运行时常量池（字符串常量池）

   29. 运行时常量池（方法引用，数量值，类引用，符合引用、字段引用），字符串常量池，静态变量（前两者在jdk7之后之后，搬到了堆中），JDK8,方法区变成了本地内存中的元空间。

   30. StringTable为什么要调整位置到堆中。永久代的回收效率不高，Full GC只有在老年代和永久代空间不足的时候才触发。而我们在开发过程中有大量的字符串被创建，回收效率低，会导致老年代内存不足，OOM。

   ### 04、框架

   1. Springboot相较于spring的优点：

   * 内嵌服务器
   * 自动starter依赖，简化构建配置
   * 自动配置spring和第三方共能
   * ...

   2. JDBC本质是一套接口：

   * 注册驱动 com.mysql.jdbc.driver
   * 获取连接 conn ,DriverManager.getConnection(url,user,password);
   * 获取数据库操作对象stat
   * 执行sql语句 executeQuery（DQL）, executeUpdate(DML)，返回数据库改变的受影响的记录条数
   * 处理查询结果集
   * 关闭资源， conn, stat

   ```java
   public void login(){
       Connection conn = null;
       Statement stat = null; 
       ResultSet rs = null; 
       try{
         Class.forName("com.mysql.jdbc.driver");
         conn = DriverManager.getConnection(url,user,password); 
   	  stat = conn.creatStatement();   
         string sql = "";
   	  rs = stat.executeQuery(sql);  
         while(rs.next()){
             
         }
       }catch(ClassNotFoundException e){
        	e.printStackTrace();   
       }finally{
           if(rs != null){
               try{
                   rs.close();
               }catch(){}
           }
           ..conn.. stat
       }
   }
   ```

4. <mark>sql注入：用户输入的信息中包含了sql语句的关键字，使原有的含义发生了改变。</mark> 例如：password = 'fsda' or '1' = '1'; or关键字，只要一边成立了，就成立了。根本原因是：关键字也参与了sql语句的编译过程。

5. 如何解决sql注入：让关键字不要参与sql的编译，prepareStatement-->让sql语句的框架先进行编译，然后再传值。ps.setString(1,username), select * from emp where username = ? and password = ?,第一个问号的下标是1。

6. JDBC是怎样提交事务的，默认机制是：每执行一条DML语句就会自动提交一次事务，但是在某些业务中，要求N条DML语句一起执行，一同失败。 关闭自动提交事务：conn.setAtuocommit(false); --> conn.commit();

