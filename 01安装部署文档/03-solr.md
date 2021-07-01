### 准备

需要先安装Java和tomcat(java8不支持tomcat7一下版本)

```
mkdir -p /data/solr6/home
mkdir -p /data/solr6/logs
```

解压安装包

```
unzip -oq solr-6.5.0.zip -d /usr/local
```

将solr所需文件及jar复制到tomcat中

```
cd /usr/local/tomcat8/webapps

#删除其它项目
rm -rf ./*
mkdir solr
cd solr

#复制solr运行所需文件
cp -r /usr/local/solr-6.5.0.zip/server/solr-webapp/webapp/* ./
cp -r /usr/local/solr-6.5.0.zip/server/lib/ext/* WEB-INF/lib/
cp -r /usr/local/solr-6.5.0.zip/server/lib/metrics*.* WEB-INF/lib/
cp -r /usr/local/solr-6.5.0.zip/dist/solr-dataimporthandler-* WEB-INF/lib/
```

日志

```
mkdir -p WEB-INF/classes
cp /usr/local/src/solr-6.5.0.zip/server/resources/log4j.properties WEB-INF/calsses/

vim WEB-INF/classes/log4j.properties
-------------------------
solr.log=/data/solr6/logs
log4j.rootLogger=INFO,file,CONSOLE

-------------------------
```

指定solr配置目录

```
vim WEB-INF/web.xml
---------------------------------
放开<env-entry>
注释掉<security-constraint>

---------------------------------
```

配置solr运行的配置

```
cd /data/solr6/home
cp -r /usr/local/src/solr-6.5.0/server/solr/* ./
cp -r /usr/local/src/solr-6.5.0/contrib/ ./
cp -r /usr/local/src/solr-6.5.0/dist/ ./

#修改tomcat端口号(可不修改,修改后需要放开该port,否则可能访问不到)
#/sbin/iptables -I INPUT -p tcp --dport 18983 -j ACCEPT
vim /usr/local/tomcat/conf/server.xml
```

启动tomcat

```
http://192.168.254.132:18983/solr/index.html/
```