# Hadoop v2.7 全分布式配置---3节点

## 一. 集群配置
|   节点    |- NameNode     |- SecondaryNN |- DataNode |- ResourceManager |- NodeManager |    
|:------:   |:------:   |:------:   |:------:   |:------:   |:------:   |
|   `ha11.woniu.com`  | **1**   |           |         | **1**   |       |
|   `ha22.woniu.com`  |         | **1**     | **1**   |         |**1**  |
|   `ha33.woniu.com`  |         |           | **1**   |         |**1**  |

------------------------------------------------------------------------

## 二. 步骤概要
* 安装CentOS7虚拟机 (VMWare Workstattion Pro)
* 克隆另外2台虚拟机 
* 免密登录配置
* 完全分布式配置
* 启动完全分布式Hadoop集群
* 测试WordCount程序

------------------------------------------------------------------------

## 三. 安装CentOS7虚拟机 
* 参考另外一个安装手册
* 划重点
    > `安装时，创建一个非root用户， 取名为： hadoop`

    > `为hadoop用户追加管理员权限`： 
    
           # su root 
           # visudo
           hadoop  ALL=(ALL)   ALL  
           #### 追加行当中的间隔为tab， hadoop为当前用户

    > `使用静态IP地址：192.168.37.11`

        * 可以使用VMWare软件直接修改, 或者参考如下命令修改 BOOTPROTO和IPADDR，GATEWAY， DNS1
    
         # ls /etc/sysconfig/network-scripts/ifcfg-en*
         # sudo vi /etc/sysconfig/network-scripts/ifcfg-ens33  ###（默认情况下是 网络连接名称为 ens33）
            TYPE="Ethernet"
            PROXY_METHOD="none"
            BROWSER_ONLY="no"
            BOOTPROTO="none"
            DEFROUTE="yes"
            IPV4_FAILURE_FATAL="no"
            IPV6INIT="yes"
            IPV6_AUTOCONF="yes"
            IPV6_DEFROUTE="yes"
            IPV6_FAILURE_FATAL="no"
            IPV6_ADDR_GEN_MODE="stable-privacy"
            NAME="ens33"
            UUID="fbbde99e-45a3-4ac2-8911-de15a2cf980f"
            DEVICE="ens33"
            ONBOOT="yes"
            IPADDR="192.168.37.11"
            PREFIX="24"
            GATEWAY="192.168.37.2"
            DNS1="192.168.37.2"
            IPV6_PRIVACY="no"

    > `修改本机主机名称:  ha11.woniu.com` 
           
            woniu.com是域名, ha11是主机名， 合在一起使用, 也可以不设置域名
            #sudo vi /etc/hostname

    > `设置Hosts`

        #sudo vi /etc/hosts  ## 追加下面三行
            192.168.37.11 ha11.woniu.com
            192.168.37.22 ha22.woniu.com
            192.168.37.33 ha33.woniu.com  

        >>> Windows宿主机上同样追加：C:/windows/system32/driver/etc/hosts    

    > `关闭防火墙和SELinux`

        #systemctl start firewalld.service    #启动firewall 
        #systemctl stop firewalld.service     #停止firewall 
        #systemctl disable firewalld.service  #禁止firewall开机启动

* 安装sz rz文件传输工具

        #sudo yum install lrzsz

* 卸载自带的OpenJDK
        
        #rpm -qa | grep java  #查看安装的jdk
        #rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.7.b09.el5 #这里jdk要指定为上面查出来的版本

* 安装JDK

        >>>使用rz命令，将JDK复制到/home/hadoop/download目录下
        #sudo tar -zxvf jdk-*** -C /opt/
        #cd /opt/
        #sudo mv jdk*** java
        #sudo chown -R hadoop:hadoop ./java        # 修改文件权限， hadoop:hadoop清修改为当前用户名和组名

* Java环境变量设置

        #sudo vi /etc/profile
        
        >>> 最下方追加下面3行，如下举例，文件夹路径自行修改
            export JAVA_HOME=/opt/java
            export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
            export PATH=$PATH:$JAVA_HOME/bin

        >>> 保存退出，执行如下命令，使其生效
        #source /etc/profile 


* 安装Hadoop

        >>>使用rz命令，将hadoop2.7复制到/home/hadoop/download目录下
        #sudo tar -zxvf ~/下载/hadoop-2.7.0.tar.gz -C /opt/ 
        #cd /opt
        #sudo mv ./hadoop-2.7.0/ ./hadoop            # 将文件夹名改为hadoop
        #sudo chown -R hadoop:hadoop ./hadoop        # 修改文件权限， hadoop:hadoop清修改为当前用户名和组名
        #cd /opt/hadoop
        #./bin/hadoop version						 # 查看版本

* Hadoop环境变量设置

        #sudo vi /etc/profile
        >>> 最下方追加下面行，如下举例，文件夹路径自行修改
            ##### Hadoop Environment Variables
            export HADOOP_HOME=/opt/hadoop
            export HADOOP_INSTALL=$HADOOP_HOME
            export HADOOP_MAPRED_HOME=$HADOOP_HOME
            export HADOOP_COMMON_HOME=$HADOOP_HOME
            export HADOOP_HDFS_HOME=$HADOOP_HOME
            export YARN_HOME=$HADOOP_HOME
            export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
            export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin

        >>> 保存退出，执行如下命令，使其生效
        #source /etc/profile 




