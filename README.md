# Setting up Hadoop, Hive, and Spark in Pseudo-Distributed Mode

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Install and Configure Hadoop](#2-install-and-configure-hadoop)
3. [Install and Configure Hive](#3-install-and-configure-hive)
4. [Install and Configure Apache Spark](#4-install-and-configure-apache-spark)
5. [Configure Hive Metastore with MySQL](#5-configure-hive-metastore-with-mysql)
6. [Integrate Spark with Hive Metastore](#6-integrate-spark-with-hive-metastore)
7. [Optional Enhancements](#7-optional-enhancements)

---

## 1. Prerequisites

### 1.1 Install Java

```bash
sudo apt update
sudo apt install openjdk-8-jdk
java -version
```

### 1.2 Configure SSH

```bash
sudo apt install openssh-server
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
ssh localhost
```

---

## 2. Install and Configure Hadoop

### 2.1 Download Hadoop

```bash
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar -xvzf hadoop-3.3.6.tar.gz
sudo mv hadoop-3.3.6 /usr/local/hadoop
```

### 2.2 Configure Environment Variables

Edit `~/.bashrc`:

```bash
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
source ~/.bashrc
```

### 2.3 Update Configuration Files

Edit `core-site.xml`:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```

Edit `hdfs-site.xml`:

```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

Edit `mapred-site.xml`:

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

Edit `yarn-site.xml`:

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

### 2.4 Format NameNode

```bash
hdfs namenode -format
```

### 2.5 Start Hadoop Services

```bash
start-dfs.sh
start-yarn.sh
```

Verify:

```bash
jps
```

---

## 3. Install and Configure Hive

### 3.1 Download Hive

```bash
wget https://dlcdn.apache.org/hive/hive-4.0.1/apache-hive-4.0.1-bin.tar.gz
tar -xvzf apache-hive-4.0.1-bin.tar.gz
sudo mv apache-hive-4.0.1-bin /usr/local/hive
```

### 3.2 Configure Environment Variables

Edit `~/.bashrc`:

```bash
export HIVE_HOME=/usr/local/hive
export PATH=$PATH:$HIVE_HOME/bin
export HADOOP_HOME=/usr/local/hadoop
source ~/.bashrc
```

### 3.3 Initialize Hive Metastore

```bash
schematool -initSchema -dbType derby
```

### 3.4 Start Hive CLI

```bash
hive
```

---

## 4. Install and Configure Apache Spark

### 4.1 Download Spark

```bash
wget https://dlcdn.apache.org/spark/spark-3.5.0/spark-3.5.0-bin-hadoop3.tgz
tar -xvzf spark-3.5.0-bin-hadoop3.tgz
sudo mv spark-3.5.0-bin-hadoop3 /usr/local/spark
```

### 4.2 Configure Environment Variables

Edit `~/.bashrc`:

```bash
export SPARK_HOME=/usr/local/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
source ~/.bashrc
```

### 4.3 Verify Spark

```bash
spark-shell
```

---

## 5. Configure Hive Metastore with MySQL

### 5.1 Install MySQL

```bash
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation
```

### 5.2 Create Database and User

```sql
CREATE DATABASE metastore;
CREATE USER 'hiveuser'@'localhost' IDENTIFIED BY 'hivepassword';
GRANT ALL PRIVILEGES ON metastore.* TO 'hiveuser'@'localhost';
FLUSH PRIVILEGES;
```

### 5.3 Configure Hive Metastore

Add MySQL JDBC driver:

```bash
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-8.0.33.tar.gz
tar -xvzf mysql-connector-java-8.0.33.tar.gz
sudo cp mysql-connector-java-8.0.33/mysql-connector-java-8.0.33.jar /usr/local/hive/lib/
```

Edit `hive-site.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://localhost:3306/metastore</value>
    </property>

    <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>hiveuser</value>
    <description>Username for the database connection.</description>
    </property>

    <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>hivepassword</value>
    <description>Password for the database connection.</description>
    </property>

</configuration>
```

### 5.4 Initialize Schema

```bash
schematool -initSchema -dbType mysql
```

---

## 6. Integrate Spark with Hive Metastore

Edit Spark configuration:

```bash
sudo nano $SPARK_HOME/conf/spark-defaults.conf
```

Add:

```
spark.sql.catalogImplementation=hive
spark.sql.warehouse.dir=hdfs://localhost:9000/user/hive/warehouse
```

Copy MySQL Driver:

```bash
sudo cp /usr/local/hive/lib/mysql-connector-java-8.0.33.jar /usr/local/spark/jars/
```

---

## 7. Optional Enhancements

### 7.1 Enable Remote Metastore

Edit `hive-site.xml`:

```xml
<property>
    <name>hive.metastore.uris</name>
    <value>thrift://localhost:9083</value>
</property>
```

Restart Hive Metastore Service:

```bash
hive --service metastore &
```

### 7.2 Install Hue for GUI Access

```bash
sudo apt-get install hue
sudo service hue start
```

### 7.3 Configure Kerberos Authentication for Security

Refer to the [Hadoop Security Guide](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SecureMode.html) for detailed steps.

---

### Now your Hadoop, Hive, and Spark setup is production-ready!

