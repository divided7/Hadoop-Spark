# Hadoop-Spark 分布式数据存储 for 训练数据
## 数据服务器配置
### 安装Java11
```bash
sudo apt update
sudo apt install -y openjdk-11-jdk
java -version
```
### 安装Hadoop
**下载Hadoop**
[官网](https://hadoop.apache.org/releases.html)

从[Hadoop 3.4.2 binary](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz)下载压缩包，或者下面指令下载
```bash
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz
tar -xzvf hadoop-3.4.2.tar.gz
sudo mv hadoop-3.4.2 /usr/local/hadoop
```
**配置环境变量**
```bash
echo 'export HADOOP_HOME=/usr/local/hadoop' >> ~/.bashrc
echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> ~/.bashrc
source ~/.bashrc
```
**配置Hadoop的java**
```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```
```bash
vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh
# 在文末添加`export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64`
```
**验证Hadoop是否安装成功**
```bash
hadoop version
>>> Hadoop 3.4.2
>>> Source code repository https://github.com/apache/hadoop.git -r 84e8b89ee2ebe6923691205b9e171badde7a495c
>>> Compiled by ahmarsu on 2025-08-20T10:30Z
>>> Compiled on platform linux-x86_64
>>> Compiled with protoc 3.23.4
>>> From source with checksum fa94c67d4b4be021b9e9515c9b0f7b6
>>> This command was run using /usr/local/hadoop/share/hadoop/common/hadoop-common-3.4.2.jar
```

## 数据库配置
### core-site
```bash
vim /usr/local/hadoop/etc/hadoop/core-site.xml
```
有如下配置: (建议先sudo lsof -i :9000看一下端口使用情况)
```xml
<configuration>
  <!-- 设置 HDFS 的主入口 (NameNode 地址) -->
  <property>
    <name>fs.defaultFS</name>
    <!-- 例如<value>hdfs:/192.168.1.100:9000</value>，这里注意每台机器都应该是相同的NameNode -->
    <value>hdfs://namenode-host:9000</value>
  </property>

  <!-- 临时文件目录(格局实际情况修改value） -->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/app/data/hadoop/tmp</value>
  </property>

  <!-- I/O 缓冲大小（可适度调大） -->
  <property>
    <name>io.file.buffer.size</name>
    <value>131072</value> <!-- 128 KB -->
  </property>

  <!-- 如果存在多网卡，可指定绑定的主机名/IP -->
  <property>
    <name>dfs.namenode.rpc-bind-host</name>
    <value>0.0.0.0</value>
  </property>
</configuration>
```
对于`core-site.xml`，可以在多台机器上使用相同的`core-site.xml`配置文件。

### hdfs-site
```bash
vim /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```
有如下配置:
```xml
<configuration>
  <!-- NameNode 元数据目录（如果本机也做 NameNode, 注意根据实际修改value路径） -->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/app/data/hadoop/hdfs/namenode</value>
  </property>

  <!-- DataNode 数据目录（必须创建，DataNode 才能存储块, 注意根据实际修改value路径） -->
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/app/data/hadoop/hdfs/datanode</value>
  </property>

  <!-- HDFS 副本数（单机设置 1，多机集群可设置 2-3，越大对读取速度和数据安全越有利，但是磁盘占用更大；最小值为1，最大值为DataNode数量） -->
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>

  <!-- HDFS 块大小（可以根据大文件或图片大小调整，默认 128MB） -->
  <property>
    <name>dfs.blocksize</name>
    <value>268435456</value> <!-- 256MB -->
  </property>

  <!-- 是否启用权限校验（单机可关闭，生产可开启） -->
  <property>
    <name>dfs.permissions</name>
    <value>false</value>
  </property>
</configuration>
```
对于`hdfs-site.xml`，可以在多台机器上使用相同的`hdfs-site.xml`配置文件。

## 多机集群
若需要多机配置，重复上述[数据服务器配置](#数据服务器配置)和[多机集群](#多机集群)的操作即可，注意在新机器上可能要替换新机器的存储路径。

## Hadoop启动
### 初始化数据库路径（初次使用或修改xml后需要初始化）
注意这里的所有路径应和`core-site.xml`中的**tmp**目录和`hdfs-site.xml`中的**namenode**，**datanode**目录相对应
```bash
sudo mkdir -p /app/data/hadoop/hdfs/namenode
sudo mkdir -p /app/data/hadoop/hdfs/datanode
sudo mkdir -p /app/data/hadoop/tmp

# 创建hadoop组和权限管理
sudo groupadd hadoop
sudo usermod -aG hadoop {user_name}
sudo chown -R {user_name}:hadoop /app/data/hadoop
sudo chmod -R 770 /app/data/hadoop
```

### 格式化数据库（初次使用需要格式化）
在Namenode机器上格式化数据库(初次使用前需要格式化)
```bash
hdfs namenode -format
```
### 启动服务（注意需要配置自己账户的ssh-key，且默认不会开机自启）
```bash
start-dfs.sh
```
检查进程是否启动
```bash
jps
>> 3290908 NameNode
>> 3286873 DataNode
>> 3287143 SecondaryNameNode
```
如果如上就正常。如果没有NameNode可能是端口冲突，可以`tail -n 50 $HADOOP_HOME/logs/hadoop-user1-namenode-*.log`查看日志

## Hadoop简单应用
* `http://localhost:9870`: 9870是默认NameNode的webui端口
* `http://localhost:9864`: 9864是默认DataNode的webui端口
```bash
hdfs dfs -ls / # 等价于本地的`ls /`
hdfs dfs -mkdir /test  # 等价于本地的`mkdir /test`
hdfs dfs -put /path/to/local/dir /path/in/hdfs/ # 将本地文件或文件夹上传到hdfs指定路径
hdfs dfs -get /file ./ # 将hdfs文件拉取回本地
```

