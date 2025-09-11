# DORIS2.0 集群安装

##### 1. 前置系统准备

```shell
#1.Java环境
tar xf jdk-8u341-linux-x64.tar -C /opt/soft
vim /etc/profile
export JAVA_HOME=/opt/soft/jdk180
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JRE_HOME=$JAVA_HOME/jre

#2.设置进程可以拥有的VMA的数量
#vim /etc/sysctl.d/99-sysctl.conf 
#vm.max_map_count=262144
vim /etc/sysctl.conf
vm.max_map_count=2621440

#3.设置系统最大打开文件句柄数
vi /etc/security/limits.conf 
* soft nofile 65536
* hard nofile 65536

#4.时间同步
yum install -y chrony
chronyd -q 'server ntp1.aliyun.com iburst' 

#5.禁用swap
swapoff -a

#6.重启系统
shutdown -r now
```

##### 2. Doris安装

```shell
#1.解压
tar xf apache-doris-2.0.0-bin-x64.tar.xz
#2.修改/opt/soft/doris2/fe/conf/fe.conf添加
mysql_service_nio_enabled = true
priority_networks = 192.168.6.0/24
#3.修改/opt/soft/doris2/be/conf/be.conf添加
storage_root_path = /opt/soft/doris2/storage
#4.创建存储目录
mkdir -p /opt/soft/doris2/storage
#5.集群将此机器复制其他两台(这里集群机器共3台)
```

##### 3.启动fe

```shell
#在1号机器上启动
/opt/soft/doris2/fe/bin/start_fe.sh --daemon
#安装mariadb方便操作
yum install -y mariadb
#使用mysql client进入fe
mysql -uroot -P 9030 -h 127.0.0.1
SHOW PROC '/frontends' \G;
```

![img](https://img2023.cnblogs.com/blog/911490/202309/911490-20230901162157831-1968837502.png)

```shell
#192.168.6.140 150两台执行  #--helper 指定当前leader的地址
/opt/soft/doris2/fe/bin/start_fe.sh  --helper 192.168.6.130:9010 --daemon

#在第一台数据库节点上将140和150两台机器fe作为follower添加到机器中
MySQL [(none)]> ALTER SYSTEM ADD FOLLOWER "192.168.6.140:9010";
MySQL [(none)]> ALTER SYSTEM ADD FOLLOWER "192.168.6.150:9010";
SHOW PROC '/frontends' \G;  #确保相关节点 join和active值为true 注意另外两台的CurrentConnected:No
```

##### 4.启动BE和Broker

```shell
#三台机器都执行以下操作
/opt/soft/doris2/be/bin/start_be.sh --daemon
/opt/soft/doris2/extensions/apache_hdfs_broker/bin/start_broker.sh --daemon

#在数据库节点上将三个be节点和三个broker节点添加到数据库
#添加BE节点
ALTER SYSTEM ADD BACKEND "192.168.6.130:9050";
ALTER SYSTEM ADD BACKEND "192.168.6.140:9050";
ALTER SYSTEM ADD BACKEND "192.168.6.150:9050";
SHOW BACKENDS;

#添加Broker节点
ALTER SYSTEM ADD BROKER my_broker "192.168.6.130:8000";
ALTER SYSTEM ADD BROKER my_broker "192.168.6.140:8000";
ALTER SYSTEM ADD BROKER my_broker "192.168.6.150:8000";
SHOW BROKER;
```

##### 5.通过控制台webui查看页面

```shell
登录控制台，访问http://192.168.6.130:8030/,默认用户root 密码为空
```

