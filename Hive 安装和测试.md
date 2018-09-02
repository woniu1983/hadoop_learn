# Hive+MySql部署（远程Mysql）

## 一. 安装

        #sudo tar -zxvf apache-hive-1.2.2-bin.tar.gz -C /opt/
        #cd /opt/ 
        #sudo mv apache-hive-1.2.2-bin* hive
        #sudo chown -R hadoop:hadoop ./hive

------------------------------------------------------------------------

## 二. 环境变量配置

        #sudo vi /etc/profile
        --------------------------------------------------------------
        export HIVE_HOME=/opt/hive
        export PATH=$PATH:$HIVE_HOME/bin
        --------------------------------------------------------------
        #source /etc/profile   //立即生效

        #hive --version         //检查安装版本

------------------------------------------------------------------------

## 三. 在HDFS上创建文件夹
  首先启动Hadoop

  $ $HADOOP_HOME/bin/hadoop dfs -mkdir       /tmp
  $ $HADOOP_HOME/bin/hadoop dfs -mkdir       /user/hive/warehouse
  $ $HADOOP_HOME/bin/hadoop dfs -chmod g+w   /tmp
  $ $HADOOP_HOME/bin/hadoop dfs -chmod g+w   /user/hive/warehouse

## 四. Hive设置
* 备份配置文件即可

        >>进入$HIVE_HOME/conf目录
        #cd /opt/hive/conf
        #mv hive-env.sh.template hive-env.sh
        #vi hvie-env.sh

        >>设置下Hadoop的路径和Hive conf的路径
        # Set HADOOP_HOME to point to a specific hadoop install directory
        HADOOP_HOME=/opt/hadoop
        # Hive Configuration Directory can be controlled by:
        export HIVE_CONF_DIR=/opt/hive/conf


## 五. 测试Hive

* 执行hive命令
   #hive
   hive>  create database if not exists db01;
   hive>  show databases;
   hive>  use db01;
   hive>  create table if not exists student ( num int, name string);
   hive>  show tables;






------------------------------------------------------------------------
## 六. 安装mysql
* 到目前为止， hive安装完毕，但是hive默认使用的是derby数据库引擎，这个derby不支持并发访问（会出现异常）
* 安装mysql数据库作为derby的替代
        >> 查询当前是否安装了Mysql
        #rpm -qa | grep mysql
        >>安装方法
        #yum install mysql-server

### yum 方式安装
* 获取 yum repository ： https://dev.mysql.com/downloads/repo/yum/ 

        >> 如果是CentOS7 x64的选择：Red Hat Enterprise Linux 7 / Oracle Linux 7 (Architecture Independent), RPM Package
        >> 可以手动下载，也可以在CentOS7上使用wget命令下载
        >> shell> wget https://dev.mysql.com/get/mysql80-community-release-el7-1.noarch.rpm
* 更新yum repo

        | shell> sudo rpm -Uvh mysql80-community-release-el7-1.noarch.rpm 
        | shell> sudo yum repolist all | grep mysql  ###查询当前mysql的版本
                ```
                mysql-cluster-7.5-community/x86_64 MySQL Cluster 7.5 Community       禁用
                mysql-cluster-7.5-community-source MySQL Cluster 7.5 Community - Sou 禁用
                mysql-cluster-7.6-community/x86_64 MySQL Cluster 7.6 Community       禁用
                mysql-cluster-7.6-community-source MySQL Cluster 7.6 Community - Sou 禁用
                mysql-connectors-community/x86_64  MySQL Connectors Community        启用:    63
                mysql-connectors-community-source  MySQL Connectors Community - Sour 禁用
                mysql-tools-community/x86_64       MySQL Tools Community             启用:    69
                mysql-tools-community-source       MySQL Tools Community - Source    禁用
                mysql-tools-preview/x86_64         MySQL Tools Preview               禁用
                mysql-tools-preview-source         MySQL Tools Preview - Source      禁用
                mysql55-community/x86_64           MySQL 5.5 Community Server        禁用
                mysql55-community-source           MySQL 5.5 Community Server - Sour 禁用
                mysql56-community/x86_64           MySQL 5.6 Community Server        禁用
                mysql56-community-source           MySQL 5.6 Community Server - Sour 禁用
                mysql57-community/x86_64           MySQL 5.7 Community Server        禁用
                mysql57-community-source           MySQL 5.7 Community Server - Sour 禁用
                mysql80-community/x86_64           MySQL 8.0 Community Server        启用:    33
                mysql80-community-source           MySQL 8.0 Community Server - Sour 禁用
                ```
* 如果你需要使用的是5.x版本，禁用默认的mysql8
        | shell> sudo yum -y install yum-utils          ###安装yum的其他命令工具
        | shell> sudo yum-config-manager --disable mysql80-community
        | shell> sudo yum-config-manager --enable mysql57-community
        | shell> sudo yum repolist all | grep mysql  ###再次查询当前启用的mysql的版本

