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

#Formatting the HDFS filesystem via the NameNode
The first step to starting up your Hadoop installation is formatting the Hadoop filesystem.
```
hadoop namenode -format
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
```

#NameNode
```
http://localhost:50070
```

#Hadoop File System Commands
You can reach file system commands from http://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-common/FileSystemShell.html

#Create a directory in HDFS
```
Usage:
hadoop fs -mkdir [-p] <paths>

Options:
The -p option behavior is much like Unix mkdir -p, creating parent directories along the path.

Example:
hadoop fs -mkdir /user/hadoop
hadoop fs -mkdir /user/hadoop/dir1 /user/hadoop/dir2 /user/hadoop/dir3
```

#List the contents of a directory
```
Usage:
Usage: hadoop fs -ls [-d] [-h] [-R] <args>

Options:
-d: Directories are listed as plain files.
-h: Format file sizes in a human-readable fashion (eg 64.0m instead of 67108864).
-R: Recursively list subdirectories encountered.

Example:
hadoop fs -ls /user/hadoop
hadoop fs -ls /user/hadoop/dir1
hadoop fs -ls /user/hadoop/dir1/names.txt
```

#Upload a file from local file system to HDFS
```
Usage:
hadoop fs -put <local-src> ... <HDFS_dest_path>

Copy single src, or multiple srcs from local file system to the destination file system. Also reads input from stdin and writes to destination file system.

Example:
hadoop fs -put names.txt /user/hadoop/dir1/names.txt
hadoop fs -put /home/testtest/Samplefile.txt  /user/testtest/dir3/
hadoop fs -put localfile /user/hadoop/hadoopfile
hadoop fs -put localfile1 localfile2 /user/hadoop/hadoopdir
hadoop fs -put localfile hdfs://nn.example.com/hadoop/hadoopfile
hadoop fs -put - hdfs://nn.example.com/hadoop/hadoopfile Reads the input from stdin.
```

#Download a file from HDFS to local file system
```
Usage:
hadoop fs -get [-ignorecrc] [-crc] <src> <localdst>

Example:
hadoop fs -get /user/hadoop/dir1/names.txt/home/
```

#Copy a file from Local file system to HDFS
```
Usage:
hadoop fs -copyFromLocal <localsrc> URI

Similar to put command, except that the source is restricted to a local file reference.

Example:
hadoop fs -copyFromLocal /home/hadoop/abc.txt  /user/hadoop/abc.txt
```

#Copy a file from HDFS to local file system
```
Usage:
hadoop fs -copyToLocal [-ignorecrc] [-crc] URI <localdst>

Similar to get command, except that the destination is restricted to a local file reference.

Example:
hadoop fs -copyToLocal /user/hadoop/purchases.txt /home/training/data
```

#Copy a file from source to destination
```
Usage:
hadoop fs -cp [-f] [-p | -p[topax]] URI [URI ...] <dest>

Copy files from source to destination. This command allows multiple sources as well in which case the destination must be a directory.

Options:
	The -f option will overwrite the destination if it already exists.
	The -p option will preserve file attributes [topx] (timestamps, ownership, permission, ACL, XAttr). If -p is specified with no arg, then preserves timestamps, ownership, permission. If -pa is specified, then preserves permission also because ACL is a super-set of permission. Determination of whether raw namespace extended attributes are preserved is independent of the -p flag.
	
Example:
hadoop fs -cp /user/hadoop/dir1/abc.txt /user/hadoop/dir2
hadoop fs -cp /user/hadoop/file1 /user/hadoop/file2
```

#Move file from source to destination
```
Usage:
hadoop fs -mv URI [URI ...] <dest>

Moves files from source to destination. This command allows multiple sources as well in which case the destination needs to be a directory. Moving files across file systems is not permitted.

Example:
hadoop fs -mv /user/hadoop/file1 /user/hadoop/file2
hadoop fs -mv hdfs://nn.example.com/file1 hdfs://nn.example.com/file2 hdfs://nn.example.com/file3 hdfs://nn.example.com/dir1
hadoop fs -mv /user/hduser/dir1/abc.txt /user/hadoop/dir2
```

#See contents of a file
```
Usage:
hadoop fs -cat URI [URI ...]

Copies source paths to stdout.

Example:
hadoop fs -cat /user/hadoop/dir1/abc.txt
hadoop fs -cat hdfs://nn1.example.com/file1 hdfs://nn2.example.com/file2
hadoop fs -cat file:///file3 /user/hadoop/file4
```

#Remove a file or directory in HDFS
```
Usage:
Usage: hadoop fs -rm [-f] [-r |-R] [-skipTrash] URI [URI ...]

Remove files specified as argument. Deletes directory only when it is empty

Options:
	The -f option will not display a diagnostic message or modify the exit status to reflect an error if the file does not exist.
	The -R option deletes the directory and any content under it recursively.
	The -r option is equivalent to -R.
	The -skipTrash option will bypass trash, if enabled, and delete the specified file(s) immediately. This can be useful when it is necessary to delete files from an over-quota directory.
	
Example:
hadoop fs -rm hdfs://nn.example.com/file /user/hadoop/emptydir
hadoop fs -rm /user/hduser/dir1/abc.txt
```

#####Recursive version of delete
```
hadoop fs -rmr /user/hadoop/
```

#Display the aggregate length of a file
```
Usage:
hadoop fs -du [-s] [-h] URI [URI ...]

Displays sizes of files and directories contained in the given directory or the length of a file in case its just a file.

Options:
	The -s option will result in an aggregate summary of file lengths being displayed, rather than the individual files.
	The -h option will format file sizes in a “human-readable” fashion (e.g 64.0m instead of 67108864)
	
Example:
hadoop fs -du /user/hadoop/dir1 /user/hadoop/file1 hdfs://nn.example.com/user/hadoop/dir1
```

#Copy a directory from one node in the cluster to another
```
hadoop fs -distcp hdfs://namenodeA/apache_hadoop hdfs://namenodeB/hadoop
```

#Merge results from HDFS and copy local file
```
hadoop fs -getmerge /user/hduser/helloWorldOutput_WorldCount/ /user/hduser/helloWorldOutput_WorldCount2 ~/tmp/output/test.txt
```

Output:
```
hduser@ubuntu:~$ cat tmp/output/test.txt
hello	3
world	2
hadoop,	1
hello	5
mapreduce!	1
world	2
hduser@ubuntu:~$ 
```

#Simple Example
You can copy job result from HDFS to the local file system. Alternatively, you can use cat command to read the file directly from HDFS without copying it to the local file system.
```
echo kenan enes enejda hasan hüseyin ahmet mehmet kadir cemil vedat ergün güler bekir halit > names1.txt
echo bilal serdar selçuk kenan enes hasan deniz halil sinan özlem cemalettin kemal nasibe pakize hasibe faruk ahmet mehmet oğuz vahap > names2.txt
cat names1.txt names2.txt >> fullnames.txt

hadoop fs -put fullnames.txt fullnames

hadoop fs -cat fullnames

hadoop jar /home/kenan/Desktop/wordcountf/wordcount.jar fullnames fullnames-output

hadoop fs -cat fullnames-output/part-r-00000
```

Output:
```
ahmet	2
bekir	1
bilal	1
cemalettin	1
cemil	1
deniz	1
enejda	1
enes	2
ergün	1
faruk	1
güler	1
halil	1
halit	1
hasan	2
hasibe	1
hüseyin	1
kadir	1
kemal	1
kenan	2
mehmet	2
nasibe	1
oğuz	1
pakize	1
selçuk	1
serdar	1
sinan	1
vahap	1
vedat	1
özlem	1
```
