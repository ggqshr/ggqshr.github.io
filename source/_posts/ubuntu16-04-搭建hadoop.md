---
title: ubuntu16.04 搭建hadoop
typora-copy-images-to: ubuntu16.04 搭建hadoop
date: 2021-05-11 11:39:21
tags:
    - swy
    - hadoop
categories: hadoop相关
---
# 1. 设置主机名
【4台机器都要修改】
```
hostnamectl set-hostname cd001
```

# 2. 配置DNS
【配置本地hosts映射】
```
vim /etc/hosts

主机IP 主机名
172.20.46.89 cdh001
172.20.46.90 cdh002
172.20.46.93 chd003
172.20.46.94 cdh004
```

# 3. 关闭防火墙
【4台机器都要做】
```
# 关闭服务
systemctl stop firewalld
# 关闭开机自启动
systemctl disable firewalld
```

# 4. 配置免密登陆
【4台机器都要做】
```
# 生成ssh密钥【会在~/ssh文件夹下生成id_rsa和id_rsa.pub两个文件】
ssh-keygen -t rsa
# 将私钥拷贝到其他机器
ssh-copy-id root@cdh002
ssh-copy-id root@cdh003
ssh-copy-id root@cdh004
```

# 5. 下载Java和hadoop包 
【放在/root/opt/文件夹下】
```
# cd到/opt下，下载hadoop
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.10.1/hadoop-2.10.1.tar.gz

# 下载java的jdk
wget https://mirrors.tuna.tsinghua.edu.cn/AdoptOpenJDK/9/jdk/x64/linux/OpenJDK9-OPENJ9_x64_Linux_jdk-9.0.4.12_openj9-0.9.0.tar.gz

# 解压
tar -zxvf jdk....tar.gz
tar -zxvf hadoop....tar.gz

# 配置环境变量
vim /etc/profile
# 追加下面内容
export JAVA_HOME=/opt/jdk... # jdk安装路径
export CLASSPATH=$JAVA_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin

export HADOOP_HOME=/opt/hadoop.. #hadoop安装路径
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop

# 使得配置文件生效
source /etc/profile

# 将Jdk和环境配置文件拷贝到其他机器
scp -r /opt/jdk... root@cdh002/003/004:/opt/
scp -r /etc/profile root@cdh002/003/004:/etc/profile
【-r:文件夹】
```

# 6. 配置hadoop环境脚本文件中的JAVA_HOME参数
```
# 进入hadoop安装目录下的/etc/hadoop文件夹下
# 分别在hadoop_env.sh\mapred_env.sh\yarn_env.sh文件中添加或修改参数
export JAVA_HOME="/opt/jdk..." # jdk安装文件夹
```
- **此处修改的JAVA_HOME的地址不能用${JAVA_HOME},会报错，一定要修改成绝对路径**

# 7. 修改hadoop配置文件
1. core-site.xml
```
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://cdh001:9000</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/home/hadoop/data/hadoopdata</value>
        </property>
</configuration>
```
2. hdfs-site.xml
```
<configuration>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/home/hadoop/data/hadoopdata/name</value>
                <description>为了保证元数据的安全一般配置多个不同目录</description>
        </property>

        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/home/hadoop/data/hadoopdata/data</value>
                <description>datanode 的数据存储目录</description>
        </property>

        <property>
                <name>dfs.replication</name>
                <value>2</value>
                <description>HDFS 的数据块的副本存储个数, 默认是3</description>
        </property>

        <property>
                <name>dfs.secondary.http.address</name>
                <value>cdh001:50090</value>
                <description>secondarynamenode 运行节点的信息，和 namenode 不同节点</description>
        </property>
</configuration>
```
3. mapred-site.xml
```
cp mapred-site.xml.template mapred-site.xml
 vi mapred-site.xml
 # 修改内容如下
 <configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
</configuration>
```
4. yarn-site.xml
```
<configuration>

<!-- Site specific YARN configuration properties -->

        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>hadoop4</value>
        </property>
        
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
                <description>YARN 集群为 MapReduce 程序提供的 shuffle 服务</description>
        </property>

</configuration>
```
5. slaves
```
cdh001
cdh002
cdh003
cdh004
```

# 8. 将配置好的hadoop文件夹拷贝至其他机器
```
scp -r /opt/hadoop... root@cdh002/003/004:/opt/
```

# 9. 格式化和启动
【只有cdh001需要格式化和启动】
```
# 格式化（去hadoop安装目录下）
bin/hdfs namenode -format

# 启动
sbin/start-dfs.sh
sbin/start-yarn.sh
```

# 10. 验证
每个机器输入jps命令查看