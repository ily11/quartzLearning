### 介绍

corn是一个UNIX工具，已经存在很长一段时间,所以他的调度功能是强大的和成熟的。CornTrigger类是基于corn的调度功能的。

CronTrigger使用“cron表达式”，可以创建诸如“每周一到周五上午8:00”或“每个月最后一个星期五上午1:30”的发布时间表。

Cron表达式是强大且相当复杂的。 本教程旨在解决创建cron表达式的一些谜题，为用户提供一个可以访问查阅的资源。

### 格式

cron表达式是由空格分隔的6或7个字段组成的字符串。字段可以包含任何允许的值，以及该字段允许的特殊字符的各种组合。 字段如下：

| 字段名称 | 强制性 | 允许值 | 允许的特殊字符 |
| :--- | :--- | :--- | :--- |
| Seconds | YES | 0-59 | , - \* / |
| Minutes | YES | 0-59 | , - \* / |
| Hours | YES | 0-23 | , - \* / |
| Day of month | YES | 1-31 | , - \* ? / L W |
| Month | YES | 1-12 or JAN-DEC | , - \* / |
| Day of week | YES | 1-7 or SUN-SAT | , - \* ? / L \# |
| Year | NO | empty, 1970-2099 | , - \* / |

所以cron表达式可以像这样简单：\* \* \* \*？\*

或更复杂，如下所示：0/5 14,18,39,52 \*？ JAN，MAR，SEP MON-FRI 2002-2010

### 特殊字符

* \*（所有值）-- 用来选择某一段的所有值。例如，在分钟字段时，表示“每分钟”；

* ?（没有指定值）-- 字符仅被用于天（月）和天（星期）两个子表达式，表示不指定值，当2个子表达式其中之一被指定了值以后，为了避免冲突，需要将另一个子表达式的值设为“？”。例如:要在每月的10号触发一个操作，但不关心是周几，所以需要周位置的那个字段设置为"?" 具体设置为 0 0 0 10 \* ?；

* -指指定范围。例如，在小时字段中使用“10-12”，则表示从10到12点，即10,11,12；

* ， -- 用来指定一个列表值，如在星期字段中使用“MON,WED,FRI”，则表示星期一，星期三和星期五；

* / -- 用来指定增量。例如，“0/15“在秒字段中指“0，15，30，45秒”的时候。“5/15”指“5，20，35，50秒”的时候。你也可以在''字符之后指定'/'，在这种情况下''等于在'/'之前有'0'。 月份字段中的“1/3”表示“从每月的第一天开始每3天发生启动一次”。

* L（last）-- 该字符只在日期和星期字段中使用，代表“Last”的意思，但它在两个字段中意思不同。 例如，月份字段中的值“L”意味着“月份的最后一天”，即1月31号或者非闰年的2月28日的。 如果单独使用在星期几字段中，则仅表示“7”或“SAT”。 但如果在星期几字段中使用另一个值，则表示“该月份的最后一个xxx日” - 例如“6L”表示“该月的最后一个星期五”。 您还可以指定月份的最后一天的偏移量，例如“L-3”，表示日历月份的倒数第三天。 当使用“L”选项时，不要指定列表或值的范围，因为您会得到令人困惑/意外的结果；

* W（“工作日”） -- 用于指定与指定日期最近的工作日（星期一至星期五）。 例如，如果您要指定“15W”作为月份日期字段的值，则其含义是：“本月15日最近的工作日”。 所以如果15日是星期六，触发器将在14日星期五启动。 如果15日是星期天，触发器将在16日星期一启动。 如果15日是星期二，那么15日星期二就会启动。但必须注意关联的匹配日期不能够跨月，如你指定1W，如果1号是星期六，结果匹配的是3号星期一，而非上个月最后的那天。W字符串只能指定单一日期，而不能指定日期范围；

  “L”和“W”字符也可以在月份日期字段中合并为“LW”，即“月份的上个工作日”\*。

* ＃ -- 该字符只能在星期字段中使用，表示当月某个工作日。如6\#3表示当月的第三个星期五\(6表示星期五，\#3表示当前的第三个\)，而4\#5表示当月的第五个星期三，假设当月没有第五个星期三，忽略不触发；

* C -- 该字符只在日期和星期字段中使用，代表“Calendar”的意思。它的意思是计划所关联的日期，如果日期没有被关联，则相当于日历中所有日期。例如5C在日期字段中就相当于日历5日以后的第一天。1C在星期字段中相当于星期日后的第一天；

Cron表达式对特殊字符的大小写不敏感，对代表星期的缩写英文大小写也不敏感。

一些例子：

| 表达式 | 说明 |
| :--- | :--- |
| 0 0 12 \* \* ? | 每天12点运行 |
| 0 15 10 ? \* \* | 每天10:15运行 |
| 0 15 10 \* \* ? | 每天10:15运行 |
| 0 15 10 \* \* ? \* | 每天10:15运行 |
| 0 15 10 \* \* ? 2008 | 在2008年的每天10：15运行 |
| 0 \* 14 \* \* ? | 每天14点到15点之间每分钟运行一次，开始于14:00，结束于14:59。 |
| 0 0/5 14 \* \* ? | 每天14点到15点每5分钟运行一次，开始于14:00，结束于14:55。 |
| 0 0/5 14,18 \* \* ? | 每天14点到15点每5分钟运行一次，此外每天18点到19点每5钟也运行一次。 |
| 0 0-5 14 \* \* ? | 每天14:00点到14:05，每分钟运行一次。 |
| 0 10,44 14 ? 3 WED | 3月每周三的14:10分到14:44，每分钟运行一次。 |
| 0 15 10 ? \* MON-FRI | 每周一，二，三，四，五的10:15分运行。 |
| 0 15 10 15 \* ? | 每月15日10:15分运行。 |
| 0 15 10 L \* ? | 每月最后一天10:15分运行。 |
| 0 15 10 ? \* 6L | 每月最后一个星期五10:15分运行。 |
| 0 15 10 ? \* 6L 2007-2009 | 在2007,2008,2009年每个月的最后一个星期五的10:15分运行。 |
| 0 15 10 ? \* 6\#3 | 每月第三个星期五的10:15分运行。 |



