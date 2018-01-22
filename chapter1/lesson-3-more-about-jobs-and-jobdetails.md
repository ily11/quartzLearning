在上一节中，可以看出，Jobs很容易实现，只要有一个“execute”接口方法，我们还需要了解更多关于Job的性质，如Job接口的execute（..）方法和JobDetails等更多信息。

我们希望通过JobDetail类来实例化Job类的各种属性。JobDetail实例是使用JobBuilder类构建的。 

第一节的代码引进了一个HelloJob类，这个job类具体可以如下定义：
```
public class HelloJob implements Job {

    public HelloJob() {
    }

    public void execute(JobExecutionContext context)
      throws JobExecutionException
    {
      System.err.println("Hello!  HelloJob is executing.");
    }
  }
```

我们给调度程序一个JobDetail实例，并知道要执行的作业类型，只需在构建JobDetail时提供作业的类即可。调度程序执行作业的每个（每一个）时间，都会在调用其execute（..）方法之前创建该类的新实例。 执行完成后，将删除对作业类实例的引用，然后对实例进行垃圾回收。构造器每次执行execute方法时，它都会先创建一个新的实例，这一行为的一个影响是Job必须有一个无参数构造方法；另一个影响是，Job类所定义的的状态数据是无意义的，因为它们在执行时无法被保存。

现在你可能会问，如何向Job实例提供属性/配置？如何在执行函数中保存状态数据？答案是JobDataMap，它是JobDetail对象的一部分。

### JobDataMap

JobDataMap是JAVA Map接口的一个实现，并添加了一些实现用于方便地存取基本类型数据。它能够存储任意规模的，你想要由Job在执行时使用的数据。

下面是一个在定义JobDetail时为JobDataMap添加数据的例子：
```
// define the job and tie it to our DumbJob class

  JobDetail job = newJob(DumbJob.class)

      .withIdentity("myJob", "group1") // name "myJob", group "group1"

      .usingJobData("jobSays", "Hello World!")

      .usingJobData("myFloatValue", 3.141f)

      .build();
```

下面是一个任务执行时从JobDataMap获取数据的例子：

```
public class DumbJob implements Job {

    public DumbJob() {
    }

    public void execute(JobExecutionContext context)
      throws JobExecutionException
    {
      JobKey key = context.getJobDetail().getKey();

      JobDataMap dataMap = context.getJobDetail().getJobDataMap();

      String jobSays = dataMap.getString("jobSays");
      float myFloatValue = dataMap.getFloat("myFloatValue");

      System.err.println("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }
  }
```

Triggers也可以与JobDataMap进行关联。比方说，当一个任务关联到多个触发器时，你就可以为每一个不同的触发器提供不同的数据。

任务执行期间，可以由JobExecutionContext 的getMergedJobDataMap方法获取一个由JobDetail 和Trigger所提供的合并体的JobDataMap，且后者的同名KEY的值会覆盖前者。当然，你也可以使用context.getJobDetail().getJobDataMap()来获取JobDetail 的数据，使用context.getTrigger().getJobDataMap()来获取Trigger的数据。

### Job实例
你可以只创建一个Job，然后创建多个JobDetail ，每一个都有自己的属性和JobDataMap，然后把它们都添加到调度器中，这样调度器内部就保存它的多个实例的定义。

比如说，你可以建立一个Job类叫做SalesReportJob，它会期望（通过JobDataMap）传递过来的参数指定了销售报告所针对的人员名称。他们可能会为这个Job创建多个定义，比如SalesReportForJoe和SalesReportForMike，分别指定了Joe和Mike。

当触发器激活时，与他关联的JobDetail 被加载，那么相关的Job类将被JobFactory 实例化。默认的JobFactory 只是简单地调用newInstance()方法，然后试图调用类内与JobDataMap的KEY名相匹配的setter方法。你可能想建立自己的JobFactory ，以便使你应用程序的IOC或DI container 来产生/初始化Job实例。

在Quartz语言中，我们将所有存储的JobDetail 称为“job 定义”或者“JobDetail 实例”；将所有的执行中的任务称为“job 实例”或者“job 定义的实例”；一般情况下如果我们使用Job这个词我们是指一个命名的定义或JobDetail 。当我们描述实现了Job接口的类时，一般会使用“job class”这个词。

### 任务状态和并发任务状态和并发
这里介绍一些关于任务状态和并发的额外的说明。有许多标注可以被添加到job class中，影响到Quartz's 在各方面的行为。

@DisallowConcurrentExecution

这个标注添加到job class中就是告诉Quartz，我不能被并行执行。拿前面的例子来说，如果SalesReportJob添加了这个标注，那么在同一时间只能有一个SalesReportForJoe的实例在执行，但是SalesReportForMike的实例可以同时执行。这个限制是基于JobDetail而不是 job class的实例。然而它被决定（在Quartz的设计中）要作用于类自身之上，因为它经常会影响到类的编写方式。

@PersistJobDataAfterExecution

它告诉Quartz，在成功执行完（没有抛出异常）之后要更新JobDetail的JobDataMap。这样下一次执行时该任务就会收到最新的数据。与@DisallowConcurrentExecution 一样，它也作用于一个 job definition 实例，而不是job类的实例。

@PersistJobDataAfterExecution 

如果使用了本标注，那么你要强烈考虑同时使用DisallowConcurrentExecution 标注，以避免当同一个任务的2个实例并行执行时最终保存的数据是什么样的（竞争条件下）这种冲突。

### 任务的其他属性
持久性

如果一个任务不是持久的，当不再有关联的活动触发器时它将从调度器中被删除。换句话说，非持久任务的生命期取决于它的触发器。

请求恢复

如果一个任务请求恢复，并且在执行中调度器遭到了硬关闭，那么当调度器重新启动时它将重新执行。在这种情况下，JobExecutionContext.isRecovering()将返回true。

### 任务执行异常
最后，我们要告诉你Job.execute(..)的一些细节。在该方法中你只被允许抛出JobExecutionException这一种异常（包括运行时异常）。因此，你通常要把所有代码包裹在try-catch块中。你还需要花一点时间来查看JobExecutionException的文档，以便于你能够向调度器发出各种指令来根据你的意愿来处理异常。

