### 插件
Quartz 为插入额外功能提供了一个接口(org.quartz.spi.SchedulerPlugin)。Quartz包含的插件提供了各种工具特性，你在org.quartz.plugins包中能找到。它们提供了诸如任务自动调度、记录历史任务和触发器事件、确保在JVM存在时调度器干净的关闭等特性。

### 任务工厂(JobFactory)
当触发器激活时，相关的任务将被调度器配置的JobFactory 实例化。默认的JobFactory 只是简单的调用任务类的newInstance()方法。你可能想要创建你自己的JobFactory 实现以实现诸如让你应用程序的IoC 或DI容器来产生或初始化任务实例。查看org.quartz.spi.JobFactory接口，以及相关的Scheduler.setJobFactory(fact) 方法。

### 'Factory-Shipped' Jobs
Quartz还提供了大量工具任务，你可以使用它们来完成诸如发送邮件和激活EJB等工作。这些开箱即用的任务在org.quartz.jobs 包中可以找到。