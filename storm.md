storm简介
===

名词解释：

* spout，读取原始数据为bolt提供数据
* bolt ，从spout或其它bolt接收数据，并处理数据，处理结果可作为其它bolt的数据源或最终结果
* nimbus ，主节点的守护进程，负责为工作节点分发任务。
* topology 拓扑结构，Storm的一个任务单元
* define field(s) 定义域，由spout或bolt提供，被bolt接收

基础知识：

Storm是一个分布式的，可靠的，容错的数据流处理系统。它会把工作任务委托给不同类型的组件，每个组件负责处理一项简单特定的任务。Storm集群的输入流由一个被称作spout的组件管理，spout把数据传递给bolt， bolt要么把数据保存到某种存储器，要么把数据传递给其它的bolt。你可以想象一下，一个Storm集群就是在一连串的bolt之间转换spout传过来的数据。

Storm组件：

对于一个Storm集群，一个连续运行的主节点组织若干节点工作。
在Storm集群中，有两类节点：主节点master node和工作节点worker nodes。主节点运行着一个叫做Nimbus的守护进程。这个守护进程负责在集群中分发代码，为工作节点分配任务，并监控故障。Supervisor守护进程作为拓扑的一部分运行在工作节点上。一个Storm拓扑结构在不同的机器上运行着众多的工作节点。

Storm的特性：

在所有这些设计思想与决策中，有一些非常棒的特性成就了独一无二的Storm。

* 简化编程：如果你曾试着从零开始实现实时处理，你应该明白这是一件多么痛苦的事情。使用Storm，复杂性被大大降低了。
* 使用一门基于JVM的语言开发会更容易，但是你可以借助一个小的中间件，在Storm上使用任何语言开发。有现成的中间件可供选择，当然也可以自己开发中间件。
* 容错：Storm集群会关注工作节点状态，如果宕机了必要的时候会重新分配任务。
* 可扩展 ：所有你需要为扩展集群所做的工作就是增加机器。Storm会在新机器就绪时向它们分配任务。
* 可靠的：所有消息都可保证至少处理一次。如果出错了，消息可能处理不只一次，不过你永远不会丢失消息。
* 快速：速度是驱动Storm设计的一个关键因素
* 事务性：你可以为几乎任何计算得到恰好一次消息语义。


以下是设置Storm群集的步骤：

1. 建立一个Zookeeper集群
2. 安装依赖项
3. 下载Storm
4. 配置storm.yaml文件
5. 启动storm集群

首先需要搭建一个zookeeper集群
===

Storm使用Zookeeper来协调群集。 Zookeeper不用于消息传递，所以Zookeeper上的负载Storm比较低。 在大多数情况下，单节点Zookeeper集群应该足够了，但是如果您想要故障切换或正在部署大型Storm集群，则可能需要较大的Zookeeper集群。 [这里]("https://www.jianshu.com/p/04b731ecf014")是部署Zookeeper的说明。

有关Zookeeper部署的一些注意事项：

* 在监督下运行Zookeeper是非常重要的，因为Zookeeper是快速失败的，如果遇到任何错误情况，将会退出进程。 在[这里]("http://zookeeper.apache.org/doc/r3.3.3/zookeeperAdmin.html#sc_supervision")看到更多的细节。
* 设置一个cron来压缩Zookeeper的数据和事务日志是至关重要的。 Zookeeper守护进程本身并不这样做，如果你没有设置cron，Zookeeper将很快用完磁盘空间。 在[这里]("http://zookeeper.apache.org/doc/r3.3.3/zookeeperAdmin.html#sc_maintenance")看到更多的细节。

安装依赖
===

* Java 7+
* Python 2.6.6

下载storm
===

```
cd /opt
wget http://mirror.bit.edu.cn/apache/storm/apache-storm-1.1.1/apache-storm-1.1.1.tar.gz
tar -zxvf apache-storm-1.1.1.tar.ge
cd apache-storm-1.1.1
```

配置storm
===

Storm版本包含`conf/storm.yaml`文件，用于配置Storm守护进程。 你可以在这里看到默认的配置值。 `storm.yaml`将覆盖`defaults.yaml`中的任何内容。 有几个配置是强制性的，以获得一个工作集群：

1. `storm.zookeeper.servers`：这是Storm集群的Zookeeper集群中的主机列表。 它应该看起来像这样：

```
storm.zookeeper.servers:
  - "192.168.10.101" #填写zookeeper集群的ip地址或者主机名
  - "192.168.10.102"
  - "192.168.10.103"
```

如果您的Zookeeper集群使用的端口与默认端口(2181)不同，您应该设置`storm.zookeeper.port`。

2. ` storm.local.dir`：Nimbus和Supervisor守护进程需要本地磁盘上的一个目录来存储少量的状态（比如jar，confs等等）。 您应该在每台机器上创建该目录(`mkdir /home/storm`)，给予适当的权限，然后使用此配置填写目录位置。 例如：

