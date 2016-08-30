Pinpoint集群安装
======
1. 环境
	centos 6.6
    软件：
    ```javascript
        [root@localhost pinpoint]# cd /data/package/
        [root@localhost package]# ll
        -rw-r--r--. 1 root root   8866946 Feb 22 11:34 apache-tomcat-7.0.65.tar.gz
        -rw-r--r--. 1 root root 196015975 Feb 19 12:24 hadoop-2.6.4.tar.gz
        -rw-r--r--. 1 root root 103847513 Feb 19 12:24 hbase-1.0.3-bin.tar.gz
        -rw-r--r--. 1 root root 153512879 Feb 19 17:11 jdk-7u79-linux-x64.gz
        -rw-r--r--. 1 root root   6455604 Feb 19 12:24 pinpoint-agent-1.5.1.tar.gz
    ```

1. **hadoop+hbase，此次环境基于伪分布式**
	1. **配置主机名**
	```javascript
    	vim /etc/hosts
        192.168.1.224   apm01
        [root@localhost hadoop]# hostname amp01
    ```

    1. **做好跟本机互信**
    ```java
    	[root@localhost ~]# ssh-keygen
		[root@localhost ~]# ssh-copy-id -i apm01
    ```

    1. **hadoop配置**
    ```java
    	cd /data/pinpoint/hadoop-2.6.4/etc/hadoop
        vim hadoop-env.sh
        export JAVA_HOME=/usr/local/jdk
        vim hdfs-site.xml
        <configuration>
            <property>
                <name>dfs.replication</name>
                <value>1</value>
            </property>
        </configuration>
        # vim hdfs-site.xml
        <configuration>
            <property>
                <name>fs.defaultFS</name>
                <value>hdfs://apm01:9000</value>
            </property>
        </configuration>

        vim core-site.xml
        <configuration>
            <property>
                <name>fs.defaultFS</name>
                <value>hdfs://apm01:9000</value>
            </property>
        </configuration>

    	# cat masters
        apm01
        # cat slaves
        apm01

        # cp mapred-site.xml.template mapred-site.xml
        cat mapred-site.xml

        <configuration>
            <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
            </property>
        </configuration>

        /data/pinpoint/hadoop-2.6.4/sbin
        # ./start-dfs.sh
        # ./start-yarn.sh

        [root@localhost sbin]# jps
        19453 NameNode
        19906 ResourceManager
        20000 NodeManager
        19731 SecondaryNameNode
        19572 DataNode

        http://192.168.1.224:50070/dfshealth.html#tab-overview
    ```

    1. **配置hbase**
    ```java
    	cd /data/pinpoint/hbase-1.0.1/conf
        vim hbase-env.sh
        export JAVA_HOME=/usr/local/jdk

        #vim hbase-site.xml
        <configuration>
            <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value></property><property>
                <name>hbase.rootdir</name>
                <value>hdfs://apm01:9000/hbase</value>
            </property>
            <property>
                <name>dfs.replication</name>
                <value>1</value>
            </property>
            <property>
                <name>zookeeper.znode.parent</name>
                <value>/hbase</value>
            </property>
            <property>
                <name>hbase.zookeeper.quorum</name>
                <value>apm01</value>
            </property>
        </configuration>

        #bin/start-hbase.sh
        http://192.168.1.224:16010

        导入表，到
        https://github.com/naver/pinpoint/tree/master/hbase/scripts

        导入表结构会提示一些错误，忽略，不知道为啥
        # bin/hbase shell scripts/hbase-create.hbase
        # bin/hbase shell scripts/hbase-flush-table.hbase
    ```
1. **配置collection，web,agent都在同一台机器上**
	1. **collection**
	```java
    	# cp pinpoint-collector-1.5.1.war  tomcat7-collector/webapps/
        启动把war解压后，再停止
        # vim WEB-INF/classes/hbase.properties   根据自己的情况来修改
        hbase.client.host=apm01
        hbase.client.port=2181

        查看有没有监听9994端口
        [root@localhost ROOT]# netstat -tanlp |grep 9994
        tcp        0      0 :::9994                     :::*                        LISTEN      27437/java
    ```

	1. **web 注意修改端口**
	```java
    	# cp pinpoint-web-1.5.1.war  tomcat7-web/webapps/
        #启动tomcat,把war解压后，把tomcat停止
        cd  /data/pinpoint/tomcat7-web/webapps/pinpoint-web-1.5.1
        # \cp -r * /data/pinpoint/tomcat7-web/webapps/ROOT/
        # vim WEB-INF/classes/hbase.properties   根据自己的情况来修改
        hbase.client.host=apm01
        hbase.client.port=2181
    ```

    1. **部署agent和应用，再开启一台tomcat，添加修改agent的配置，端口要与collection的端口保持一致**
    ```java
    	#cd /data/pinpoint/pinpoint-agent-1.5.1
        # vim pinpoint.config
        profiler.collector.ip=apm01

        # placeHolder support "${key}"
        profiler.collector.span.ip=${profiler.collector.ip}
        profiler.collector.span.port=9996

        # placeHolder support "${key}"
        profiler.collector.stat.ip=${profiler.collector.ip}
        profiler.collector.stat.port=9995

        # placeHolder support "${key}"
        profiler.collector.tcp.ip=${profiler.collector.ip}
        profiler.collector.tcp.port=9994

        # vim tomcat7-app/bin/catalina.sh
        CATALINA_OPTS="$CATALINA_OPTS -javaagent:/data/pinpoint/pinpoint-agent-1.5.1/pinpoint-bootstrap-1.5.1.jar"
        CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=PPMoneyActicity"
        CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=PPMoneyActicity"

        http://192.168.1.224:8282/
    ```
