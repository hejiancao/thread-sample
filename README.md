# thread-demo
多线程知识集合

## 线程状态
1. 新建：NEW
2. 可运行：RUNNABLE 
3. 阻塞：BLOCKED 
4. 等待：WAITING , TIMED_WAITING 
5. 终止：TERMINATED

## 线程终止
## 线程通信
## ThreadLocal
## 线程池
### 线程池源码解释如下：<br>
 #### 核心池和最大池大小
 ThreadPoolExecutor将根据corePoolSize（参见getCorePoolSize）和maximumPoolSize（参见getMaximumPoolSize）设置的界限自动调整池大小（参见getgetpoolsize）。当在方法execute（Runnable）中提交新任务，并且运行的线程少于corePoolSize时，将创建一个新线程来处理请求，即使其他工作线程处于空闲状态。如果运行的线程超过corePoolSize但小于maximumPoolSize，则仅当队列已满时才会创建新线程。通过将corePoolSize和maximumPoolSize设置为相同的值，可以创建一个固定大小的线程池。通过将maximumPoolSize设置为一个本质上无边界的值，例如Integer.MAX_value，可以允许池容纳任意数量的并发任务。最典型的情况是，核心池和最大池大小仅在构造时设置，但也可以使用setCorePoolSize和setMaximumPoolSize动态更改它们。<br>
 #### 生存时间
 如果池当前有多个corePoolSize线程，则如果多余的线程空闲时间超过keepAliveTime（请参阅getKeepAliveTime（TimeUnit））则会终止它们。这提供了一种在池未被积极使用时减少资源消耗的方法。如果池稍后变得更为活动，则将构造新线程。此参数也可以使用方法setKeepAliveTime（long，TimeUnit）动态更改。使用Long.MAX_value TimeUnit.NANOSECONDS的值可以有效地禁止空闲线程在关闭之前终止。默认情况下，仅当存在多个corePoolSize线程时才应用keep-alive策略。但是方法allowCoreThreadTimeOut（boolean）也可以用于将此超时策略应用于核心线程，只要keepAliveTime值不为零。<br>
 #### 队列
 如果运行的线程少于corePoolSize，则执行器总是倾向于添加新线程，而不是排队。
 如果corePoolSize或更多线程正在运行，则执行器总是倾向于将请求排队，而不是添加新线程。
 如果请求不能排队，则创建一个新线程，除非该线程超过maximumPoolSize，在这种情况下，任务将被拒绝。
 排队的一般策略有三种：
 - 直接交接。工作队列的一个好的默认选择是SynchronousQueue，它将任务交给线程，而不保留线程。在这里，如果没有线程可以立即运行任务，则尝试对任务进行排队将失败，因此将构造一个新线程。此策略在处理可能具有内部依赖项的请求集时避免锁定。直接切换通常需要无限的最大工具大小，以避免拒绝新提交的任务。这反过来又允许了当命令的平均到达速度超过可以处理的速度时，无限线程增长的可能性。
 - 无限队列。当所有corePoolSize线程都很忙时，使用无边界队列（例如，没有预定义容量的LinkedBlockingQueue）将导致新任务在队列中等待。因此，不会创建超过corePoolSize的线程。（因此，maximumPoolSize的值没有任何影响。）当每个任务完全独立于其他任务时，这可能是适当的，因此任务不会影响彼此的执行；例如，在web页面服务器中。虽然这种排队方式有助于消除请求的暂时性突发，但它允许在命令平均到达速度超过其处理速度时，无限工作队列增长的可能性。
 - 有界队列。当与有限的maximumPoolSizes一起使用时，有界队列（例如ArrayBlockingQueue）有助于防止资源耗尽，但可能更难进行优化和控制。队列大小和最大池大小可以相互交换：使用大队列和小池可以最小化CPU使用率、操作系统资源和上下文切换开销，但可能导致人为的低吞吐量。如果任务经常阻塞（例如，如果它们是I/O绑定的），系统可能能够为比您允许的更多的线程安排时间。使用小队列通常需要较大的池大小，这使得cpu更加繁忙，但可能会遇到不可接受的调度开销，这也会降低吞吐量。<br>
 #### 拒绝的任务
 在方法execute（Runnable）中提交的新任务将在执行器关闭，以及执行器对最大线程和工作队列容量使用有限边界时被拒绝，并且已饱和。在这两种情况下，execute方法都会调用其RejectedExecutionHandler的rejectedExecution.rejectedExecution（Runnable，ThreadPoolExecutor）方法。提供了四个预定义的处理程序策略：
 - 在默认的ThreadPoolExecutor.AbortPolicy中，处理程序在拒绝时抛出运行时RejectedExecutionException。
 - 在ThreadPoolExecutor.callerrunpolicy中，调用execute本身的线程运行任务。这提供了一个简单的反馈控制机制，可以降低提交新任务的速度。
 - 在ThreadPoolExecutor.DiscardPolicy中，无法执行的任务将被简单地删除。
 - 在ThreadPoolExecutor.DiscardOldestPolicy中，如果未关闭执行器，则删除工作队列头部的任务，然后重试执行（这可能会再次失败，导致重复执行）
 可以定义和使用其他类型的RejectedExecutionHandler类。这样做需要一些注意，特别是当策略设计为仅在特定容量或排队策略下工作时。<br>
 #### 钩子法
 此类提供在执行每个任务之前和之后调用的受保护的可重写beforexecute（Thread，Runnable）和afterExecute（Runnable，Throwable）方法。它们可用于操作执行环境；例如，重新初始化ThreadLocals、收集统计信息或添加日志项。此外，可以重写终止的方法以执行在执行器完全终止后需要执行的任何特殊处理。
 如果钩子或回调方法抛出异常，则内部工作线程可能会依次失败并突然终止。<br>
 #### 队列维护
 方法getQueue（）允许访问工作队列以进行监视和调试。强烈建议不要将此方法用于任何其他目的。提供了两个方法remove（Runnable）和purge，用于在取消大量排队任务时帮助进行存储回收。<br>
 #### 定稿
 不再在程序中引用且没有剩余线程的池将自动关闭。如果您希望确保即使用户忘记调用shutdown也回收未引用的池，则必须通过设置适当的保持活动时间、使用零核心线程的下限和/或设置allowCoreThreadTimeOut（布尔值）来安排未使用的线程最终消亡。<br>
 
