# Hadoop.Base
Installation of Hadoop on Ubuntu (Single-Node Cluster)


##Install Java
```csharp
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

###Check Java
```csharp
java -version
```
Output will be like below
```csharp
kenan@ubuntu:~$ java -version
java version "1.8.0_77"
Java(TM) SE Runtime Environment (build 1.8.0_77-b03)
Java HotSpot(TM) 64-Bit Server VM (build 25.77-b03, mixed mode)
```

###Set JAVA_HOME environment variable
List installed java paths
```csharp
update-java-alternatives --list
```

Output will be like below
```csharp
kenan@ubuntu:~$ update-java-alternatives --list
java-8-oracle 1 /usr/lib/jvm/java-8-oracle
```

Open system scope environment
```csharp
sudo gedit /etc/environment
```

Add below code line end of the file, save and close.
```csharp
JAVA_HOME="/usr/lib/jvm/java-8-oracle"
```

Reload system environment
```csharp
source /etc/environment
```
