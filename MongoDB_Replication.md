# Replication

## 概要
  - 特点：多台服务器维护相同的副本，类似redis的主从架构。主服务器挂了后，会选择一台从服务器变为主服务器。
  - 参考： https://docs.mongodb.com/manual/replication/
  
# Replication模拟配置
  - 同一台机器上，开启 27017  27018  27019 三个端口，使用不同的配置文件启动，模拟三个MongoDB实例
  - YAML配置文件
	'''
		systemLog:
		   destination: file
		   path: "/opt/mongodb/log/27017.log"
		   logAppend: true
		storage:
		   dbPath: "/opt/mongodb/data/27017" 
		   mmapv1:
			  smallFiles: true
		   journal:
			  enabled: true
		processManagement:
		   fork: true
		net:
		   bindIp: 127.0.0.1
		   port: 27017
		replication:
		   oplogSizeMB: 128
		   replSetName: "rsa"

	'''
  - 注意: log, data等文件夹必须提前创建好
  - 执行命令,开启三个MongoDB实例进程
	./bin/mongod --config ./config/27017.conf
	./bin/mongod --config ./config/27018.conf
	./bin/mongod --config ./config/27019.conf
  - 连接一个实例 ：  ./bin/mongo --port=27017
  - 创建配置变量
	> var rsconf = {
		_id:'rsa',
		members:
		[
			{_id:0, host:'127.0.0.1:27017' },
			{_id:1, host:'127.0.0.1:27018' },
			{_id:2, host:'127.0.0.1:27019' }
		]
	}
  - 初始化集群
    > rs.initiate(rsconf);
  
  - 查看状态
    > rs.status(); //此时其中一个就会成为Primary主节点，一般是27017这个
	
  - 查看当前配置
    > rs.config();
	
  - 删除节点
    > rs.remove('127.0.0.1:27019');
	
	
  - tips：
	rsa:SECONDARY> show colletions;
	JavaScript execution failed: error: { "$err" : "not master and slaveOk=false", "code" : 13435 } 
	出现上述错误,是因为slave默认不许读写，如果非要解决，方法如下：
	>rs.slaveOk();




  
	
