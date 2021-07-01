##### 下載

```
http://repo.mysql.com/mysql80-community-release-el6.rpm
或
https://dev.mysql.com/downloads/repo/yum/ 找到对应centos版本
```

##### 安装rpm包

```
sudo rpm -Uvh mysql80-community-release-el6.rpm
yum repolist all | grep mysql
```

##### 安装mysql

```
sudo yum install mysql-community-server
(y/n 输入y即可)
```

##### 启动

```
sudo service mysqld start 
或 
sudo systemctl start mysqld.service
```

##### 查看状态

```
sudo service mysqld status
或
sudo systemctl status mysqld.service
```

##### 密码修改

安装完成后,系统系统默认添加root'@'localhost，随机生成了密码，此时使用mysql命令会失败，其密码添加到了异常日志里，

###### 查看密码

```
sudo grep 'temporary password' /var/log/mysqld.log
----------------------------------------
2019-02-21T14:04:52.039987Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: -a?C-rjs/8Gn
----------------------------------------
```

###### 修改密码

```
mysql -uroot -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'newPassword';
```

mysql默认安装位置

```
1.数据库目录
/var/lib/mysql
2.配置文件(mysql.server命令及配置文件)
/usr/share/mysql
3.相关命令(mysqladmin mysqldump等)
/usr/bin
4.启动脚本
/etc/rc.d/init.d/
```

开机启动

```
vi /etc/rc.local 添加service mysqld start
```



------

##### 修改密码

```
  service mysqld stop #停止mysql
  mysqld_safe --user=mysql --skip-grant-tables --skip-networking & #使用mysqld_safe模式启动mysqld服务器
  
  mysql -u root -p #无密码登录
  flush privileges；
  ALTER USER ‘root’@‘localhost’ IDENTIFIED BY ‘新密码’
  exit
  
  service mysqld restart #重启
  
  mysql -u root -p #重新登录
```

查看是否已经安装：`yum list installed | grep mysql`

之前有安装过的或者装失败的，请先卸载干净后再重新安装

查看：`rpm -qa | grep mysql`

卸载：`yum -y remove 包名`





**navicat连接不上服务器的mysql**

错误：*ERROR 1130 (HY000): Host '192.168.3.134' is not allowed to connect to this MySQL server*

1.服务器是否关闭防火墙
2.3306端口是否对外开放
3.权限不够

```
##修改表
mysql -u root -proot123
use mysql;
update user set host = '%' where user = 'root';
select host, user from user; 

或

##授权
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'root123' WITH GRANT OPTION;
FLUSH   PRIVILEGES; 
```

错误：*Client does not support authentication protocol requested by server*

```
alter user 'root'@'%' identified with mysql_native_password by 'root123';
flush privileges;
```