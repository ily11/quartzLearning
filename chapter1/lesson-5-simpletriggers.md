SimpleTrigger 能够满足你这样的需求：你希望job在某一个特定的时间执行，或者在某刻执行后以一个指定的周期进行重复。比如说，你希望某个任务在2015年1月13日早上11:23:54执行，或者在该时间执行后每隔10秒又重复执行5次。

通过上面的描述，你不难发现SimpleTrigger 的属性应当包含：start-time，end-time, repeat count, repeat interval。

repeat count可以是0或者一个正整数，或者SimpleTrigger.REPEAT_INDEFINITELY值。repeat interval必须是0或者一个正的长整型数，代表毫秒数。注意，0表示任务的重复将并行（或以调度器的管理能力近似并行）执行。

如果你对Quartz的DateBuilder类还不熟悉，你会发现它对于根据startTime (或 endTime)计算触发器的激活时间非常有用。

endTime属性（如果被设置）将会覆盖repeat count属性。这很有用，如果你想让触发器每隔10秒触发一次直到给定的时间--相对于你自己根据startTime 和endTime来计算repeat count，你可以直接设置endTime，然后将 repeat count设置为REPEAT_INDEFINITELY 。

SimpleTrigger 的实例通过TriggerBuilder（包括SimpleTrigger的主要属性） 和SimpleScheduleBuilder （包括SimpleTrigger特有属性）来构建，为了以DSL风格引用以上类，使用如下的静态引用：
```
import static org.quartz.TriggerBuilder.*;

import static org.quartz.SimpleScheduleBuilder.*;

import static org.quartz.DateBuilder.*:
```

下面是各种使用简单构造器来定义触发器的例子，把它们阅读完，因为每个都展示了不同的方面。

构造一个构造器，在指定时刻激活，没有重复：
```
SimpleTrigger trigger = (SimpleTrigger) newTrigger() 
    .withIdentity("trigger1", "group1")
    .startAt(myStartTime) // some Date 
    .forJob("job1", "group1") // identify job with name, group strings
    .build();
```
构造一个构造器，在指定时刻激活，并以10秒为周期重复10次：

```
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .startAt(myTimeToStartFiring)  // if a start time is not given (if this line were omitted), "now" is implied
    .withSchedule(simpleSchedule()
        .withIntervalInSeconds(10)
        .withRepeatCount(10)) // note that 10 repeats will give a total of 11 firings
    .forJob(myJob) // identify job with handle to its JobDetail itself                   
    .build();
```
 
构造一个构造器，在5分钟后激活1次：

```
  trigger = (SimpleTrigger) newTrigger() 
    .withIdentity("trigger5", "group1")
    .startAt(futureDate(5, IntervalUnit.MINUTE)) // use DateBuilder to create a date in the future
    .forJob(myJobKey) // identify job with its JobKey
    .build();
```
构造一个构造器，立刻激活，并以5分钟为周期重复，直到22:00停止：
```
trigger = newTrigger()
    .withIdentity("trigger7", "group1")
    .withSchedule(simpleSchedule()
        .withIntervalInMinutes(5)
        .repeatForever())
    .endAt(dateOf(22, 0, 0))
    .build();
```
 

构造一个构造器，在下一个整点激活，并以2小时为周期无限重复：

```
  trigger = newTrigger()
    .withIdentity("trigger8") // because group is not specified, "trigger8" will be in the default group
    .startAt(evenHourDate(null)) // get the next even-hour (minutes and seconds zero ("00:00"))
    .withSchedule(simpleSchedule()
        .withIntervalInHours(2)
        .repeatForever())
    // note that in this example, 'forJob(..)' is not called 
    //  - which is valid if the trigger is passed to the scheduler along with the job  
    .build();
    scheduler.scheduleJob(trigger, job);
```
 

花点时间看看TriggerBuilder和SimpleScheduleBuilder所有可以函数，这样能够熟悉以上例子没有介绍到的可能会对你由于的选项。

注意，TriggerBuilder（以及Quartz的其它builder）通常会为你没有明确指定的属性选择一个合理的值。比如：如果你没有调用*withIdentity(..)*方法，那么TriggerBuilder 将为你的触发器生成一个随机的名字；如果你没有调用 *startAt(..)*方法，那么当前时间（立刻激活）将被赋值。

### SimpleTrigger 激活失败指令
SimpleTrigger 有很多指令可用于通知Quartz 当发生激活失败时应如何处理（激活失败已经在第4节中介绍）。这些指令被定义为SimpleTrigger类的常数（JavaDoc描述了它们的行为）。这些指令包括：

MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY

MISFIRE_INSTRUCTION_FIRE_NOW

MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_REPEAT_COUNT

MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_REMAINING_REPEAT_COUNT

MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT

MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_EXISTING_COUNT

前面提到过所有触发器都有一个Trigger.MISFIRE_INSTRUCTION_SMART_POLICY指令，这是所有类型的触发器的默认指令。

如果使用了“聪明策略”，SimpleTrigger 将根据配置和触发器实例的状态从它的指令中动态选择。JavaDoc关于SimpleTriggerImpl .updateAfterMisfire() 方法的介绍解释了这一动态行为的细节，具体如下。

Repeat Count=0：instruction selected = MISFIRE_INSTRUCTION_FIRE_NOW;

Repeat Count=REPEAT_INDEFINITELY：instruction selected = MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT

Repeat Count>0：instruction selected = MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_REPEAT_COUNT


