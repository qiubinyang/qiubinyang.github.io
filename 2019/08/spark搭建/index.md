# Spark搭建


1. 下载http://mirrors.tuna.tsinghua.edu.cn/apache/spark/spark-2.3.4/spark-2.3.4-bin-hadoop2.7.tgz
2. 解压
3. bin目录添加环境变量
4. 修改 spark-env.sh 和 slaves

```shell
cp spark-env.sh.template spark-env.sh
vim spark-env.sh
```

```yml
#配置如下
#hadoop的配置文件路径，spark会去读取配置文件，如果不在同一个集群，需拷贝配置文件到spark的节点，让spark能够读到
HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
#本机ip
SPARK_LOCAL_IP=master
JAVA_HOME=/opt/java
```

5. 复制给子节点
```
scp -r /opt/spark root@slave1:/opt
scp -r /opt/spark root@slave2:/opt
```

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://master:7077 \
--executor-memory 1G \
--total-executor-cores 2 \
examples/jars/spark-examples_2.11-2.3.4.jar \
20
```