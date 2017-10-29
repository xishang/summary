# Centos软件安装

***

## 目录

[TOC]

***

## 1. JDK

### 1）下载jdk并解压（预先下载并上传）

    tar -zxvf jdk-8u121-linux-x64.tar.gz
    mv jdk-8u121-linux-x64 jdk1.8.0_121

### 2）设置环境变量

    vim ~/.bash_profile
    # 添加内容
```
export JAVA_HOME=/usr/local/jdk/jdk1.8.0_121
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
PATH=$PATH:$HOME/bin:$JAVA_HOME/bin
export PATH
```

### 3）修改生效

    source ~/.bash_profile
	
## 2. Nginx

### 1）解决依赖

    GCC:        yum install gcc
    PCRE:       yum install -y pcre pcre-devel
    Zlib:       yum install -y zlib zlib-devel
    OpenSSL:    yum install -y openssl openssl-devel

### 2）下载nginx压缩包并解压

    wget http://nginx.org/download/nginx-1.12.1.tar.gz
    tar -zxvf nginx-1.12.1.tar.gz

### 3）编译安装

    cd nginx-1.12.1/
    ./configure
    make
    make install
    
### 4）启动

    /usr/local/nginx/sbin/nginx
    
## 3. MongoDB

### 1）解决依赖

    GLIBC:  注意mongodb最低版本要求
    
### 2）下载mongodb并解压

    wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.2.8.tgz
    tar -zvxf mongodb-linux-x86_64-rhel70-3.2.8.tgz
    mv mongodb-linux-x86_64-rhel70-3.2.8/ mongodb-3.2.8

### 3）建立db和logs文件夹

    cd mongodb-3.2.8/
    mkdir db
    mkdir logs

### 4）bin目录下新建并编辑mongodb.conf文件

    cd bin/
    vim mongodb.conf
    # 添加内容
```
# mongodb config
dbpath=/usr/local/mongodb/mongodb-3.2.8/db
logpath=/usr/local/mongodb/mongodb-3.2.8/logs/mongodb.log

# auth=true
port=27017
fork=true
logappend=true
```

### 5）以配置文件启动

    ./mongod -f mongodb.conf

### 6）配置用户名密码

    ./mongo
    use admin
    # 角色权限：root[超级管理员], readWrite[读写权限], read[读权限]
    db.createUser({user:”dev",pwd:"Dev235",roles:["readWrite"]})
    
### 7）在其他机器上连接mongodb

    # 远程连接要先连接到admin数据库
    mongo 182.61.18.144:27017/admin -udev -pDev235
    
## 4. Zookeeper

### 1）下载zookeeper并解压

    wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz
    tar -zxvf zookeeper-3.4.10.tar.gz

### 2）创建数据及日志文件夹

    cd zookeeper-3.4.10/
    mkdir data
    mkdir logs

### 3）编辑配置文件

    cd conf/
    cp zoo_sample.cfg zoo.cfg
    vim zoo.cfg
    # 添加内容
```
# 数据文件夹
dataDir=/usr/local/zookeeper/zookeeper-3.4.10/data
# 日志文件夹
dataLogDir=/usr/local/zookeeper/zookeeper-3.4.10/logs
```

### 4）启动zookeeper

    cd ../bin/
    ./zkServer.sh start
    
## 5. Redis

### 1）安装redis

    yum install redis

### 2）修改配置文件

    # 修改bind、requirepassword等参数
    vim /etc/redis.conf

### 3）指定配置文件启动

    redis-server /etc/redis.conf &

### 4）远程连接

    redis-cli -h 182.62.18.144 -p 6379 -a 123456
    
## 6. MySQL

### 1）添加repo

    vim /etc/yum.repos.d/mysql-community.repo
    # 添加内容
```
[mysql56-community]
name=MySQL 5.6 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.6-community/el/6/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

### 2）查看repo是否可用

    yum repolist enabled|grep mysql

### 3）安装MySQL

    sudo yum install mysql-community-server

### 4）启动并设置

    sudo service mysql start
    # 此时没有密码
    mysql -uroot -p
    # 设置本地连接密码
    set password for root@localhost = password(‘123456’)

### 5）设置远程连接

    # 设置远程连接的ip和密码
    grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option
    # 修改生效
    flush privileges

### 6）修改配置

    vim /etc/my.cnf
    # 添加内容
```
[mysqld]
character-set-server = utf8

# 默认配置占用内存较大，其中'table%'影响最大
max_connections = 100
table_definition_cache = 80
table_open_cache = 100
thread_concurrency=2
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_thread_concurrency = 2
```

## 7. RabbitMQ

### 1）下载rpm文件

    rabbitmq-server-3.6.10-1.el7.noarch.rpm

### 2）安装

    yum install rabbitmq-server-3.6.10-1.el7.noarch.rpm

### 3）启动

    rabbitmq-server start &

### 4）开启管理界面插件

    rabbitmq-plugins enable rabbitmq_management

### 5）创建用户并授权

    # 查看所有用户
    rabbitmqctl list_users
    # 添加用户admin
    rabbitmqctl add_user admin admin
    # 设置角色
    rabbitmqctl set_user_tags admin administrator
    # 设置权限
    rabbitmqctl set_permissions -p "/" admin ".*" ".*"
