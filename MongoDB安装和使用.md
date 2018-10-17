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
