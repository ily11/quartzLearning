触发器监听器(TriggerListeners)和任务监听器(JobListeners )分别接收关于Trigger和Job的事件。触发器相关的事件包括：触发器激活、激活失败以及激活完成（执行的任务运行完毕）。

TriggerListener 接口形式如下：
```
public interface TriggerListener {

    public String getName();
    public void triggerFired(Trigger trigger, JobExecutionContext context);
    public boolean vetoJobExecution(Trigger trigger, JobExecutionContext context);
    public void triggerMisfired(Trigger trigger);
    public void triggerComplete(Trigger trigger, JobExecutionContext context,int triggerInstructionCode);
}
```
 
任务相关的事件包括：任务即将被执行的通知、任务执行完毕的通知。JobListener 接口形式如下：

```
public interface JobListener {

    public String getName();

    public void jobToBeExecuted(JobExecutionContext context);

    public void jobExecutionVetoed(JobExecutionContext context);

    public void jobWasExecuted(JobExecutionContext context,

            JobExecutionException jobException);

}
```

### 使用你自己的监听器
实现TriggerListener和/或JobListener即可实现你自己的监听器。监听器需要注册到调度器，且必须给他一个名称（或者说它们必须能够通过其getName方法来获取其名称）。

为了方便起见，你可以继承JobListenerSupport 类或者TriggerListenerSupport ，然后重载你感兴趣的方法即可。

监听器需要与一个匹配器一起注册到调度器，该匹配器用于指定监听器想要接收哪个触发器/任务的事件。监听器在运行期间注册到调度器，且并没有与触发器和任务一起存储到JobStore 中。这是因为监听器通常是你应用的一个结合点，因此每次应用运行时它都需要重新注册到调度器。

在指定任务中添加感兴趣的任务监听器方法如下：

scheduler.getListenerManager().addJobListener(myJobListener, KeyMatcher.jobKeyEquals(new JobKey("myJobName", "myJobGroup")));

通过以下对匹配器和Key的静态引用，可以使代码更整洁：

```
import static org.quartz.JobKey.*;

import static org.quartz.impl.matchers.KeyMatcher.*;

import static org.quartz.impl.matchers.GroupMatcher.*;

import static org.quartz.impl.matchers.AndMatcher.*;

import static org.quartz.impl.matchers.OrMatcher.*;

import static org.quartz.impl.matchers.EverythingMatcher.*;

...etc.
```
 

它使代码变成这样：

scheduler.getListenerManager().addJobListener(myJobListener, jobKeyEquals(jobKey("myJobName", "myJobGroup")));

在组中为所有任务添加感兴趣的任务监听器方法如下：

scheduler.getListenerManager().addJobListener(myJobListener, jobGroupEquals("myJobGroup"));

在两个指定组中为所有任务添加感兴趣的任务监听器方法如下：

scheduler.getListenerManager().addJobListener(myJobListener, or(jobGroupEquals("myJobGroup"), jobGroupEquals("yourGroup")));

为所有任务添加感兴趣的任务监听器方法如下：

scheduler.getListenerManager().addJobListener(myJobListener, allJobs());

注册TriggerListeners 的方法与以上相同。大多数Quartz用户并不会用到监听器，然而应用程序需要得到事件通知的话它们是非常易用的，而且不需要任务本身显式地通知应用程序。