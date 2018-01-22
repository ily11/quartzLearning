调度器监听器(SchedulerListeners)与TriggerListeners 和JobListeners非常类似，除了它是接收来自于调度器自己的事件--不一定与某个特定的trigger或job相关。

Scheduler相关的事件包括：trigger/job的添加与删除、Scheduler内部的严重错误、调度器被关闭的通知，等等。

SchedulerListener 接口形式如下：

```
public interface SchedulerListener {

    public void jobScheduled(Trigger trigger);

    public void jobUnscheduled(String triggerName, String triggerGroup);

    public void triggerFinalized(Trigger trigger);

    public void triggersPaused(String triggerName, String triggerGroup);

    public void triggersResumed(String triggerName, String triggerGroup);

    public void jobsPaused(String jobName, String jobGroup);

    public void jobsResumed(String jobName, String jobGroup);

    public void schedulerError(String msg, SchedulerException cause);

    public void schedulerStarted();

    public void schedulerInStandbyMode();

    public void schedulerShutdown();

    public void schedulingDataCleared();

}
```
 

SchedulerListeners 注册到调度器的ListenerManager。基本上任何实现了SchedulerListener 的类都可以作为调度器监听器。

添加SchedulerListener的方法如下：

scheduler.getListenerManager().addSchedulerListener(mySchedListener);

删除SchedulerListener的方法如下：

scheduler.getListenerManager().removeSchedulerListener(mySchedListener);