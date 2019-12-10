# Hadoop环境搭建


{{% admonition "note" "特别注意" %}}
完全分布式搭建，搭建好master之后，直接把master的hadoop文件夹发送给slaves就可以了
{{% /admonition  %}}

# 伪分布式搭建

1. 准备Linux环境
   1. 安装虚拟机master、slave1、slave2
   2. [java安装](#java安装)
   3. ip修改
   4. hostname修改
   5. hosts：/etc/hosts
   6. iptables关闭
   7. chkconfig
   8. ssh免密登录
2. 下载Hadoop
   1. 下载上传
   2. 解压
   3. 添加环境变量
   4. 修改配置文件
      1. [core-site.xml](#core-site)
      2. [hdfs-site.xml](#hdfs-site)
      3. [yarn-site.xml](#yarn-site)
      4. [mapred-site.xml](#mapred-site)
      5. [hadoop-env.sh](#hadoop-env)
      6. [yarn-env.sh](#yarn-env)
      7. slaves文件

## 详细配置
>安装Linux软件
```shell
yum install -y 
                vim
                net-tools(ifconfig netstate route)
                bash-completion(自动补全)
```

| 软件名    | 根目录         |
| --------- | -------------- |
| java      | /opt/java      |
| hadoop    | /opt/hadoop    |
| zookeeper | /opt/zookeeper |
| kafka     | /opt/kafka     |

### hostname
* master
* slave1
* slave2

### ip
* 192.168.100.100
* 192.168.100.101
* 192.168.100.102

### 改ip地址
```shell
nmcli connection ens33 ipv4.address 192.168.100.100
nmcli connection ens33 ipv4.gateway 192.168.100.2
nmcli connection ens33 ipv4.dns 192.168.100.2
nmcli connection ens33 ipv4.method manual

# 或
vim /etc/sysconfig/network-scripts/ifcfg-ens33

systemstl restart network
```

### 修改hosts
```shell
vim /etc/hosts
```
填入
```
192.168.100.100 master
192.168.100.101 slave1
192.168.100.102 slave2
```

### ssh无密码登录
```shell
ssh-keygen -t rsa

cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

chmod 600  ~/.ssh/authorized_keys
```
全部采用默认值，第一次会输入yes，后面就是无密码登录了

### 关闭防火墙和selinux
```shell
systemctl start firewalld.service#启动firewall

systemctl stop firewalld.service#停止firewall

systemctl disable firewalld.service
```

### 添加环境变量
```
echo 'export HADOOP_HOME=/opt/hadoop/bin:/opt/hadoop/sbin' >> /etc/profile

echo 'export PATH=$PATH:$HADOOP_HOME' >> /etc/profile
```

### <b id="hadoop-site">hadoop-env.sh</b>
export JAVA_HOME=java安装目录(/opt/jdk_1.8)   
export HADOOP_LOG_DIR=日志目录(创建一个/opt/hadoop_repo/logs/hadoop)

### <b id="yarn-site">yarn-env.sh</b>
export JAVA_HOME=java目录   
export YARN_LOG_DIR=日志目录(创建一个/opt/hadoop_repo/logs/yarn)

### <b id="core-site">core-site.xml</b>
```xml
<configuration>
  <property>
   <name>hadoop.tmp.dir</name>
   <value>/opt/hadoop_repo</value>
   <description>A base for other temporary directories.</description>
 </property>
 <property>
  <name>io.file.buffer.size</name>
   <value>131072</value>
 </property>
 <property>
   <name>fs.defaultFS</name>
   <value>hdfs://master:9000</value>
   <description>hdfs://hostname:port_number</description>
 </property>
</configuration>
```

### <b id="hdfs-site">hdfs-site.xml</b>
namenode和datanode的配置可以不用在这儿写，第一次启动hadoop时格式化操作会在repo中自动生成他们的工作文件夹。只配置一个replication就行。replication的数量指的是副本数量，**因为是伪分布式，没有三个副本，所以修改为1。**
```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <!-- <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/home/hadoop/hadoop-2.7.3/hdfs/name</value>
    <final>true</final>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/home/hadoop/hadoop-2.7.3/hdfs/data</value>
    <final>true</final>
  </property>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>master:9001</value>
  </property>
  <property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
  </property>
  <property>
    <name>dfs.permissions</name>
    <value>false</value>
  </property> -->
</configuration>
```

### <b id="mapred-site">mapred-site.xml</b>
这里需要把```mapred-site.xml.template```改成```mapred-site.xml```

```shell
cp mapred-site.xml.template mapred-site.xml
```

```xml
<property>
   <name>mapreduce.framework.name</name>
   <value>yarn</value>
</property>
```


### <b id="yarn-site">yarn-site.xml</b>
```xml
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
```

### slaves文件
伪分布时保留为localhost就行

---
## Java安装
1. tar -zxvf  jdk1.8.0_161.tar.gz
2. 配置环境变量
   - java环境变量可以在/etc/profile.d中添加jdk.sh
   - 或直接在/etc/profile里面写
   - 直接source /etc/profile就行

如果是用jdk.sh这种配置法：
```
#!bin/bash
JAVA_HOME=/opt/java/
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
```

## 运行
>第一次运行需格式化
```
hadoop namenode -format
```

# 分布式集群搭建
比如这里规划：

* master
* slave1
* slave2

>强调：先配置maser，然后直接把hadoop文件夹发送给slaves就行了。

>注意：针对免密登录，不仅要各自能够免密登录自己，还需配置**主节点**可以免密登录从节点。

```
scp ~/.ssh/id_rsa.pub hadoop@slave1:~/

ssh slave1

cat ~/id_rsa.pub >> ~/.ssh/authorized_keys

chmod 600  ~/.ssh/authorized_keys

exit
```

>注意：需要保证集群的各个节点时间同步。
```
ntpdate -u ntp.sjtu.edu.cn
```


## 与伪分布配置的不同

### hdfs-site.xml
增加副本为2。
增加参数指定在哪个机器上启动SecondaryNameNode。
```xml
<configuration>
　　<property>
　　　　<name>dfs.replication</name>
　　　　<value>2</value>
　　</property>
　　<property>
　　　　<name>dfs.namenode.secondary.http-address</name>
　　　　<value>master:50090</value>
　　</property>
</configuration>
```

### yarn-site.xml
指定yarn的主节点。
```xml
<configuration>
　　<property>
　　　　<name>yarn.resourcemanager.hostname</name>
　　　　<value>master</value>
　　</property>
　　<property>
　　　　<name>yarn.nodemanager.aux-services</name>
　　　　<value>mapreduce_shuffle</value>
　　</property>
</configuration>
```

### slaves
```
slave1
slave2
```