## Hadoop 的环境搭建

#### 1 下载解压

下载路径<https://archive.apache.org/dist/hadoop/common/>  通过FTP上传到服务器/usr/local/source下

以下以2.7.3为例部署在linux的单节点服务器上，保存位置为 /usr/local/source/

```sh
# 解压命令到指定文件夹
# 解压命令 tar 指定文件夹 -C 
tar -zxvf hadoop-2.7.3.tar.gz -C /usr/local/hadoop
```



#### 2 安装与环境配置

##### 2.1 hadoop 设定到环境变量中

```sh
# 进入到cd /usr/local/hadoop/hadoop-2.7.3/etc/hadoop/ , vim修改hadoop-env.sh
export JAVA_HOME=/usr/local/jdk1.8
```



```sh
# 在/etc/profile文件中配置hadoop环境变量：
export PATH=$PATH:/usr/local/hadoop/hadoop-2.7.3/bin:/usr/local/hadoop/hadoop-2.7.3/sbin

# 更好一点的写法
export HADOOP_HOME="/usr/local/hadoop/hadoop-2.7.3"
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

```



```sh
然后执行生效
$:  source /etc/profile
生效后可以执行部分 hdfs 命令
```

##### 2.2  配置../etc/hadoop中的文件

```SH
# /usr/local/hadoop/hadoop-2.7.3/etc/hadoop 中有许多配置文件
# hadoop-env.sh、mapred-env.sh、yarn-env.sh
# 修改其中的java环境变量
 export JAVA_HOME=/usr/local/jdk1.8
```

core-site.xml

```xml
这里有很多可以配置的属性，目前不会，就设定基本的属性
<configuration>
    <property>
	<name>hadoop.tmp.dir</name>
        <!--文件处理的临时存放路径 -->
	<value>/usr/local/hadoop/tmp</value>
   </property>
    <property>
         <!--服务器的端口映射，内网在系统的/etc/host中查看
		::1	localhost	ip6-localhost	ip6-loopback
		ff02::1	ip6-allnodes
		ff02::2	ip6-allrouters
		172.26.51.242	iZg49eyde8w3v0Z	iZg49eyde8w3v0Z-->
       <name>fs.defaultFS</name>
       <value>hdfs://iZg49eyde8w3v0Z:9001</value>
   <!--之后的yarn外网访问ip还是47.92.28.186-->
    </property>
</configuration>

 hadoop.tmp.dir配置的是Hadoop临时目录，比如HDFS的NameNode数据默认都存放这个目录下，查看*-default.xml等默认配置文件，就可以看到很多依赖${hadoop.tmp.dir}的配置。
```



##### 2.3 完成后格式化 namenode

``` sh
$ hdfs namenode -format
```

``` sh
执行完成后可以/usr/local/hadoop/tmp路径下查看生成的dfs目录

/usr/local/hadoop/tmp/dfs/name/current/VERSION
namespaceID=489763247
clusterID=CID-38bf17fe-78a5-40a8-9755-0d0d7af2ac8a
cTime=0
storageType=NAME_NODE
blockpoolID=BP-898761891-172.26.51.242-1565580465210
layoutVersion=-63

namespaceID：NameNode的唯一ID。
clusterID:集群ID，NameNode和DataNode的集群ID应该一致，表明是一个集群。
```

``` sh
# 启动namenode
$  ${HADOOP_HOME}/sbin/hadoop-daemon.sh start namenode

#启动DataNode
$ ${HADOOP_HOME}/sbin/hadoop-daemon.sh start datanode

# 启动SecondaryNameNode
$ ${HADOOP_HOME}/sbin/hadoop-daemon.sh start secondarynamenod
```

``` sh
# jps查看
jps -l
这里启动正常的化就就可以完全执行hdfs 命令 
如上传下载
```

#### yarn的配置启动

 ``` xml
默认没有mapred-site.xml文件，但是有个mapred-site.xml.template配置模板文件。复制模板生成mapred-site.xml
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>

 ```

``` xml
经验：yarn-site如果是集中启动，其实只需要在管理机上配置一份即可，但是如果单独启动，需要每台机器一份，在网页上可以看到当前机器的配置，以及这个配置的来源
<!--配置yarn-site.xml-->
<property>
    <name>yarn.resourcemanager.hostname</name>
    # 这里是主节点的ip的别名 本机windows是127.0.0.1的别名是localhost
    <value>iZg49eyde8w3v0Z</value>
</property>

<property> 
    <name>yarn.nodemanager.aux-services</name> 
    <value>mapreduce_shuffle</value> 
</property> 

<property>
  <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>


<!--配置其他节点:
配置到其他节点只需要把上面两个文件copy到其他主机即可，命令如下：-->
scp mapred-site.xml slave1:/usr/local/hadoop/etc/hadoop/mapred-site.xml
scp mapred-site.xml slave2:/usr/local/hadoop/etc/hadoop/mapred-site.xml
scp mapred-site.xml slave3:/usr/local/hadoop/etc/hadoop/mapred-site.xml

scp yarn-site.xml slave1:/usr/local/hadoop/etc/hadoop/yarn-site.xml
scp yarn-site.xml slave2:/usr/local/hadoop/etc/hadoop/yarn-site.xml
scp yarn-site.xml slave3:/usr/local/hadoop/etc/hadoop/yarn-site.xml
```



``` java
{
	/**
	 * hadoop安装中的要点理解
	 */
	1 机器（硬件的资源）
	2 节点 （装系统）
	3 网络 （ip分配）
	4 主从 （备份管理等）
	5 集群管理


	/////////////linx安装
	///查看/etc/profile 文件查看JAVA环境 
	export JAVA_HOME=/usr/local/jdk1.8
	//安装解压包
	tar -zxvf hadoop-2.7.3.tar.gz -C /usr/local/hadoop
	//修改hadoop-evn.sh 添加JAVA环境变量
	export JAVA_HOME=/usr/local/jdk1.8

	//在/etc/profile文件中加入：
	export PATH=$PATH:/usr/local/hadoop/hadoop-2.7.3/bin:/usr/local/hadoop/hadoop-2.7.3/sbin

	//执行/etc/profile文件：
	source /etc/profile
	
	///////////////这里hadoop已经可以启动了
	///配置文件的路径
	/usr/local/hadoop/hadoop-2.7.3/etc/hadoop
	//修改节点名称与关系 core-site.xml
	
	//主机的ip重新命名
	在进行Hadoop集群配置中，需要在”/etc/hosts”文件中添加集群中所有机器的IP与主机名，
	这样Master与所有的Slave机器之间不仅可以通过IP进行通信，
	而且还可以通过主机名进行通信。所以在所有的机器上的”/etc/hosts”文件中都要添加如下内容：

	192.168.1.141 Master.Hadoop

	192.168.1.142 Slave1.Hadoop

	192.168.1.137 Slave2.Hadoop

	core-site中修改：
	
}
```








