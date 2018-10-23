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

## 五. 游标Cursor
        # 1. find() 返回的是一个Cursor游标
        # 2. 如果不用var 修饰变量，则Cursor自动显示前20个
        # 3. 如果使用了var，则不会自动显示
        
## 六. 索引Index
        # 1. MongoDB的索引采用的是btree(B树或B-树，-不发音)， Mysql采用的是B+树
        # 2. 创建索引 db.student.ensureIndex({sn:1})   // +1升序  -1降序
        # 3. 查看索引 db.student.getIndexes()   // 查看索引
        # 4. 删除索引 db.student.dropIndexes()  // 删除索引
        # 5. 创建多索引 db.student.ensureIndex({sn:1, name:1})
        # 6. 子文档索引 db.shop.ensureIndex({'spec.area':1})  // 子文档指的是子属性，使用.操作符访问
        # 7. 唯一索引： db.student.ensureIndex({sn:1}, {unique:true}) // 该字段的数值不能重复
        # 8. 稀疏索引： db.student.ensureIndex({name:1}, {sparse:true}) 如果不含该字段，则不建立索引；普通索引是将其值作为NULL建立索引的
        # 9. Hash索引： db.student.ensureIndex({name:'hashed'}) 
             - 缺点： 范围查询时，优化有限
             - 优点： 比普通索引快
             
        # 使用B-树而不是B+树作为索引的原因：
           B+树只有叶子节点才存储数据，每次的查询时间复杂度固定： LogN,查询到树的叶子节点；
           B-树所有节点都包含有数据，查询复杂度<= LogN
           MongoDB是文档型数据库，KV聚合，所以采用B-树反而更优；
        
## 七. 用户及认证
        创建用户，且支持以下四种认证方式: 
        参考： https://docs.mongodb.com/manual/tutorial/create-users/
        # 1. UserName/Password
        # 2. Kerberos
        # 3. LDAP
        # 4. x.509 Client Certificate
