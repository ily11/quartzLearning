Quartz 的结构是模块化的，因此要让它运行起来的话各个组件之间必须很好地结合起来。在运行Quartz 之前必须先进行配置的主要组件有：

* 线程池

* JobStore

* DataSources （如果需要的话）

* 调度器自身

线程池为Quartz 提供了一个线程集以便在执行任务时使用。线程池中线程越多，可以并发执行的任务数越多。然而，过多的线程可能会是你的系统变慢。许多Quartz 用户发现5个左右的线程就足够了--因为在任意时刻的任务数都不会超过100，而且通常它们都不会要求要同时执行，而且这些任务的生命期都很短（很快就结束）。另一些用户发现他们需要10,15,50或者甚至100个线程，因为他们在任意时刻都有成千上万的触发器和大量的调度器--它们都平均至少有10-100个任务需要在任一时刻运行。为你的调度器池找到合适容量完全取决于你如何使用它。并没有严格的规律，除了保持线程数量尽可能的少（为了节约你机器的资源）--然而你要确保足以按时激活你的任务。注意，如果一个触发器达到了激活时间而没有可用线程，Quartz 将阻塞（暂停）直到有一个可用的线程，然后任务将被执行--在约定时间的数个毫秒之后。这甚至会导致线程的激活失败--如果在配置的"misfire threshold"期间都没有可用的线程。

在org.quartz.spi 包中定义了一个线程池接口，你可以根据你的洗好创建一个线程池应用。Quartz 包含一个简单（但非常令人满意的）线程池叫做org.quartz.simpl.SimpleThreadPool。这个线程池只是维护一个固定的线程集合--从不会增多也不会减少。但它非常健壮并且得到了非常好的测试--几乎所有使用Quartz 的用户都会使用它。

JobStores 和DataSources 在第9章已经讨论过。这里需要提醒的是一个事实，所有的JobStores 都实现了org.quartz.spi.JobStore接口--如果某个打包好的JobStores 不满足你的需要，你可以自己创建一个。

最后，你需要创建你自己的调度器实例。调度器自身需要一个名称，告知它的RMI参数并把JobStore 和线程池的实例传递给它。RMI 参数包含了调度器是否应该将自己创建为一个RMI的服务对象（使它对远程连接可用），使用哪个主机和端口等等。StdSchedulerFactory 也可以生产调度器实例，它实际上是到远端进程创建的调度器的代理。

### StdSchedulerFactory
StdSchedulerFactory是一个org.quartz.SchedulerFactory接口的实现。它使用一系列的属性 (java.util.Properties)来创建和初始化Quartz 调度器。属性通常存储在文件中并从它加载，也可以由你的应用程序产生并直接传递到factory。直接调用factory 的getScheduler() 方法将产生调度器，初始化它（以及它的线程池、JobStore 和DataSources），然后返回它的公共接口的句柄。

在Quartz 分发包的"docs/config"目录下有一些示例配置（包含属性的描述）。你可以Quartz 文档的"Reference"章节的配置手册中寻找完整的文档。

### DirectSchedulerFactory
DirectSchedulerFactory是另一个SchedulerFactory 实现。它对于那些想要以更程序化的方式来创建调度器实例的人来说是有用的。它一般不建议使用，因为：（1）它要求用户非常明白他在做什么（2）它不允许声明式的配置--换句话说，你需要对调度器的设置进行硬编码。

### 日志
Quartz 使用SLF4J 框架来满足所有的日志需要。为了调整日志设置（比如输出量，以及输出路径），你需要理解SLF4J 框架，这超出了本文的范围。如果你想要获取关于触发器激活和任务执行的额外信息，你可能会对如何启用org.quartz.plugins.history.LoggingJobHistoryPlugin和/或rg.quartz.plugins.history.LoggingTriggerHistoryPlugin感兴趣。