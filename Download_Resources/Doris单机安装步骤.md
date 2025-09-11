# Doris单机安装步骤

##### 1. 前置环境配置

```shell
# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

#永久关闭selinux

sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
setenforce 0  # 临时

# 关闭swap
swapoff -a  # 临时
sed -ri 's/.*swap.*/#&/' /etc/fstab    # 永久



cat >> /etc/security/limits.conf  <<EOF
* soft nofile 65536
* hard nofile 65536
* soft nproc 65536
* hard nproc 65536
EOF



#设置时间同步
yum install -y chrony
#修改阿里时间服务器地址
vi /etc/chrony.conf
server ntp1.aliyun.com
server ntp2.aliyun.com
server ntp3.aliyun.com
注释掉server 0.centos.pool.ntp.org iburst
#启动chrony
systemctl start chronyd

#安装jdk及依赖包 这里安装了jdk1.8 如果手动安装也可以
yum install -y build-essential gcc-10 g++-10 java-1.8.0-openjdk.x86_64 maven cmake byacc flex automake libtool-bin bison binutils-dev libiberty-dev zip unzip libncurses5-dev curl git ninja-build python

#配置java环境变量
cat >>/etc/profile <<EOF
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
EOF

source /etc/profile

#修改/etc/sysctl.conf
vm.max_map_count=2621440

#激活
sysctl -p
#最好重新启动下虚拟机
```

##### 2.解压配置环境

```shell
cd /opt
#注意这里使用的1.2.7.1版本 
xz -d apache-doris-1.2.7.1-bin-x64-noavx2.tar.xz
tar -xvf apache-doris-1.2.7.1-bin-x64-noavx2.tar
mkdir soft
mv apache-doris-1.2.7.1-bin-x64-noavx2 soft/doris127
mkdir -p /opt/soft/doris127/data/{storage,doris-meta}
#进入fe文件夹的conf修改fe.conf
meta_dir=/opt/soft/doris127/data/doris-meta
priority_networks=192.168.179.144/24
sys_log_dir开的话可以看日志

#进入be文件夹的conf修改be.conf
priority_networks=192.168.6.128/24
storage_root_path=/opt/soft/doris127/data/storage

#/etc/profile环境变量
#doris127
export DORIS_FE_HOME=/opt/soft/doris127/fe
export DORIS_BE_HOME=/opt/soft/doris127/be
export PATH=$PATH:$DORIS_FE_HOME/bin:$DORIS_BE_HOME/bin
```

##### 3.启动设置密码并将be引入到fe中

```shell
#启动fe
./opt/soft/doris127/fe/bin/start_fe.sh --daemon

```

```mysql
#使用datagrid 用192.168.6.128:9030 登录doris的be端 然后再查询窗口
#设置密码
set password = password('ok');
#添加be
alter system add backend "192.168.6.128:9050";
```

```shell
#启动be端
./opt/soft/doris127/be/bin/start_be.sh --daemon
```



##### 4.最后在webui界面操作

```shell
#http://192.168.6.128:8030 用户名默认root 密码ok
```