* 开始安装MySQL 
        | shell> sudo yum install mysql-community-server

* 查看安装结果
        shell> rpm -qa | grep mysql
                mysql80-community-release-el7-1.noarch
                mysql-community-common-5.7.23-1.el7.x86_64
                mysql-community-client-5.7.23-1.el7.x86_64
                mysql-community-server-5.7.23-1.el7.x86_64
                mysql-community-libs-5.7.23-1.el7.x86_64
                mysql-community-libs-compat-5.7.23-1.el7.x86_64

* 启动MySQL服务（CentOS7）
        shell> sudo systemctl start mysqld.service
        shell> sudo systemctl status mysqld.service   ### 检查启动状态

* 初始化MySQL
        | 从5.7版本开始，Mysql默认初始化完成，并给root用户赋值一个随机的密码，用如下命令查看
        | shell> sudo grep 'temporary password' /var/log/mysqld.log
                2018-08-08T13:10:28.194933Z 1 [Note] A temporary password is generated for root@localhost: 3p5QMqi(ySjT
        | 如下命令开始修改密码
        | shell> mysql -uroot -p     
        | mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Mysql111!';  ## 密码必须要有大小写数字和特殊字符，长度>=8

* 授权
        | mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Mysql111!' WITH GRANT OPTION;
        | mysql> FLUSH PRIVILEGES;   ###刷新授权命令

------------------------------------------------------------------------
## 七. 配置Hive连接mysql
* 将mysql的连接jar包拷贝到$HIVE_HOME/lib目录下
* 配置hive-site.xml
        | cd /opt/hive/conf
        | cp hive-default.xml.template  hive-site.xml
        | vim hive-site.xml
                <?xml version="1.0" encoding="UTF-8" standalone="no"?>
                <configuration>
                        <property>
                        <name>javax.jdo.option.ConnectionURL</name>
                        <value>jdbc:mysql://ha33.woniu.com:3306/hive?createDatabaseIfNotExist=true</value>
                        </property>
                        <property>
                        <name>javax.jdo.option.ConnectionDriverName</name>
                        <value>com.mysql.jdbc.Driver</value>
                        </property>
                        <property>
                        <name>javax.jdo.option.ConnectionUserName</name>
                        <value>root</value>
                        </property>
                        <property>
                        <name>javax.jdo.option.ConnectionPassword</name>
                        <value>Mysql111!</value>
                        </property>
                </configuration>
* 先启动hadoop或者启动HA集群
* 启动hive
        * Hive2.1的启动需要先执行schematool命令进行初始化， 
        * 否则执行hive下面的sql脚本会失败：Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
        > schematool -dbType mysql -initSchema 

        * 执行hive命令
        #hive
        hive>  create database if not exists db01;
        hive>  show databases;
        hive>  use db01;
        hive>  create table if not exists student ( num int, name string);
        hive>  show tables;

        
------------------------------------------------------------------------
* 参考： https://www.cnblogs.com/biehongli/p/7693598.html
        https://www.cnblogs.com/xieyulin/p/7050545.html
        https://blog.csdn.net/zhihaoma/article/details/48578589  （远程sql模式）

------------------------------------------------------------------------

------------------------------------------------------------------------
## 八. UDF
* 用户自定义函数
        追加到Hive中，用来实现用户自定义的功能。
* 配置IDE(Eclipse) 
        pom (maven)
        hive-site.xml
* 继承类 org.apache.hadoop.hive.ql.exec.UDF
* 定义方法并填充自定义逻辑
        public Object evaluated(Object args) ;
* 导出jar
* 关联jar
        add jar localpath;
* 创建临时函数
        //本地
        hive> create temporary function $函数名 as '$packagename.$calssname' ;

        // HDFS上临时函数
        hive> create temporary function $函数名 as '$packagename.$calssname' using jar 'hdfs://ha11.woniu.com:8020/udf.jar';
        
        //永久函数的话
        ** 将jar文件添加到hive的环境变量中 
        ** 然后编译Hive的源码

        // 伪永久方式
        // TODO

* 执行
        hive> show functions; // 查找创建的UDF函数



------------------------------------------------------------------------
## 九. hiveserver2
* 官方文档 
https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients

* 启动hiveserver2
        | bin/hiveserver2

* shell上尝试连接
        | bin/beeline -u jdbc:hive2://ha11.woniu.com:10000 -n root -p 123456
        | 这里默认端口是 10000， 用户名-n 和 密码-p 随意写
        | 注意是： jdbc:hive2  不是 hive

* Eclipse IDE编写连接hive2, 参考官方文档

* 正则处理
        | https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-ApacheWeblogData 

* 常见错误
- org.apache.hive.service.cli.HiveSQLException: Error while compiling statement: FAILED: ParseException line 1:11 extraneous input ';' expecting EOF near '<EOF>'
  原因： sql语句结尾加了';', 去掉运行即可

- 


