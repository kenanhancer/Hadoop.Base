# Hadoop.Base
Installation of Hadoop on Ubuntu (Single-Node Cluster)

##Install Gedit
```
sudo apt-get remove gedit
sudo apt-get install gedit
```

##Install Java
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

#####Check Java
```
java -version
```
Output will be like below
```
kenan@ubuntu:~$ java -version
java version "1.8.0_77"
Java(TM) SE Runtime Environment (build 1.8.0_77-b03)
Java HotSpot(TM) 64-Bit Server VM (build 25.77-b03, mixed mode)
```

#####Set JAVA_HOME environment variable
List installed java paths
```
update-java-alternatives --list
```

Output will be like below
```
kenan@ubuntu:~$ update-java-alternatives --list
java-8-oracle 1 /usr/lib/jvm/java-8-oracle
```

Open system scope environment
```
sudo gedit /etc/environment
```

Add below code line end of the file, save and close.
```
JAVA_HOME="/usr/lib/jvm/java-8-oracle"
```

Reload system environment
```
source /etc/environment
```

##Install Hadoop

#####Adding a dedicated Hadoop system user
```
sudo addgroup hadoop
sudo adduser --ingroup hadoop hduser
sudo adduser hduser sudo
su hduser
```

Output will be like below
```
kenan@ubuntu:~$ sudo addgroup hadoop
Adding group `hadoop' (GID 1001) ...
Done.
kenan@ubuntu:~$ sudo adduser --ingroup hadoop hduser
Adding user `hduser' ...
Adding new user `hduser' (1001) with group `hadoop' ...
Creating home directory `/home/hduser' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for hduser
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] y
kenan@ubuntu:~$ 
```

#####Configuring SSH
We have to generate an SSH key for the hduser user.
```
sudo apt-get install openssh-server
/etc/init.d/ssh start
sudo su hduser
ssh-keygen -t rsa -P ""
```

Output will be like below
```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hduser/.ssh/id_rsa): 
Created directory '/home/hduser/.ssh'.
Your identification has been saved in /home/hduser/.ssh/id_rsa.
Your public key has been saved in /home/hduser/.ssh/id_rsa.pub.
The key fingerprint is:
14:e3:82:8f:48:46:1f:ee:c8:3c:fc:85:2c:61:1b:65 hduser@ubuntu
The key's randomart image is:
```

You have to enable SSH access to your local machine with this newly created key.
```
cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
```

#####Fetch and Install Hadoop
You can find hadoop mirror site for your download http://www.apache.org/dyn/closer.cgi/hadoop/common/

```
cd /usr/local
sudo wget "http://mirror.idealhosting.net.tr/Apache/hadoop/common/stable/hadoop-2.7.2.tar.gz"
sudo tar -xzvf hadoop-2.7.2.tar.gz
sudo mv hadoop-2.7.2 /usr/local/hadoop
sudo chown hduser:hadoop -R /usr/local/hadoop
```

Creating namenode and datanode folders
```
sudo mkdir -p /usr/local/hadoop_store/hdfs/namenode
sudo mkdir -p /usr/local/hadoop_store/hdfs/datanode
sudo chown hduser:hadoop -R /usr/local/hadoop_store/
```

#####Edit and Setup Configuration Files
To complete the setup of Hadoop, the following files will have to be modified:

