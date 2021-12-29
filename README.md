# hadoop2-cluster-docker


利用docker容器模拟hadoop集群的部署。

使用步骤：

1. 在项目根目录下放置hadoop2的部署包，一般情况下，官方包名为 hadoop-x.x.x.tar.gz, 其中xxx为版本号
2. 利用命令`docker build -t hadoop2 --build-arg HADOOP_PACKAGE=hadoop-x.x.x .` 构建一个名为hadoop2的镜像；
3. 修改etc-hadoop文件夹下的配置文件;
4. 使用`docker-compose up` 启动hadoop容器集群；

关于hadoop容器集群的规划在文件docker-compose.yml中:
- hadoop-01: NameNode, DataNode, NodeManager, JobHistoryServer
- hadoop-02: SecondaryNameNode, DataNode, NodeManager
- hadoop-03: ResourceManager, DataNode, NodeManager
- hadoop-04: DataNode, NodeManager
- hadoop-05: DataNode, NodeManager

接下来开始愉快的hadoop之旅吧！😏
