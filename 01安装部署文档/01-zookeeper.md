### zookeeper安装(单机)

```
#安装
tar -zxf zookeeper.3.4.13.tar.gz #解压
mv zookeeper.3.4.13 zookeeper 
cd zookeeper/conf 
cp zoo_sample.cfg zoo.cfg #复制到zoo.cfg
vim zoo.cfg #修改zoo.cfg
----------------------------
dataDir=/usr/local/zookeeper/server/server1/data
dataDirLog=/usr/local/zookeeper/server/server1/data/log 
clientPort=2181 #客户端端口

#集群需要配置
#server.1=192.168.254.132:2887:3887
#server.2=192.168.254.132:2888:3888
#server.3=192.168.254.132:2889:3889
----------------------------
#集群
cd /usr/local/zookeeper/server/server1/data
vim myid
依次写入:1 2 3


#开放端口
iptables -A INPUT -p tcp --dport 2181 -j ACCEPT  #开放端口(2181,2182,2183)
/etc/rc.d/init.d/iptables save #保存修改
/etc/rc.d/init.d/iptables restart #重启
iptables -L -n #查看开放了哪些端口

#启动
bin/zkServer.sh start conf/zoo.cfg #启动zk
bin/zkServer.sh status conf/zoo.cfg #查看状态

./zkCli.sh start #启动客户端
telent 

./zkServer.sh stop #关闭
启动正常,查看启动状态:
zookeeper/zookeeper/bin/zkServer.sh status zookeeper/zookeeper/conf/zoo1.cfg
报错：Error contacting service. It is probably not running.
cat zookeeper.out #查看zookeeper日志
-----------------------------------
报错日志：Cannot open channel to 3 at election address /192.168.254.132:3889
java.net.ConnectException: 拒绝连接 (Connection refused)
原因：还没有启动另外两个zookeeper

-----------------------------------

全部启动,zookeeper.out
-----------------------------------
报错：Unexpected exception, exiting abnormally
java.lang.RuntimeException: My id 333 not in the peer list
原因：zoo3.cfg 中配置的是server.3
server3/data/myid中配置的是333,不对应所以报错

-----------------------------------
```

单机集群(正常启动)： 





### ZKUI

先安装maven

```
#编译
unzip zkui-master.zip
cd zkui-master
mvn clean install #重新编译

#安装
mkdir /usr/local/zk/data/zkui
cp zkui-master/config.cfg zkui-master/target/zkui-2.0-SNAPSHOT-jar-with-dependencies.jar /usr/local/zk/data/zkui/
#拷贝config.cfg和jar包到data/zkui目录下

#配置
vim /usr/local/zk/data/zkui/config.cfg
--------------------------
serverPort=9090
zkServer=192.168.254.132:2181,192.168.254.132:2182,192.168.254.132:2183
userSet = {"users": [{ "username":"admin" , "password":"NewPassword","role": "ADMIN" },{ "username":"appconfig" , "NewPassword":"","role": "USER" }]}
--------------------------
```

启动

```
cd /usr/local/zk/data/zkui/ && nohup java -jar zkui-2.0-SNAPSHOT-jar-with-dependencies.jar &
```

开机启动

```
# vim /etc/rc.local
source /etc/profile
cd /usr/local/zk/data/zkui && nohup java -jar zkui-2.0-SNAPSHOT-jar-with-dependencies.jar &

# chmod +x /etc/rc.d/rc.local
```

访问

```
http://192.168.254.132:9090
如果访问不了,需要关闭防火墙service iptables stop
```