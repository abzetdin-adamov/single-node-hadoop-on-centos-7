# HADOOP on CENTOS 7

##================================DOWNLOAD and CONFIGURE ENVIRONMENT

**1) specify hostname** 
hostnamectl set-hostname namenode

specify IP address and FQDN
vi /etc/hosts

**2) Install JDK**
curl -LO -H "Cookie: oraclelicense=accept-securebackup-cookie" “http://download.oracle.com/otn-pub/java/jdk/8u121-b13/e9e7ea248e2c4826b92b3f075a80e441/jdk-8u121-linux-x64.rpm”

Install Package
rpm -Uvh jdk-8u121-linux-x64.rpm

**3) Create user**
useradd -d /home/hadoop hadoop
passwd hadoop

**4) Download Hadoop**
curl -O http://apache.javapipe.com/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz

**5) Extract archive into hadoop folder and change ownership appropriately** 
tar xfz hadoop-2.7.3.tar.gz
cp -rf hadoop-2.7.3/* /home/hadoop/
chown -R hadoop:hadoop /home/hadoop/

**6) Login as a hadoop user and set environment variables for Hadoop and Java editing .bash_profile of home folder of user**

su - hadoop
vi .bash_profile
 
Add these lines

--- JAVA
export JAVA_HOME=/usr/java/default
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar

--- HADOOP
export HADOOP_HOME=/home/hadoop/hadoop-2.7.3

export HADOOP_COMMON=$HADOOP_HOME

export HADOOP_HDFS=$HADOOP_HOME

export HADOOP_MAPRED=$HADOOP_HOME

export HADOOP_YARN=$HADOOP_HOME

export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"

export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native

export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin

**7) run .bash_profile check variables**
source .bash_profile
echo $HADOOP_HOME
echo $JAVA_HOME

**8) set ssh authentication for hadoop user at specific host (FQDN) and leave passphrase empty**
ssh-keygen -t rsa
ssh-copy-id namenode.hadoop.net

##================================HADOOP CONFIG

**9) Set configuration files at $HADOOP_HOME/etc/hadoop** 

--- core-site.xml configuration of core Hadoop
vi $HADOOP_HOME/etc/hadoop/core-site.xml

Add following text between <configuration> and </configuration> tags
<property>
<name>fs.defaultFS</name>
<value>hdfs://namenode.hadoop.net:9000/</value>
</property>

---hdfs-site.xml configuaration of HDFS
vi $HADOOP_HOME/etc/hadoop/hdfs-site.xml

Add following text between <configuration> and </configuration> tags
<property>
    <name>dfs.data.dir</name>
    <value>file:///home/hadoop/volume/datanode</value>
</property>
<property>
    <name>dfs.name.dir</name>
    <value>file:///home/hadoop/volume/namenode</value>
</property>

---mapred-site.xml to specify that we are using yarn MapReduce
vi $HADOOP_HOME/etc/hadoop/mapred-site.xml

Add following content
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
</configuration>

---yarn-site.xml specifies job for yarn
vi $HADOOP_HOME/etc/hadoop/yarn-site.xml

Add following text between <configuration> and </configuration> tags
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>

**10) Create datanode and namenode folders (declared in hdfs-site.xml) for HDFS as a root and grant permission to hadoop user**
su root
mkdir -p /home/hadoop/volume/namenode
mkdir -p /home/hadoop/volume/datanode
chown -R hadoop:hadoop /home/hadoop/volume/
ls -al /home/hadoop/  #Verify permissions
exit  #Exit root account to turn back to hadoop user

**11) Set Java home variable for Hadoop editing hadoop-env.sh**
vi $HADOOP_HOME/etc/hadoop/hadoop-env.sh

find the string "The java implemention to use" and edit as
export JAVA_HOME=/usr/java/default/

**12) replace FQDN at slave file**
vi $HADOOP_HOME/etc/hadoop/slaves 

replace in our case localhost to "namenode.hadoop.net" 

##================================START HADOOP

**13) Format HDFS**
hdfs namenode -format

**14) Start Hadoop (HDFS and YARN) using commands at $HADOOP_HOME/sbin folder**
start-dfs.sh
start-yarn.sh

jps - to check java processes

===HDFS DEAMONS:
-NameNode
-DataNode
-SecondaryNameNode

===YARN DEAMONS:
-Resource Manager
-NodeManager

===Getting Report on HDFS
hdfs dfsadmin -report

**14) Copy files from local to HDFS**
hdfs dfs -mkdir /my_storage
hdfs dfs -put LICENSE.txt /my_storage

**15) View HDFS file list and content of file**
hdfs dfs -ls /my_storage/
hdfs dfs -cat /my_storage/LICENSE.txt

