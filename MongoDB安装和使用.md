# MongoDB安装

## 一. 下载
        Linux相关的社区版下载， CentOS7 x64 下载Redhat x64版本
        # https://www.mongodb.com/download-center/v2/community
        

## 二. 安装
        安装参考：https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/
        #sudo tar -zxvf tar -zxvf mongodb-linux-*-4.0.3.tgz -C /opt/
        #cd /opt/ 
        #sudo mv mongodb-linux-* mongodb
        #sudo chown -R hadoop:hadoop ./mongodb

------------------------------------------------------------------------

## 三. 环境变量配置

        #sudo vi /etc/profile
        --------------------------------------------------------------
        export MONGODB_HOME=/opt/mongodb
        export PATH=$PATH:$MONGODB_HOME/bin
        --------------------------------------------------------------
        #source /etc/profile   //立即生效

------------------------------------------------------------------------

## 四. 单个服务启动和使用

        1. Damen服务启动   --fork --smallfiles
        # cd /opt/mongodb
        # mkdir 27017
        # cd 27017
        # mkdir data
        # mongod --dbpath 27017/data --logpath 27017/log --smallfiles --fork --port 27017
        
        2. 连接
        # mongo --port 27017
        # >help   //查看命令
        # >show dbs // 查看数据库
        # >db.user.insert({name:'woniu', sex:'male'})  //插入数据, 此时如果没有使用use db命令，则默认会创建一个test 数据库，并在此建表user
        # >db.user.find() //查询
        # >use test
        # >show collections  // 查看当前数据库的表

------------------------------------------------------------------------

## 五. Studio3T 工具使用
        1. 创建mongodb.conf并使用conf文件启动mongodb守护进程
        ##mongodb.conf
        dbpath=/opt/mongodb/27017/data/    #需要提前创建文件夹
        logpath=/opt/mongodb/27017/log/mongodb.log #需要提前创建文件夹
        logappend=true
        bind_ip=127.0.0.1
        port=27017
        maxConns=100
        fork=true
        smallfiles=true
        
        2.启动方法
        # mongod -f conf/mongodb.conf
        # 排错： 如果失败，建议杀掉前一个mongodb进程，删除相关文件夹，重新创建相关data和log目录
        
        3. Studio3T测试        
        #新版本指令：插入，删除
        db.inventory.insertMany([
        {item:'journal',qty:25,size:{h:14,w:21,uom:'cm'},status:'A'},
        {item:'notebook',qty:50,size:{h:8.5,w:11,uom:'in'},status:'P'},
        {item:'paper',qty:100,size:{h:8.5,w:11,uom:'in'},status:'D'},
        {item:'planner',qty:75,size:{h:22.85,w:30,uom:'cm'},status:'D'},
        {item:'postcard',qty:45,size:{h:10,w:15.25,uom:'cm'},status:'A'},
        ]);
        db.inventory.deleteMany({}) //删除所有
        db.inventory.deleteOne({status:'D'}) // 只删除一个，即便找到匹配的多个记录

        ##旧版本: 插入，删除
        db.stu.insert({_id:1,sn:'001',name:'zhang'})
        db.stu.insert([{_id:2,sn:'002',name:'zhang2'}, {_id:3,sn:'003',name:'zhang3'},{_id:4,sn:'004',name:'zhang4'}])
        db.stu.remove({sn:'001'})

        # 更新
        db.collection.update(query,update,options)
        - query--查询条件
        - update--新值
        - options--附加条件
        #db.stu.update({name:'zhang'},{$set:{name:'songjiang'}})
