# hadoop2-cluster-docker

## 新增hive
容器启动后，需要检查hive和hadoop使用的guava包的版本是否一致，例如hive使用的guava版本高于hadoop使用的guava版本，那么`rm ${HADOOP_HOME}/share/hadoop/common/lib/guava-*.jar && cp ${HIVE_HOME}/lib/guava-*.jar ${HADOOP_HOME}/share/hadoop/common/lib/`

hive启动

配置
```
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://mysql:3306/yhy</value>
    <description>
      JDBC connect string for a JDBC metastore.
      To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
      For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
    </description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.cj.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>yhy</value>
    <description>Username to use against metastore database</description>
  </property>
  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>my-secret-pw</value>
    <description>password to use against metastore database</description>
  </property>


# 在vi中替换变量
:%s#${system:java.io.tmpdir}#/tmp/javaiotmp#
:%s#${system:user.name}#root#

----- hadoop core-site.xml -----
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
    </property>
```

初始化元数据库：`schematool -initSchema -dbType mysql`

通过hiveserver2, 需要将hive-site.xml文件中的hive.server2.thrift.bind.host属性配置为hadoop01
```Bash
$ hiveserver2                     # 启动hiveserver2服务
$ hive --service hiveserver2 &    # 后台启动hiveserver2服务

$ bin/beeline -u jdbc:hive2://hadoop01:10000 -n root
```
通过metastore, 需要将hive-site.xml文件中的hive.metastore.uris属性配置为thrift://hadoop01:9083
```Bash
$ hive --service metastore &      # 后台启动metastore服务

$ bin/hive                        # 连接metastore
```
---
利用docker容器模拟hadoop集群的部署。

使用步骤：

1. 在项目根目录下放置hadoop2的部署包，一般情况下，官方包名为 hadoop-x.x.x.tar.gz, 其中xxx为版本号
2. 使用脚本`build.sh`构建镜像，脚本内部的命令`docker build -t hadoop2 --build-arg HADOOP_PACKAGE=hadoop-x.x.x .` 构建一个名为hadoop2的镜像，此处记得将版本号换成自己所用的hadoop版本号；
3. 修改etc-hadoop文件夹下的配置文件;
4. 使用`docker-compose up` 启动hadoop容器集群；

关于hadoop容器集群的规划在文件docker-compose.yml中:
- hadoop-01: NameNode, DataNode, NodeManager, ResourceManager, JobHistoryServer
- hadoop-02: SecondaryNameNode, DataNode, NodeManager
- hadoop-03: DataNode, NodeManager

## 当前配置
```
# ----- core-site.xml ----
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop01/</value>
    </property>
</configuration>

# ----- hdfs-site.xml ----
<configuration>
    <property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop01:50070</value>
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop02:50090</value>
    </property>
</configuration>

# ----- yarn-site.xml -----
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop01</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>

# ----- mapred-site.xml -----
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>

# ----- hadoop-env.sh -----
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/hadoop

# ----- slaves -----
hadoop01
hadoop02
hadoop03
```

按照如上规划，接下来应该：
1. 在hadoop-01上执行`hdfs namenode -format`；
2. 在hadoop-01的${HADOOP_HOME}/sbin/目录下，执行`./start-dfs.sh`；
3. 在hadoop-01的${HADOOP_HOME}/sbin/目录下，执行`./start-yarn.sh`；
4. 在hadoop-01的${HADOOP_HOME}/sbin/目录下，执行`./mr-jobhistory-daemon.sh start historyserver`；

接下来开始愉快的hadoop之旅吧！😏