```
storm.local.dir: "/home/storm"
```

3. `nimbus.seeds`：工作节点(worker nodes)需要知道哪些机器是主机的候选者，以便下载拓扑结构的jar和conf。 例如：

```
nimbus.seeds: ["192.168.10.101"]
```

4. `supervisor.slots.ports`：对于每个工作机器，使用此配置可以配置在该机器上运行的worker数量。 每个worker使用单个端口接收消息，并且此设置定义哪些端口是打开使用的。 如果你在这里定义了五个端口，那么Storm将会分配多达五个worker在这台机器上运行。 如果你定义了三个端口，Storm最多只能运行三个端口。 默认情况下，此设置被配置为在端口6700,6701,6702和6703上运行4个worker。例如：

```
supervisor.slots.ports:
    - 6700
    - 6701
    - 6702
    - 6703
```

5. Storm提供了一种机制，管理员可以通过配置该机制管理定期运行的脚本，以确定节点是否健康。 管理员可以通过在位于`storm.health.check.dir`中的脚本中执行任何选择检查来确定节点是否处于健康状态。如果脚本检测到节点处于不健康的状态，则必须以字符串ERROR打印一行到标准输出。 supervisor将定期在健康检查目录中运行脚本并检查输出。 如果脚本的输出包含字符串ERROR，如上所述，supervisor将关闭所有workers并退出。
如果supervisor在监督下运行，则可以调用“/ bin / storm node-health-check”来确定是否应该启动supervisor或者该节点是否不健康。

运行状况检查目录位置可以配置为：

```
storm.health.check.dir: "healthchecks"
```

脚本必须具有执行权限。 允许任何给定的健康检查脚本在被标记为失败之前运行的时间由于超时而失败，可以使用以下方法进行配置：

```
storm.health.check.timeout.ms: 5000
```

ok,保存退出，配置就算完成了。

6. 配置外部库和环境变量（可选）
如果您需要外部库或自定义插件的支持，可以将这些jar放入extlib /和extlib-daemon /目录中。 请注意，extlib-daemon /目录存储了仅由守护进程（Nimbus，Supervisor，DRPC，UI，Logviewer）使用的jar，例如HDFS和定制调度库。 因此，用户可以配置两个环境变量STORM_EXT_CLASSPATH和STORM_EXT_CLASSPATH_DAEMON，以包含外部类路径和仅守护进程的外部类路径。

启动守护进程
===

最后一步是启动所有的Storm守护进程。 在监督下运行这些守护进程是非常重要的。 Storm是一个快速失败(fail-fast)系统，意味着只要遇到意外错误，进程就会停止。 Storm的设计可以在任何时候安全停止，并在重新启动过程时正确恢复。 这就是为什么Storm在进程中不保持状态 - 如果Nimbus或Supervisors重新启动，运行的拓扑结构不受影响。 以下是如何运行Storm守护进程：

1. `Nimbus`：在Storm主控节点上运行命令`bin/storm nimbus &`，启动Nimbus后台程序，并放到后台执行。
2. `Supervisor`：在Storm各个工作节点上运行命令`bin/storm supervisor &`。启动Supervisor后台程序，并放到后台执行。 `Supervisor`守护进程负责启动和停止该机器上的工作进程。
3. `UI`： 在Storm主控节点上运行命令`bin/storm ui &`，启动UI后台程序，并放到后台执行。运行Storm UI（一个您可以从浏览器访问的网站，该网站可以对集群和拓扑进行诊断）。 可以通过浏览您的Web浏览器访问UI：`http：// {ui host}：8080`。

通过命令`jps`可以看到相应的进程已经启动起来了。

配置环境变量
===

```
vim /etc/profile
export STORM_HOME=/opt/apache-storm-1.1.1
export PATH=$PATH:$STORM_HOME/bin
#保存退出即可
source /etc/profile #使配置文件立即生效
```

storm常用命令
===

通过执行命令`storm`就可以列出storm的所有命令列表了。

1. jar命令负责把拓扑提交到集群，并执行它，通过StormSubmitter执行主类。

```
storm jar path-to-topology-jar class-with-the-main arg1 arg2 argN
```

path-to-topology-jar是拓扑jar文件的全路径，它包含拓扑代码和依赖的库。 class-with-the-main是包含main方法的类，这个类将由StormSubmitter执行，其余的参数作为main方法的参数。

2. 停用拓扑：

```
storm deactivte topology-name 
```

3. 启动一个停用的拓扑：

```
storm activate topology-name
```

4. 杀死一个拓扑：

```
storm kill topology-name
```

5. 再平衡拓扑(再平衡使你重分配集群任务。这是个很强大的命令。比如，你向一个运行中的集群增加了节点。再平衡命令将会停用拓扑，然后在相应超时时间之后重分配工人，并重启拓扑。)：

```
storm rebalance topology-name
```

END
