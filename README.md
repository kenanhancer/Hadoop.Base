# Hadoop.Base
Installation of Hadoop on Ubuntu (Single-Node Cluster)


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

You can find hadoop mirror site for your download http://www.apache.org/dyn/closer.cgi/hadoop/common/
