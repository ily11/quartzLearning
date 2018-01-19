在使用调度器之前，需要先将其实例化。只有这样，你才可以使用SchedulerFactory。Quartz的一些用户可能会在JNDI存储中保留一个工厂的实例，其他人可能会发现它直接实例化和使用工厂实例（如下面的例子）一样简单（或者更容易）。

一旦调度程序被实例化，它就可以启动，进入待机模式，并关机。 请注意，一旦调度程序关闭，如果没有重新实例化是不能自动重启的。 触发器不会触发（作业不会执行），直到调度程序启动，也不会处于暂停状态。

这里有一个快速的代码片段，它实例化并启动一个调度程序，并安排一个作业执行：

```
SchedulerFactory schedFact = new org.quartz.impl.StdSchedulerFactory();

  Scheduler sched = schedFact.getScheduler();

  sched.start();

  // define the job and tie it to our HelloJob class
  JobDetail job = newJob(HelloJob.class)
      .withIdentity("myJob", "group1")
      .build();

  // Trigger the job to run now, and then every 40 seconds
  Trigger trigger = newTrigger()
      .withIdentity("myTrigger", "group1")
      .startNow()
      .withSchedule(simpleSchedule()
          .withIntervalInSeconds(40)
          .repeatForever())
      .build();

  // Tell quartz to schedule the job using our trigger
  sched.scheduleJob(job, trigger);
```

正如你所看到的，使用quartz是相当简单的。 在第2课中，我们将简要介绍作业和触发器以及Quartz的API，以便您可以更全面地了解这个示例。