- ~/.bashrc
- ~~/usr/local/hadoop/etc/hadoop/hadoop-env.sh~~(doesn't need to update it. Because, JAVA_HOME is updated in /etc/environment)
- /usr/local/hadoop/etc/hadoop/core-site.xml
- /usr/local/hadoop/etc/hadoop/yarn-site.xml
- /usr/local/hadoop/etc/hadoop/mapred-site.xml.template
- /usr/local/hadoop/etc/hadoop/hdfs-site.xml

#####Updating $HOME/.bashrc
```
sudo gedit $HOME/.bashrc
```

This will open the .bashrc file in a text editor. Go to the end of the file and paste/type the following content in it:
```
# -- HADOOP -- #
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
# -- HADOOP -- #
```

After saving and closing the .bashrc file, execute the following command so that your system recognizes the newly created environment variables:
```
source $HOME/.bashrc
```

#####Updating /usr/local/hadoop/etc/hadoop/core-site.xml
```
sudo gedit /usr/local/hadoop/etc/hadoop/core-site.xml
```

This will open the core-site.xml file in a text editor. Enter the following content in between the <configuration></configuration> tag.
```
<property>
   <name>fs.default.name</name>
   <value>hdfs://localhost:9000</value>
</property>
```

#####Updating /usr/local/hadoop/etc/hadoop/yarn-site.xml
```
sudo gedit /usr/local/hadoop/etc/hadoop/yarn-site.xml
```

This will open the yarn-site.xml file in a text editor. Enter the following content in between the <configuration></configuration> tag.
```
<property>
   <name>yarn.nodemanager.aux-services</name>
   <value>mapreduce_shuffle</value>
</property>
<property>
   <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
   <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
```

#####Creating and Updating /usr/local/hadoop/etc/hadoop/mapred-site.xml
```
sudo cp /usr/local/hadoop/etc/hadoop/mapred-site.xml.template /usr/local/hadoop/etc/hadoop/mapred-site.xml
sudo gedit /usr/local/hadoop/etc/hadoop/mapred-site.xml
```

This will open the mapred-site.xml file in a text editor. Enter the following content in between the <configuration></configuration> tag.
```
<property>
   <name>mapreduce.framework.name</name>
   <value>yarn</value>
</property>
```

#####Updating /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```
sudo gedit /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```

This will open the hdfs-site.xml file in a text editor. Enter the following content in between the <configuration></configuration> tag.
```
<property>
   <name>dfs.replication</name>
   <value>1</value>
 </property>
 <property>
   <name>dfs.namenode.name.dir</name>
   <value>file:/usr/local/hadoop_store/hdfs/namenode</value>
 </property>
 <property>
   <name>dfs.datanode.data.dir</name>
   <value>file:/usr/local/hadoop_store/hdfs/datanode</value>
 </property>
<property>
    <name>dfs.http.address</name>
    <value>localhost:50070</value>
</property>
<property>
    <name>dfs.datanode.http.address</name>
    <value>localhost:50075</value>
</property>
```

#Start HDFS and YARN manually
Start NameNode
```
hadoop-daemon.sh --script hdfs start namenode
```

Start DataNode
```
hadoop-daemon.sh --script hdfs start datanode
```

Start Yarn ResourceManager
```
yarn-daemon.sh start resourcemanager
```

Start Yarn NodeManager
```
yarn-daemon.sh start nodemanager
```

#Start HDFS and YARN with script
```
start-dfs.sh
start-yarn.sh
```

#####Checking whether the Hadoop processes are running
```
jps
```

Output will be like below:
```
hduser@ubuntu:/usr/local$ jps
6978 Jps
6680 NodeManager
5881 NameNode
6426 ResourceManager
6251 SecondaryNameNode
6047 DataNode
```

#Stopping HDFS and YARN manually
Stop NameNode
```
hadoop-daemon.sh --script hdfs stop namenode
```

Stop DataNode
```
hadoop-daemon.sh --script hdfs stop datanode
```

Stop Yarn ResourceManager
```
yarn-daemon.sh stop resourcemanager
```

Stop Yarn NodeManager
```
yarn-daemon.sh stop nodemanager
```

#Stopping HDFS and YARN with script
```
stop-dfs.sh
stop-yarn.sh
```

#Check HDFS status
```
hdfs dfsadmin -report
```

It should show a report like:
```
Configured Capacity: 18889830400 (17.59 GB)
Present Capacity: 12390932480 (11.54 GB)
DFS Remaining: 12390899712 (11.54 GB)
DFS Used: 32768 (32 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0
Missing blocks (with replication factor 1): 0

-------------------------------------------------
Live datanodes (1):

Name: 127.0.0.1:50010 (localhost)
Hostname: ubuntu
Decommission Status : Normal
Configured Capacity: 18889830400 (17.59 GB)
DFS Used: 32768 (32 KB)
Non DFS Used: 6498897920 (6.05 GB)
DFS Remaining: 12390899712 (11.54 GB)
DFS Used%: 0.00%
DFS Remaining%: 65.60%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Tue Apr 12 05:54:12 PDT 2016
```

#Check YARN status
```
yarn node -list
```


It should show a report like:
```
Total Nodes:1
         Node-Id	     Node-State	Node-Http-Address	Number-of-Running-Containers
    ubuntu:38478	        RUNNING	      ubuntu:8042	                           0
```


#ResourceManager

```
http://localhost:8088 

http://ipaddress:8088
```

#NameNode

```
http://localhost:50070

http://ipaddress:50070
```
