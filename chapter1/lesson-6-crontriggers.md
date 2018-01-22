如果你需要的调度器的任务循环是基于类似于日历那种，而不是以一个明确的周期为依据，那么CronTriggers往往比SimpleTrigger更有用。

有了CronTrigger，你可以指定“每个周五下午”或者“每个工作日早上9:30”，甚至是“一月的每个周一，周二和周五早上9:00--10:00之间每隔5分钟1次”这样的调度器。即使如此，与SimpleTrigger一样，CronTrigger也有一个startTime和一个（可选的）endTime，指定了调度器在应该在何时启动和停止。

### Cron表达式
Cron表达式(Cron Expressions)用于配置CronTrigger实例。它是由7个子表达式组成的字符串，指定了调度器的每一个细节。这些子表达式由空格分隔，分别代表以下内容：

* 秒

* 分

* 时

* 日

* 月

* 星期

* 年（可选）

以 "0 0 12 ? * WED"为例，它表示每月每个周二的12点。每个子表达式都可以包含范围与/或列表。比如，前例中的”WED”部分可以替换为 "MON-FRI", "MON,WED,FRI", 甚至"MON-WED,SAT"。

掩码(“*”)代表任何允许的值。因此，“月”区域的“*”表示每个月；“星期”区域的“*”表示一周的每一天。

所有区域都有一些可分配的有效值，这些值都是非常显而易见的。

 斜杠('/')表示值的增加。比如说，如果在“分钟”区域填写'0/15'，它表示从0分开始并每隔15分钟。 '3/20'表示从3分开始并 每隔20分钟，即03，23,43分。 "/35"等同于"0/35"。注意，原文档说，“ "/35"不代表每隔35分钟，而是每个小时的第35分，及等同于'0,35'。”经测试发现，该说法不正确。应等同于"0/35"。

问号 '?' 可被“日”和“星期”域使用。它表示没有指定值。当你指定了这俩域的其中一个时，另一个域就可以使用问号。

 字母'L' 可被“日”和“星期”域使用。它是”last”的缩写，但是在这两个域中有着不同的含义。比如，“日”区域中的"L"表示本月的最后一天，比如1月的31日或者平年2月的28日。如果只有它自己用在“星期”域中，它表示7或者星期六。如果在“星期”域跟在某个星期的后面，那它表示本月的上一个xx日。比如，"6L" 或者 "FRIL"都表示本月上一个星期五。你也可以为本月的最后一天指定个偏移量，比如 "L-3"表示本月的倒数第3天。当使用L选项时，非常重要的一点是不要同时指定列表或范围，否则你会得到混乱的结果。

字母'W'用于指定距离某天最近的工作日（周一到周五）。比如，你在“日”区域使用了"15W"，那它表示距离本月15号最近的工作日。

井号 '#' 用于本月第xx个工作日。比如，“星期”域的"6#3" or "FRI#3"表示本月第3个星期五。

### Cron表达式实例
下面是一些表达式实例及其含义。你可以在JavaDoc中org.quartz.CronExpression部分找到更多内容。

CronTrigger Example 1："0 0/5 * * * ?"

每天，从00分开始每隔5分钟；

CronTrigger Example 2："10 0/5 * * * ?"

每天，从00分开始每隔5分钟，在该分钟的10秒；

CronTrigger Example 3："0 30 10-13 ? * WED,FRI"

每月星期二和星期五上午，10点至13点，期间的每个半点；

CronTrigger Example 4："0 0/30 8-9 5,20 * ?"

每月5号和20号，早上8点至9点，期间的每个半点；注意不包含10:00。

注意，有一些调度要求用一个触发器来表达可能过于复杂，比如“早上9点至10点每隔5分钟，以及早上10点至13点每隔20分钟”。这时你可以构造2个触发器，然后绑定到同一个任务上。

### 构造CronTrigger
CronTrigger 的实例通过TriggerBuilder（包括CronTrigger 的主要属性） 和CronScheduleBuilder （包括CronTrigger 特有属性）来构建，为了以DSL风格引用以上类，使用如下的静态引用：

import static org.quartz.TriggerBuilder.*;

import static org.quartz.CronScheduleBuilder.*;

import static org.quartz.DateBuilder.*:
 

构造一个每天上午8点到下午五点之间每隔2分钟的CronTrigger ：
```
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 0/2 8-17 * * ?"))
    .forJob("myJob", "group1")
    .build();
``` 

构造一个每天上午10:42触发的CronTrigger ：
```
 trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(dailyAtHourAndMinute(10, 42))
    .forJob(myJobKey)
    .build();
``` 

 或者
```
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 42 10 * * ?"))
    .forJob(myJobKey)
    .build();
``` 

构造一个非系统默认时区内每周二上午10:42触发的CronTrigger ：

```
trigger = newTrigger()
    .withIdentity("trigger3", "group1")
.withSchedule(weeklyOnDayAndHourAndMinute(DateBuilder.WEDNESDAY, 10, 42))
    .forJob(myJobKey)
    .inTimeZone(TimeZone.getTimeZone("America/Los_Angeles"))
    .build();
``` 

或者
````
  trigger = newTrigger()
    .withIdentity("trigger3", "group1")
    .withSchedule(cronSchedule("0 42 10 ? * WED"))
    .inTimeZone(TimeZone.getTimeZone("America/Los_Angeles"))
    .forJob(myJobKey)
    .build();
``` 
### CronTrigger激活失败指令
下列实例用于CronTrigger发生激活失败(misfire)时通知Quartz 如何处理。这些指令被定义为CronTrigger类内部的常量，包括：

MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY

MISFIRE_INSTRUCTION_DO_NOTHING

MISFIRE_INSTRUCTION_FIRE_NOW

如前所述，Trigger.MISFIRE_INSTRUCTION_SMART_POLICY依然是CronTrigger的默认指令，然而它默认选择MISFIRE_INSTRUCTION_FIRE_NOW作为执行策略。更详细的解释请查看CronTriggerImpl类的updateAfterMisfire函数。

CronTrigger激活失败指令在构造CronTrigger实例时指定。