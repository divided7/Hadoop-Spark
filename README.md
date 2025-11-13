# Hadoop-Spark 分布式数据存储 for 训练数据
## 数据服务器配置
### 安装Java11
```bash
sudo apt update
sudo apt install -y openjdk-11-jdk
java -version
```
### 安装Hadoop
[官网](https://hadoop.apache.org/releases.html)

从[Hadoop 3.4.2 binary](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz)下载压缩包，或者下面指令下载
```bash
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.2/hadoop-3.4.2.tar.gz
tar -xzvf hadoop-3.4.2.tar.gz
sudo mv hadoop-3.4.2 /usr/local/hadoop
```
