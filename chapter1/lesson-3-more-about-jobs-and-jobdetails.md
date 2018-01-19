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

