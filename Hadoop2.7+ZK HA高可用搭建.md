# Hadoop v2.7 全分布式 + ZooKeeper HA 高可用配置---3节点

## 一. 集群配置
|   节点    |- ZooKeeper |- ZKFC |- JourNode |- NameNode |- DataNode |- ResourceManager |- NodeManager |    
|:------:   |:------:   |:------:   |:------:   |:------:   |:------:   |:------:   |:------:   |
|   `ha11.woniu.com`  | **1** | **1** | **1** | **1** |      |**1** ||
|   `ha22.woniu.com`  | **1** | **1** | **1** | **1** |**1** ||**1** |
|   `ha33.woniu.com`  | **1** |       | **1** |       |**1** ||**1** |

------------------------------------------------------------------------

## 二. 步骤概要
* 备份上次设置的Hadoop完全分布式环境
* 配置确认所有机器互相ssh免密 
* ha11上配置zookeeper
* ha11上配置hadoop HA高可用
* 第一次启动HA
* 常规启动HA
* 测试WordCount程序

------------------------------------------------------------------------

## 三. 备份上次设置的Hadoop完全分布式环境
* 备份配置文件即可

        >>进入$HADOOP_HOME/etc/目录
        #cd /opt/hadoop/etc/
        #cp -r hadoop/ hadoop-full 



------------------------------------------------------------------------

## 四. 配置确认所有机器互相ssh免密
*  上次已经配置好了
*  原理是： 

     > 在所有机器上生成密钥

     > 然后复制所有的公钥到一台机器A上（ssh-copy-id）

     > 然后所有的公钥都会汇集到A上的~/.ssh/authorized_keys

     > 之后把A上的~/.ssh/authorized_keys复制到其他机器~/.ssh/

     > A上的~/.ssh/known_hosts文件也复制到其他机器~/.ssh/

     > 这样就可以实现个机器互相免密了


------------------------------------------------------------------------

## 五. ha11上配置zookeeper
* 下载zookeeper，复制到ha11上
* 解压缩安装

        #sudo tar -zxvf zookeeper-**.tar.gz -C /opt/
        #cd /opt/ 
        #sudo mv zook* zookeeper

* 配置环境变量

        #sudo vi /etc/profile
        --------------------------------------------------------------
        export ZOOKEEPER_PREFIX=/opt/test/zookeeper
        export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_PREFIX/bin:$HADOOP_PREFIX/sbin:$ZOOKEEPER_PREFIX/bin
        export ZOO_LOG_DIR=/opt/data/ha/zk/logs
        --------------------------------------------------------------

* 环境变量复制到其他机器

        #sudo scp /etc/profile ha22.woniu.com:/etc/
        #sudo scp /etc/profile ha33.woniu.com:/etc/
        #source /etc/profile  ## ha11, ha22, ha33上都执行

* 配置zoo.cfg

        #cd /opt/zookeeper/conf
        #cp zoo_sample.cfg zoo.cfg
        #vi zoo.cfg
        -------------------------------------------
                # 配置zookeeper数据存放目录
                dataDir=/opt/data/ha/zk
                # 设置zookeeper位置信息
                server.1=ha11.woniu.com:2888:3888
                server.2=ha22.woniu.com:2888:3888
                server.3=ha33.woniu.com:2888:3888
        -------------------------------------------

* 设置zookeeper节点对应的ID: myid

        进入zookeeper数据存放目录
        # cd /opt/data/
        # mkdir ha
        # cd ha
        # mkdir zk 
        # cd /opt/data/ha/zk
        # echo 1 > myid
        
        在ha22和ha33上参考如上进行操作:
        # echo 2 > myid   ## ha22
        # echo 3 > myid   ## ha33

* 将zookeeper目录分发到其他节点上

        # sudo scp -r /opt/zookeeper ha22.woniu.com:/opt/
        # sudo chown -R hadoop:hadoop ./zookeeper

        # sudo scp -r /opt/zookeeper ha33.woniu.com:/opt/
        # sudo chown -R hadoop:hadoop ./zookeeper


*  验证zookeeper是否安装成功

        在三个节点上执行
        # zkServer.sh start
        
        全部执行完之后，然后在各节点执行
        # zkServer.sh status
        当出现FOLLOWER和LEADER，说明启动成功

* 使用zookeeper客户端

        >> 进入zookeeper客户端
        # zkCli.sh

        >> 退出zookeeper客户端
        # quit

* 停止zookeeper服务

        #zkServer.sh stop



## 六. ha11上配置hadoop HA高可用

* /opt/hadoop/etc/hadoop/core-site.xml

        <configuration>
                <!--设置fs.defaultFS为nameservices的逻辑主机名-->
                <property>
                        <name>fs.defaultFS</name>
                        <value>hdfs://mycluster</value>
                        </property>
                        <!--设置zookeeper数据存放目录-->
                <property>
                        <name>hadoop.tmp.dir</name>
                        <value>/opt/data/ha/hadoop/tmp</value>
                </property>
                <!--设置zookeeper位置信息-->
                <property>
                        <name>ha.zookeeper.quorum.mycluster</name>
                        <value>ha11.woniu.com:2181,ha22.woniu.com:2181,ha33.woniu.com:2181</value>
                </property>
        </configuration>