------------------------------------------------------------------------

## 四. 克隆另外2台虚拟机 (VMWare workstation Pro)
* 先关闭要复制的虚拟机CentOS
* 右键 -- 管理 -- 完整克隆
* VMWare下，修改网络适配器 MAC地址(点击"生成"按钮)
* 启动后，修改主机名, 参考之前的方法

    `ha22.woniu.com 和 ha33.woniu.com`
* 修改IP地址, 参考之前的方法

    `192.168.37.22 和 192.168.37.33`
   
------------------------------------------------------------------------

## 五. 免密登录配置 
*  在3台机器上创建密钥

        #ssh-keygen -t rsa    ###全部回车执行完毕

* 在3台机器上执行如下命令，将公钥集中到ha11机器上的~/.ssh/authorized_keys

        #ssh-copy-id -i ~/.ssh/id_rsa.pub ha11.woniu.com

* 在ha11机器上把~/.ssh/authorized_keys复制到ha22和ha33上

        #scp ~/.ssh/authorized_keys ha22.woniu.com:~/.ssh/
        #scp ~/.ssh/authorized_keys ha33.woniu.com:~/.ssh/

        #scp ~/.ssh/known_hosts ha22.woniu.com:~/.ssh/
        #scp ~/.ssh/known_hosts ha22.woniu.com:~/.ssh/
------------------------------------------------------------------------

## 六. 完全分布式配置

### 0. /opt/hadoop/etc/hadoop/hadoop-env.sh

        #vi /opt/hadoop/etc/hadoop/hadoop-env.sh
        >>> 设置JAVA_HOME
        export JAVA_HOME=/opt/java

### 1. /opt/hadoop/etc/hadoop/core-site.xml

        <configuration>
                <property>
                        <name>fs.defaultFS</name>
                        <value>hdfs://ha11.woniu.com:9000</value>
                </property>
                <!-- 指定hadoop临时目录,自行创建 -->
                <property>
                        <name>hadoop.tmp.dir</name>
                        <value>/opt/data/full/hadoop</value>
                </property>
        </configuration>

### 2. /opt/hadoop/etc/hadoop/hdfs-site.xml

        <configuration>
	        <!-- 将备份数修改为2，小于等于当前datanode数目即可-->
                <property>
                        <name>dfs.replication</name>
                        <value>2</value>
                </property>
                        <!-- 将secondary namenode改为hadoop2-->
                <property>
                        <name>dfs.namenode.secondary.http-address</name>
                        <value>ha22.woniu.com:50090</value>
                </property>
	        <property>
                        <name>dfs.namenode.name.dir</name>
		        <value>file://${hadoop.tmp.dir}/dfs/name</value>
	        </property>
	        <property>
		        <name>dfs.namenode.data.dir</name>
		        <value>file://${hadoop.tmp.dir}/dfs/data</value>
	        </property>
	        <property>
                        <name>dfs.permissions.enabled</name>
                        <value>false</value>
	        </property>
        </configuration>

### 3. /opt/hadoop/etc/hadoop/yarn-site.xml

        <configuration>
                <!-- 添加了yarn.resourcemanager.hostname 属性-->
                <property>
                        <name>yarn.resourcemanager.hostname</name>
                        <value>ha11.woniu.com</value>
                </property>
                
                <property>  
                        <name>yarn.nodemanager.aux-services</name>  
                        <value>mapreduce_shuffle</value>  
                </property>  
                <!-- 添加了yarn.nodemanager.auxservices.mapreduce.shuffle.class属性-->
                <property>
                        <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
                        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
                </property>
        </configuration>


### 4. /opt/hadoop/etc/hadoop/mapred-site.xml

        <configuration>
                <property>
                        <name>mapreduce.framework.name</name>
                        <value>yarn</value>
                </property>
                <property>
                        <name>mapreduce.jobhistory.address</name>
                        <value>ha11.woniu.com:10020</value>
                </property>
                <property>
                        <name>mapreduce.jobhistory.address</name>
                        <value>ha11.woniu.com:19888</value>
                </property>
        </configuration>

### 5. /opt/hadoop/etc/hadoop/slaves
        ## DataNode节点， 输入如下

        ha22.woniu.com
        ha33.woniu.com


### 6. 分发hadoop配置到所有节点

        #cd /opt/hadoop/etc/hadoop
        #scp hadoop-env.sh core-site.xml hdfs-site.xml yarn-site.xml mapred-site.xml slaves ha22.woniu.com:`pwd`
        #scp hadoop-env.sh core-site.xml hdfs-site.xml yarn-site.xml mapred-site.xml slaves ha33.woniu.com:`pwd`


------------------------------------------------------------------------

## 七. 启动完全分布式Hadoop集群
0. 创建hadoop tmp dir
   
        #cd /opt
        #sudo mkdir -p ./data/full/hadoop/
        #sudo chown -R hadoop:hadoop ./data

1. 在ha11上重新格式化namenode

        #hdfs namenode -format

2. 在ha11上启动HDFS

        #start-dfs.sh

3. 在ha11上启动yarn

        #start-yarn.sh

4. 在各节点上查看进程

        #jps

------------------------------------------------------------------------

## 八. 测试WordCount程序
------------------------------------------------------------------------




