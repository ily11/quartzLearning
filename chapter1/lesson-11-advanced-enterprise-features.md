### 集群
集群目前与JDBC-Jobstore (JobStoreTX 或 JobStoreCMT) 以及TerracottaJobStore一同工作。这些特性包括负载均衡和任务容错（如果JobDetail的"request recovery" 设置为true）。

使用JobStoreTX 或 JobStoreCMT的集群

通过将"org.quartz.jobStore.isClustered"属性值设置为true来启用集群。集群中的每个实例都应使用quartz.properties文件的同一份拷贝。使用唯一配置文件的例外情况包含以下可允许的例外：线程池容量不同，以及 "org.quartz.scheduler.instanceId" 属性值不同。集群中的每个节点都必须有一个唯一的实例ID，将该值设为"AUTO"即可轻易实现（不需要不同的配置文件）。

用于不要在不同的机器上使用集群，除非它们使用了某种时钟同步服务实现了时钟同步，且运行的非常规律（每个机器的时钟必须在同一秒内）。如果你还不太熟悉怎么实现可以看这里http://www.boulder.nist.gov/timefreq/service/its.htm。

如果已经有集群实例使用了一组数据表，永远不要激活一个同样使用该组数据表的非集群实例，可能发生严重的数据冲突，并且运行状态一定是不稳定的。

每个任务每次只会被一个节点激活。我的意思是，如果某任务有一个触发器，它每隔10秒钟激活一次，那么在12:00:00只有一个节点执行该任务，在12:00:10也有一个节点执行该任务，以此类推。并不需要每次都是同一个节点--具体是哪个多少有一些随机性。对于繁忙调度器（有大量触发器）的负载均衡机制是近似随机的，但对于非繁忙调度器（只有一两个触发器）每次都是相同的活跃节点。

使用TerracottaJobStore的集群

只需要将调度器配置为使用TerracottaJobStore （第9章已经介绍），你的调度器就会全部配置为集群。也许你还想考虑如何设置你的Terracotta 服务器，尤其是如何开启持久性之类选项的配置，并且为了高可用而运行一批Terracotta 服务器。商业版的TerracottaJobStore 提供了Quartz 的高级特性，允许智能地将任务定位到合适的集群节点。关于JobStore 和Terracotta 的更多信息请查看 http://www.terracotta.org/quartz。

### JTA 事务
在第9章已经解释过，JobStoreCMT 允许Quartz 的调度器在JTA 事务中运行。通过将"org.quartz.scheduler.wrapJobExecutionInUserTransaction"属性设置为true，任务也可以在JTA 事务中运行（UserTransaction）。设置该值后，一个JTA 事务的begin()将在任务的execute 方法执行前被调用，并且在commit() 结束时commit() 被调用。这适用于所有任务。

如果你想为每个任务指定是否要由JTA 事务来包装它的执行，那你应当在该任务的类中使用@ExecuteInJTATransaction标注。

当使用JobStoreCMT时，除了JTA 事务自动包装任务执行之外，你在调度器接口上所进行的调用也会参与到事务中。你只需要确认在调度器上调用方法前已经启动了一个事务即可。你可以使用UserTransaction来直接完成，或通过将你使用调度器的代码放到一个使用了容器管理事务的SessionBean 中。