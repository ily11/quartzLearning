Triggers与Job一样的易于使用，但是它包含了大量的可定制的选项。前面提到过，有不同类型的Triggers可供你选择以满足不同的需求。

### 触发器通用属性触发器通用属性
除了用于标识身份的TriggerKey 外，还有许多各种Triggers所通用的属性，它们是在创建Triggers定义时通过TriggerBuilder 设置的。

下面列出这些通用属性：

jobKey：指定了Triggers激活时将被执行的job的身份；

startTime：指定了调度器何时开始作用。该值是一个java.util.Date对象。某些类型的Triggers确实会在startTime激活，而另一些Triggers仅简单地标记下调度器将来应当启动的时间。这意味着你可以存储一个Trigger和一个“1月的第5天”这样的调度器，如果startTime设在了4月1号，那么第一次启动时间要在几个月以后。

endTime：指定了Trigger的调度器在何时不再生效。换句话说，一个触发器的如果设置为“每月的第5天”并且endTime为“7月1日”那么它最后一次激活应该是在6月5日。

其它属性在以后的章节中介绍。

### 优先级
有时候当你有许多触发器（或者Quartz 线程池中的线程很少）时，Quartz 可能没有足够的资源来同时激活所有的触发器。这时，你可能想要控制让哪一个触发器先被触发。出于这个目的，你可以设置触发器的priority 属性。如果有N个触发器将要同时被激活，然而Quartz 只有Z个线程，那么只有前Z个优先级最高的触发器将被激活。如果没有设置，优先级的默认值为5。优先级的取值适用于所有整数，包括正数和负数。

注意：仅当触发器将同时被激活时才会比较优先级。设定在10:59的触发器永远比设定在11:00的触发器先激活。

注意：如果检测到一个job要求恢复，那么恢复后的优先级不变。

### 激活失败指令（Misfire Instructions）
触发器的另一个重要属性是激活失败指令。当一个持久的触发器因为调度器被关闭或者线程池中没有可用的线程而错过了激活时间时，就会发生激活失败(Misfire)。不同类型的触发器具有不同的激活失败指令。默认情况下它们使用一个“聪明策略”指令，它具有基于触发器类型和配置的动态行为。当调度器启动时，它会搜索所有激活失败的持久触发器，然后根据各自已配置的激活失败指令来对它们进行更新。当你在项目中使用Quartz时，你应当熟悉相应触发器的激活失败指令，它们在JavaDoc中有解释。在本教程针对各种触发器的章节中有更详细的介绍。

### 日历
Quartz Calendar 对象（不是java.util.Calendar对象）可以在触发器被定义时被关联并存储到调度器。在从触发器的激活策略中排除时间块时Calendar 非常有用。比如说，你可以添加一个触发器，它在每天早上的9:30激活，然后添加一个Calendar 来排除掉所有的商业假日。

Calendar 可以是任何实现了Calendar 接口的可序列化对象。Calendar 接口如下所示：
```
package org.quartz;

public interface Calendar {

  public boolean isTimeIncluded(long timeStamp);

  public long getNextIncludedTime(long timeStamp);

}
```
注意上述方法的参数类型是long，就像你猜想的那样，它们是毫秒单位的时间戳。这意味着Calendar 能够以毫秒的精度来排除时间块。你很可能会对排除整日感兴趣，为了方便起见，Quartz 包含了org.quartz.impl.HolidayCalendar，它可以实现这个。

Calendars 必须被实例化并使用调度器的addCalendar(..) 进行注册。如果你使用HolidayCalendar，在初始化以后你必须使用 addExcludedDate(Date date) 来生成你想要排除的日期。同一个Calendar实例可以被不同的触发器使用，就像下面这样：

Calendar Example

```
HolidayCalendar cal = new HolidayCalendar();

cal.addExcludedDate( someDate );

cal.addExcludedDate( someOtherDate );

sched.addCalendar("myHolidays", cal, false);

Trigger t = newTrigger()
    .withIdentity("myTrigger")
    .forJob("myJob")
    .withSchedule(dailyAtHourAndMinute(9, 30)) // execute job daily at 9:30
    .modifiedByCalendar("myHolidays") // but not on holidays
    .build();
// .. schedule job with trigger

Trigger t2 = newTrigger()
    .withIdentity("myTrigger2")
    .forJob("myJob2")
    .withSchedule(dailyAtHourAndMinute(11, 30)) // execute job daily at 11:30
    .modifiedByCalendar("myHolidays") // but not on holidays

    .build();
// .. schedule job with trigger2
```
触发器的创建/构造将在后面的章节中介绍，这里你只需要记住上面的代码生成了2个触发器，均在每天激活。然而，排除日期中的激活都会被跳过。
