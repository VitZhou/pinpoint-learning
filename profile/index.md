# Pinpoint介绍

	Pinpoint 是用 Java 编写的 APM（应用性能管理）工具，用于大规模分布式系统。在 Dapper(dapper.docx) 之后，Pinpoint 提供了一个解决方案，以帮助分析系统的总体结构以及分布式应用程序的组件之间是如何进行数据互联的。

1. **特点**
    1. 安装agent是无侵入的
    1. 对性能的影响非常小（只增加约3%资源利用率）

1. **支持模块**
    1. JDK6+
    1. Tomcate 6/7/8,Jetty 9
    1. Spring, Spring Boot
    1. Apache Http Client 3.x/4.x,JDK HttpConnector,GoogleHttpClinet,OkHttpClient,NingAsyncHttpClient
    1. Thrift Cliet,Thirft Service
    1. MySql,Oracle,MSSQL,CUBRID,DBCP,POSTGRESQL
    1. Arcus,Memcached,Redis
    1. IBATIS,MyBatis
    1. Gson,Jackson,Json Lib
    1. Log4j,Logback

1. **构建要求**
    1. JDK 6，7，8安装
    1. Maven 3.2.x+ 安装
    1. 环境变量设置：JAVA_6_HOME=JDK6目录,JAVA_7_HOME=JDK7目录，JAVA_8_HOME=JDK8目录

1. **架构**
	![1.png](..\profile\1.png)

1.	**作用**
    1. 服务器地图(ServerMap)通过可视化分布式系统的模块和他们之间的相互联系来理解系统拓扑。点击某个节点会展示这个模块的详情，比如它当前的状态和请求数量。

    1. 实时活动线程图表(Realtime Active Thread Chart)实时监控应用内部的活动线程。

    1. 请求/应答分布图表(Request/Response Scatter Chart)长期可视化请求数量和应答模式来定位潜在问题。通过在图表上拉拽可以选择请求查看更多的详细信息。
	![2.png](..\profile\2.png)

	1. 调用栈(CallStack)在分布式环境中为每个调用生成代码级别的可视图，在单个视图中定位瓶颈和失败点。
	![3.png](..\profile\3.png)

    1. 巡查(Inspector)查看更多细节上的应用，如CPU使用率、内存/垃圾收集，TPS和JVM参数。
    ![4.png](..\profile\4.png)
