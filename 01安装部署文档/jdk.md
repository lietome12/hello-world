# yum方式安装

```
# 安装jdk1.8
yum install java-1.8.0-openjdk java-1.8.0-openjdk-devel
# jdk版本
java -version
```

# 安装包方式安装

1.解压

```
tar -zxvf jdk-8u181-linux-x64.tar.gz
```

2.配置环境变量

```
vi /etc/profile
// 修改内容
# JAVA环境变量
export JAVA_HOME=/home/java1.8/jdk1.8.0_181
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}
# linux最大句柄数
ulimit -n 65535
// 保存，刷新文件
source /etc/profile
```

3.查看jdk版本

```
java -version
```

如果结果是openJDK，需要先卸载openJDK

```
// 查看安装了哪些和java相关的
rpm -qa | grep java
// 删除所有和openJDK相关
rpm -e --nodeps [target]
// 再次刷新文件
source /etc/profile
// 查看版本
java -version
```