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

        Windows宿主机上同样追加：C:/windows/system32/driver/etc/hosts    

* 安装sz rz文件传输工具

        #sudo yum install lrzsz

* 

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
------------------------------------------------------------------------

## 六. 完全分布式配置
------------------------------------------------------------------------

## 七. 启动完全分布式Hadoop集群
------------------------------------------------------------------------

## 八. 测试WordCount程序
------------------------------------------------------------------------


## 配置免密码登录，生成各种密码文件
```查看ssh状态，如果不存在则安装
rpm -qa | grep ssh 
sudo yum install openssh-clients
sudo yum install openssh-server
```

```
ssh localhost
#### 此时会有如下提示(SSH首次登陆提示)，输入 yes 。
#### 然后按提示输入密码 hadoop，这样就登陆到本机了。
exit                           # 退出刚才的 ssh localhost
cd ~/.ssh/                     # 若没有该目录，请先执行一次ssh localhost
ssh-keygen -t rsa              # 会有提示，都按回车就可以
cat id_rsa.pub >> authorized_keys  # 加入授权
chmod 600 ./authorized_keys    # 修改文件权限
#### 此时再用 ssh localhost 命令，无需输入密码就可以直接登陆了
exit 
```

## 安装Java
### 卸载openjdk
rpm -qa | grep java  #查看安装的jdk
rpm -e --nodeps java-1.6.0-openjdk-1.6.0.0-1.7.b09.el5 #这里jdk要指定为上面查出来的版本
### 下载SUN Linux的jdk， 使用tar.gz的就可以
sudo tar -zxvf jdk-*** -C /opt/
cd /opt/
sudo mv jdk*** java
###  配置java环境变量
vi /etc/profile 
### 最后面追加下面3行，如下举例，文件夹路径自行修改
```
export JAVA_HOME=/opt/java
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
export PATH=$PATH:$JAVA_HOME/bin
```
### 执行如下命令，使其生效
source /etc/profile


## 安装Hadoop
### 下载hadoop-*.*.*.tar.gz
```
sudo tar -zxvf ~/下载/hadoop-2.6.0.tar.gz -C /opt/ 
cd /opt
sudo mv ./hadoop-2.6.0/ ./hadoop            # 将文件夹名改为hadoop
sudo chown -R hadoop:hadoop ./hadoop        # 修改文件权限， hadoop:hadoop清修改为当前用户名
cd /opt/hadoop
./bin/hadoop version						 # 查看版本
```
###  配置Hadoop环境变量
sudo vi /etc/profile
```
##### Hadoop Environment Variables
export HADOOP_HOME=/opt/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
```

### 执行如下命令，使其生效
source /etc/profile

## 安装ZooKeeper
sudo tar -zxvf ~/download/zookeeper-3.4.12.tar.gz -C /opt/ 
cd /opt
sudo mv ./zookeeper-3.4.12/ ./zookeeper            # 将文件夹名改为zookeeper
###  配置ZooKeeper环境变量
sudo vi /etc/profile
##### ZooKeeper Environment Variables
export ZOOKEEPER_HOME=/opt/zookeeper
export PATH=$PATH:$ZOOKEEPER_HOME/bin

### 执行如下命令，使其生效
source /etc/profile

## 配置zookeeper集群，修改配置文件
```
 tickTime=2000
 dataDir=/opt/data/zookeeper # 数据存放路径
 clientPort=2181
 initLimit=5
 syncLimit=2
 server.1=ha11.woniu.com:2888:3888
 server.2=ha22.woniu.com:2888:3888
 server.3=ha33.woniu.com:2888:3888 
 ```
 ### 创建节点ID，在配置的 dataDir 路径中添加myid文件 （延迟到克隆后处理）######




