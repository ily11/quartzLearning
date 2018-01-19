### The Quartz API {#TutorialLesson2-QuartzAPI}

最主要的Quartz API接口有：

* 调度器（Scheduler）—— 与调度程序交互的主要API。
* 任务（Job）—— 由调度程序执行的组件实现的接口。
* JobDetail —— 用于定义Jobs的实现。
* 触发器（Trigger）—— 定义执行给定Job时间表的组件。
* JobBuilder —— 用于定于/构建JobDetail实现。
* TriggerBuilder —— 用于定义/构建触发器的实现。

调度程序的生命周期受其创建的限制，通过SchedulerFactory和对其shutdown（）方法的调用。 一旦创建了Scheduler接口，就可以使用添加，删除和列出作业和触发器，并执行其他调度相关的操作（例如暂停触发器）。 但是，如第1课所示，调度程序实际上不会对任何触发器（执行作业）执行任何操作，直到它已经用start（）方法启动。

Quartz提供定义域特定语言（或DSL，有时也称为“流畅接口”）的“构建器”类。 在之前的课程中，您看到了一个例子，我们在这里再次介绍一部分：

```
 // define the job and tie it to our HelloJob class
  JobDetail job = newJob(HelloJob.class)
      .withIdentity("myJob", "group1") // name "myJob", group "group1"
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

构建作业定义的代码块使用从JobBuilder类静态导入的方法。 同样，构建触发器的代码块使用从TriggerBuilder类导入的方法 - 以及SimpleScheduleBulder类。

DSL的静态导入可以通过如下的导入语句来实现：

```
import static org.quartz.JobBuilder.*;
import static org.quartz.SimpleScheduleBuilder.*;
import static org.quartz.CronScheduleBuilder.*;
import static org.quartz.CalendarIntervalScheduleBuilder.*;
import static org.quartz.TriggerBuilder.*;
import static org.quartz.DateBuilder.*;
```

各种“ScheduleBuilder”类具有与定义不同类型的时间表有关的方法。

DateBuilder类包含各种方法，可以方便地为特定的时间点构造java.util.Date实例（例如表示下一个偶数小时的日期 - 换句话说，如果它是当前的9:43:27）。

### Jobs and Triggers

Job是一个实现Job接口的类，它只有一个简单的方法：

job接口
```
  package org.quartz;

  public interface Job {

    public void execute(JobExecutionContext context)
      throws JobExecutionException;
  }
```

当作业的触发器触发（更多的时候），execute（..）方法由调度程序的一个job线程调用。传递给此方法的JobExecutionContext对象为作业实例提供有关其“运行时”环境的信息 - 调度程序执行的句柄，触发执行的触发器的句柄，作业的JobDetail对象以及其他几个项目。

JobDetail对象是在Job添加到调度器时由Quartz客户端（你的程序）创建的。它包含Job的各种属性设置，以及一个JobDataMap，它可以用来存储作业类的给定实例的状态信息。它基本上是作业实例的定义，在下一课中会进一步详细讨论。

触发器对象用于触发作业的执行（或“触发”）。当你想安排一个工作，你实例化一个触发器并“调整”它的属性来提供你想要的调度。触发器也可能有一个与它们相关的JobDataMap - 这对于传递参数到一个特定于触发器的触发的Job是很有用的。 Quartz提供了一些不同的触发器类型，但最常用的类型是SimpleTrigger和CronTrigger。

如果您需要“一次性”执行（在某个特定时间只执行一项作业），或者您需要在给定时间开始工作，并重复执行N次，SimpleTrigger会很方便的T之间执行。如果您希望基于类似日历的时间表（例如“每个星期五，中午”或“每个月的第10天的10:15”）触发，则CronTrigger非常有用。

为什么工作和触发器？许多工作调度员没有单独的工作和触发器的概念。一些人将“工作”定义为执行时间（或时间表）以及一些小的工作标识符。其他人很像Quartz的工作和触发对象的联合。在开发Quartz的时候，我们决定把时间表和在这个时间表上进行的工作区分开来。这在我们看来有很多好处。

例如，可以独立于触发器创建作业并将其存储在作业调度程序中，并且许多触发器可以与同一作业相关联。这种松散耦合的另一个好处是能够配置在相关的触发器过期之后保留在调度器中的作业，以便以后可以重新调度，而不必重新定义它。它还允许您修改或替换触发器，而无需重新定义其关联的作业。

### Identities

作业和触发器被赋予标识键，因为它们是在Quartz调度器中注册的。 作业和触发器的键（JobKey和TriggerKey）允许它们被放置到“组”中，这可以用于组织作业和触发器，例如“报告作业”和“维护作业”等类别。 作业或触发器的键的名称部分在组中必须是唯一的 - 换句话说，作业或触发器的完整键（或标识符）是名称和组的组合。

