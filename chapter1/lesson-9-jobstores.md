JobStore的职责是记录所有你提供给调度器的“工作数据”：任务、触发器、日历等等。为你的调度器实例选择合适的JobStore的一个非常重要的步骤。幸运的是，一旦你理解了它们之间的不同，选择起来是非常容易的。你在配置文件中声明将要选择哪个JobStore，以此将它提供给SchedulerFactory，因为你的调度器实例是由它生成的。

永远不要在代码中直接使用JobStore实例。有的人因为某些原因想要这样做。JobStore是给Quartz 在后台使用的。你需要（通过配置）告诉Quartz 使用哪一个JobStore，然后你只应将代码集中在调度器接口上。

### RAMJobStore
RAMJobStore 是最易于使用的obStore，也是性能最好的（从CPU时间的角度）。RAMJobStore 的名字就很说明问题：它将数据保存在内存中。这就是它为何快如闪电且已易于配置。它的缺点在于如果你的应用程序结束了或崩溃了，那么所有数据都将丢失，这说明RAMJobStore与触发器和任务设置中的”non-volatility”并不匹配。在某些应用中这是可以接受的甚至是要求的行为，但对于另一些来说这可能是灾难性的。

要使用RAMJobStore （假设你使用的是StdSchedulerFactory）只需要在配置文件中将JobStore class属性值设置为org.quartz.simpl.RAMJobStore即可。配置Quartz使用RAMJobStore的方法如下：

org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore

### JDBCJobStore
JDBCJobStore 的作用也是显而易见的--它将所有数据通过JDBC保存在数据库中。正是如此它的配置要比RAMJobStore更困难一些，也没有它快。然而性能的缺陷也不是特别的糟糕，尤其是你的数据表使用主键作为索引。在相当现代且网络环境相当好（在调度器与数据库之间）的机器上，获取并更新一个激活的触发器所需要的时间通常小于 10毫秒。

JDBCJobStore 几乎兼容所有数据区，它被广泛使用到Oracle, PostgreSQL, MySQL, MS SQLServer, HSQLDB, 以及DB2上。使用JDBCJobStore时，你必须首先为quartz创建一个表格，你在Quartz 分发包的 "docs/dbTables" 目录下能够找到创建表格的SQL脚本。如果还没有针对你的数据库的脚本，那就随便看一个，并以任何方式进行修改。有一件事情要注意，在这些脚本中所有表格都是以QRTZ开头（比如"QRTZ_TRIGGERS"和 "QRTZ_JOB_DETAIL"）。这个前缀可以是任何值，你只需要告诉JDBCJobStore 即可（在配置文件中）。在同一个数据库中为不同的调度器实例使用不同的数据表时，使用不同的前缀是有用处的。

一旦建立了数据表，在配置和使用JDBCJobStore 之前你还需要做一个重要的决定。你需要决定你的应用使用什么类型的事务(transaction)。如果你不需要将你的调度指令（例如增加或删除触发器）绑定到其他的事务，那你可以使用JobStoreTX （这是最常用的选项）让Quartz 来管理事务。

如果你要让Quartz 与其它事务（比如与一个J2EE应用服务器）一起工作，那你应当使用JobStoreCMT ，Quartz 将使用APP服务器容器来管理这些事务。

最后一部分是设置DataSource ，JDBCJobStore 要从它这里获取到你数据库的连接。DataSources 在属性文件中定义，且有一些不同的定义方式。一种是让Quartz 自己来创建和管理DataSource ，提供所有到数据库的连接信息。还有一种是让Quartz 使用自身所在的应用服务器提供的DataSource ，向JDBC提供DataSource的JNDI 名称。具体的配置请参阅 "docs/config"文件夹中的配置文件。

要使用JDBCJobStore （假设你使用的是StdSchedulerFactory）你首先要将配置文件中的JobStore class 设置为org.quartz.impl.jdbcjobstore.JobStoreTX 或者org.quartz.impl.jdbcjobstore.JobStoreCMT二者其一，取决于你对以上选项的选择。

配置Quartz使用JobStoreTx

org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX

接下来你需要选择供JobStore使用的DriverDelegate。DriverDelegate 的职责是处理你的数据库所需要的JDBC相关的工作。StdJDBCDelegate 使用了"vanilla" JDBC代码（以及SQL语句）来做这些工作。如果没有为你的数据库指定其它的Delegate你可以尝试这个--我们仅为看起来使用StdJDBCDelegate 会有问题的数据库提供了特定的Delegate。其它的Delegate位于 "org.quartz.impl.jdbcjobstore"包或其子包内，包括： DB2v6Delegate (for DB2 version 6 及以前)， HSQLDBDelegate (for HSQLDB)，MSSQLDelegate (for Microsoft SQLServer), PostgreSQLDelegate (for PostgreSQL), WeblogicDelegate (for using JDBC drivers made by Weblogic), OracleDelegate (for using Oracle),等等。

当选择了Delegate以后，你需要在配置文件中设置它的名称。

配置JDBCJobStore 使用DriverDelegate

org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate

接下来你需要通知JobStore 你使用的数据表前缀是什么。

配置JDBCJobStore 使用的数据表前缀：

org.quartz.jobStore.tablePrefix = QRTZ_

最后，你需要设置JobStore使用的DataSource 。你填写的DataSource 必须在配置文件中有定义，因此我们指定Quartz 使用"myDS"（已经在配置文件的某处定义）。

配置JDBCJobStore 使用的DataSource 名称：

org.quartz.jobStore.dataSource = myDS

如果你的调度器很繁忙（比如在运行任务数量几乎总是等于线程池的容量）那么你可能需要将DataSource 的连接数设置为线程池的容量+2。

"org.quartz.jobStore.useProperties"配置参数默认为false，为了使JDBCJobStore 的JobDataMaps的所有值均为String类型，而不是存储为更复杂的可序列化对象，该值可以设置为true。长远来看这样更加安全，因为你避免了将非字符串类序列化为BLOB类是的类版本问题。

### TerracottaJobStore
TerracottaJobStore 提供了一种不使用数据库情况下的量化和鲁棒性手段。这意味着你的数据库可以与Quartz的负载无关，并将其节省下来的资源用于你的其它应用。

TerracottaJobStore 可被集群化或非集群化使用，在这些场景下都会为你的任务数据提供一个存储介质，它在你的应用重启期间也具有持久性，因为数据存储在Terracotta 服务器。它的性能比使用基于JDBCJobStore 的数据库要好得多（大约一个数量级），但仍比RAMJobStore慢得多。

要使用TerracottaJobStore （假设你使用的是StdSchedulerFactory），只需要将配置文件中的JobStore class 设置为org.terracotta.quartz.TerracottaJobStore，并额外添加一行配置指定Terracotta 服务器的位置。

配置Quartz 使用TerracottaJobStore的方法：

org.quartz.jobStore.class = org.terracotta.quartz.TerracottaJobStore

org.quartz.jobStore.tcConfigUrl = localhost:9510

关于JobStore和Terracotta的详细信息请参阅http://www.terracotta.org/quartz。