* /opt/hadoop/etc/hadoop/hdfs-site.xml
        
  > 删除secondary的配置信息
  
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>hadoop2:50090</value>
        </property>
        
  > 将原有hdfs-site.xml配置替换为如下内容

        <configuration>
                <property>
                <name>dfs.replication</name>
                <value>2</value>
                </property>
                <!--定义nameservices逻辑名称-->
                <property>
                <name>dfs.nameservices</name>
                <value>mycluster</value>
                </property>
                <!--映射nameservices逻辑名称到namenode逻辑名称-->
                <property>
                <name>dfs.ha.namenodes.mycluster</name>
                <value>nn1,nn2</value>
                </property>
                <!--映射namenode逻辑名称到真实主机名称(RPC)-->
                <property>
                <name>dfs.namenode.rpc-address.mycluster.nn1</name>
                <value>ha11.woniu.com:8020</value>
                </property>
                <!--映射namenode逻辑名称到真实主机名称(RPC)-->
                <property>
                <name>dfs.namenode.rpc-address.mycluster.nn2</name>
                <value>ha22.woniu.com:8020</value>
                </property>
                <!--映射namenode逻辑名称到真实主机名称(HTTP)-->
                <property>
                <name>dfs.namenode.http-address.mycluster.nn1</name>
                <value>ha11.woniu.com:50070</value>
                </property>
                <!--映射namenode逻辑名称到真实主机名称(HTTP)-->
                <property>
                <name>dfs.namenode.http-address.mycluster.nn2</name>
                <value>ha22.woniu.com:50070</value>
                </property>
                <!--配置journalnode集群位置信息及目录-->
                <property>
                <name>dfs.namenode.shared.edits.dir</name>
                <value>qjournal://ha11.woniu.com:8485;ha22.woniu.com:8485;ha33.woniu.com:8485/mycluster</value>
                </property>
                <property>
                <name>dfs.journalnode.edits.dir</name>
                <value>/opt/data/ha/hadoop/jn</value>
                </property>
                <!--配置故障切换实现类-->
                <property>
                <name>dfs.client.failover.proxy.provider.mycluster</name>
                <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
                </property>
                <!--指定切换方式为SSH免密钥方式-->
                <property>
                <name>dfs.ha.fencing.methods</name>
                <value>sshfence</value>
                </property>
                <property>
                <name>dfs.ha.fencing.ssh.private-key-files</name>
                <value>/home/hadoop/.ssh/id_rsa</value>
                </property>
                <!--设置自动切换-->
                <property>
                <!--<name>dfs.ha.automatic-failover.enabled.mycluster</name>--> <!--start-dfs.sh 命令不识别mycluster-->
                <name>dfs.ha.automatic-failover.enabled</name>
                <value>true</value>
                </property>
        </configuration>



*  分发hadoop配置到所有节点

        #cd /opt/hadoop/etc/hadoop
        #scp hdfs-site.xml core-site.xml ha22.woniu.com:`pwd`
        #scp hdfs-site.xml core-site.xml ha33.woniu.com:`pwd`


------------------------------------------------------------------------

## 七. 第一次启动HA
0. ha11,ha22,ha33 上分别启动zookeeper
   
        # zkServer.sh start
        # zkServer.sh start
        # zkServer.sh start

        查看 
        # zkServer.sh status
        # jps

1.  ha11,ha22,ha33 上启动journalnode, 奇数个, 且>=3

        # hadoop-daemon.sh start journalnode
        # hadoop-daemon.sh start journalnode
        # hadoop-daemon.sh start journalnode

        查看 
        # jps

2.  在ha11上格式化namenode

        # hdfs namenode -format

3.  在ha11上启动namenode
        
        # hadoop-daemon.sh start namenode
        # jps

4.  在ha22（另一台namenode）上同步ha11的CID等信息

        # hdfs namenode -bootstrapStandby
        # jps

5.  在ha11上启动其他服务

        # start-dfs.sh

6.  在ha11上格式化zookeeper

        # hdfs zkfc -formatZK

7.  在所有namenode上（ha11, ha22）启动ZKFC

        # hadoop-daemon.sh start zkfc

7.  在ha11,ha22上使用zkCli.sh查看格式化结果

        # zkCli.sh
        # zkCli.sh

------------------------------------------------------------------------

##  八. 常规启动HA
1.  启动zookeeper

        >> ha11,ha22,ha33 上分别启动zookeeper
        # zkServer.sh start
        # zkServer.sh start
        # zkServer.sh start

2. 启动hdfs集群

        >> 在ha11上启动整个集群start-dfs.sh
        # start-dfs.sh        

3. 启动yarn

        >> 在ha11上启动yarn
        # start-yarn.sh

------------------------------------------------------------------------

##  九. 停止HA + HADOOP
> 下面是停止Hadoop的HA集群的流程：

1. 第一步，在Ha11机器上停止HDFS：
[root@hadoop01 ~]# sbin/stop-dfs.sh

2. 第二步，在Ha11机器上停止YARN：
[root@hadoop01 ~]# sbin/stop-yarn.sh

3. 第三步，在Ha11机器上单独停止ResourceManager：
[root@hadoop02 ~]# sbin/yarn-daemon.sh stop resourcemanager

4. 第四步，在Ha11机器上停止Zookeeper：
[root@hadoop01 ~]# zkServer.sh stop

5. 第五步，在Ha22机器上停止Zookeeper：
[root@hadoop02 ~]# zkServer.sh stop

6. 第六步，在Ha33机器上停止Zookeeper：
[root@hadoop03 ~]# zkServer.sh stop



## 十. 测试WordCount程序

        # cd /opt/test/hadoop-2.6.5/share/hadoop/mapreduce/
        # hdfs dfs -mkdir test
        # hdfs dfs -put test.txt /test
        # hadoop jar hadoop-mapreduce-examples-2.6.5.jar wordcount /test/test.txt /output

        >> 等待执行完毕， 查看输出结果
        # hdfs dfs -text /output/part-*

------------------------------------------------------------------------




