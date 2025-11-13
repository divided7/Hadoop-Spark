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
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
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
```vim /usr/local/hadoop/etc/hadoop/core-site.xml
```
有如下配置:
```xml
<configuration>
  <!-- 设置 HDFS 的主入口 (NameNode 地址) -->
  <property>
    <name>fs.defaultFS</name>
    <!-- 下面的value如果要只支持单机可以写为：<value>hdfs://data-server:9000</value>，但是考虑到扩展性，写成下面的多机版本更好 也兼容单机 -->
    <value>hdfs://namenode-host:9000</value>
  </property>

  <!-- 临时文件目录（建议放到 SSD 或系统盘） -->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/data/hadoop/tmp</value>
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
```vim /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```
有如下配置:
```xml
<configuration>
  <!-- NameNode 元数据目录（如果本机也做 NameNode） -->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///data/hadoop/hdfs/namenode</value>
  </property>

  <!-- DataNode 数据目录（必须创建，DataNode 才能存储块） -->
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///data/hadoop/hdfs/datanode</value>
  </property>

  <!-- HDFS 副本数（单机设置 1，多机集群可设置 2-3） -->
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
