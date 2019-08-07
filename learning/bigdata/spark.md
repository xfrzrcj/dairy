# **SPARK LEARNING**

## 1、Spark架构

如图：

![spark架构](../../pic/1004194-20160829174157699-296881431.png)

- Cluster Manager：在standalone模式中即为Master主节点，控制整个集群，监控worker。在YARN模式中为资源管理器。

- Worker节点：从节点，负责控制计算节点，启动Executor或者Driver。

- Driver： 运行Application 的main()函数

- Executor：执行器，是为某个Application运行在worker node上的一个进程

## 4、运行流程

如图：

![运行流程](../../pic/1004194-20160830094200918-1846127221.png)



## 3、Spark 的任务层次

+ job：由动作算子作为分界线，将整个任务分成多个job。

+ stage:每个job由多个stage组成，一个job以宽窄依赖作为分界，即shuffle作为分界分成多个stage.

+ task:多个task组成一个stage。一个task表示被送到Executor上的工作单元。



参考：

1、[基本架构及原理](https://www.cnblogs.com/cxxjohnson/p/8909578.html)

