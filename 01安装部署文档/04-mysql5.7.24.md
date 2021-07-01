1.解压

```
tar -zxf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```

2.修改文件名（移动文件）

```
mv mysql-5.7.24-linux-glibc2.12-x86_64 /usr/local/mysql
```

3.检查mysql用户组和用户是否存在，不存在则创建

```
cat /etc/group | grep mysql
cat /etc/passwd | grep mysql
```

4.新增mysql用户组

```
groupadd mysql
```

5.新增用户

```
useradd -r -g mysql mysql
```

6.创建data和log目录

```
cd /usr/local/mysql
mkdir data
mkdir log
```

7.为mysql目录赋权限

```
chown -R mysql.mysql /usr/local/mysql
```

新增mysql日志文件，并为其赋权

```
cd log
touch mysqld.log
chmod 777 mysqld.log
```

8.初始化数据，指定安装目录和数据目录

```
// 需要进入mysql用户下操作
su - mysql
// 创建软连接
ln -s /usr/local/mysql/data/mysql.sock /tmp/mysql.sock
// 注意：此步骤要记住打印的密码，后边登录需要
./bin/mysqld --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --initialize
// 退出mysql用户
```

9.复制启动文件

```
cp -a ./support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
```

10.修改启动路径

```
vi /etc/init.d/mysqld
```

![img](file:///E:/software/WeiZhi/temp/d7b32a95-ecc0-4729-a4c0-1f0f11b47844/128/index_files/17947312.png)basedir bindir datadir sbindir libexecdir11.增加环境变量

```
vi /etc/profile
// 增加内容
export MYSQL_HOME="/usr/local/mysql"
export PATH="$PATH:$MYSQL_HOME/bin"
```

12.刷新环境变量文件

```
source /etc/profile
```

13.修改配置文件

```
vi /etc/my.cnf
// 修改内容
[client]
port=3310
[mysqld]
user=mysql
#Mysql服务的唯一编号 每个mysql服务Id需唯一
server-id = 1
#服务端口号 默认3306
port=3310
#mysql安装根目录
basedir=/usr/local/mysql
#mysql数据文件所在位置
datadir=/usr/local/mysql/data
#临时目录 比如load data infile会用到
tmpdir  = /tmp
#设置socke文件所在目录
socket=/tmp/mysql.sock
#主要用于MyISAM存储引擎,如果多台服务器连接一个数据库则建议注释下面内容
skip-external-locking
#只能用IP地址检查客户端的登录，不用主机名
skip_name_resolve = 1
#事务隔离级别，默认为可重复读，mysql默认可重复读级别（此级别下可能参数很多间隙锁，影响性能）
transaction_isolation = READ-COMMITTED
#数据库默认字符集,主流字符集支持一些特殊表情符号（特殊表情符占用4个字节）
character-set-server = utf8mb4
#数据库字符集对应一些排序等规则，注意要和character-set-server对应
collation-server = utf8mb4_general_ci
#设置client连接mysql时的字符集,防止乱码
init_connect='SET NAMES utf8mb4'
#是否对sql语句大小写敏感，1表示不敏感
lower_case_table_names = 1
#最大连接数
max_connections = 400
#最大错误连接数
max_connect_errors = 1000
#TIMESTAMP如果没有显示声明NOT NULL，允许NULL值
explicit_defaults_for_timestamp = true
#SQL数据包发送的大小，如果有BLOB对象建议修改成1G
max_allowed_packet = 128M
#MySQL连接闲置超过一定时间后(单位：秒)将会被强行关闭
#MySQL默认的wait_timeout  值为8个小时, interactive_timeout参数需要同时配置才能生效
interactive_timeout = 1800
wait_timeout = 1800
#内部内存临时表的最大值 ，设置成128M。
#比如大数据量的group by ,order by时可能用到临时表，
#超过了这个值将写入磁盘，系统IO压力增大
tmp_table_size = 134217728
max_heap_table_size = 134217728
##----------------------------用户进程分配到的内存设置BEGIN-----------------------------##
##每个session将会分配参数设置的内存大小
#用于表的顺序扫描，读出的数据暂存于read_buffer_size中，当buff满时或读完，将数据返回上层调用者
#一般在128kb ~ 256kb,用于MyISAM
#read_buffer_size = 131072
#用于表的随机读取，当按照一个非索引字段排序读取时会用到，
#一般在128kb ~ 256kb,用于MyISAM
#read_rnd_buffer_size = 262144
#order by或group by时用到
#建议先调整为2M，后期观察调整
sort_buffer_size = 2097152
#一般数据库中没什么大的事务，设成1~2M，默认32kb
binlog_cache_size = 524288
##---------------------------用户进程分配到的内存设置END-------------------------------##
#在MySQL暂时停止响应新请求之前的短时间内多少个请求可以被存在堆栈中
#官方建议back_log = 50 + (max_connections / 5),封顶数为900
back_log = 130
#资源缓冲区  ************
innodb_buffer_pool_size=1024M
#解决sql_mode=only_full_group_by的问题
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
#skip-grant-tables
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd
[mysqld_safe]
log-error=/usr/local/mysql/log/mysqld.log
pid-file=/usr/local/mysql/data/mysqld.pid
#
# include all files from the config directory
#
#!includedir /etc/my.cnf.d
```

在配置完成后，修改mysqk目录权限，其中data、log给所有的操作权限

```
chmod 777 data
chmod 777 log
```

14.切换到mysql用户下

```
su - mysql
//
./bin/mysqld --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --initialize
// 启动mysql
service mysqld start
// 登录mysql,回车输入密码(参见第8步)
mysql -uroot -p
// 修改密码
alter user 'root'@'localhost' identified by 'youpassword';
// 修改root用户的远程访问权限
update user set host='%' where user='root' LIMIT 1;
// 刷新权限
flush privileges;
help contents;
// 退出mysql和mysql用户
```

15.加入开机自启动

```
chkconfig --add mysqld
chkconfig mysqld on
// 查看是否加入成功，第3个是on说明成功
chkconfig --list
```