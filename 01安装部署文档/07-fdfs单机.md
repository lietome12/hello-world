### 软件包

```
CentOS 6.5
fastdfs-master-V5.05.zip
fastdfs-nginx-module-master.zip
libfastcommon-master.zip
nginx-1.13.7.zip
```

### 编译环境

```
yum install unzip zip gcc-c++
```

### 安装libfastcommon

首先在 /usr/local/fastdfs路径下上传所有需要的压缩包。

安装 libfastcommon模块。解压缩libfastcommon

```
unzip libfastcommon-master.zip 解压
cd libfastcommon-master 
./make.sh  编译完成
./make.sh install 安装
```

### 安装fastDfs

#### 1.安装fastdfs-master

```
unzip fastdfs-master.zip
cd fastdfs-master
./make.sh && ./make.sh install
```

#### 2.复制fastdfs-master目录的配置文件到/etc/fdfs

```
cp -r conf/* /etc/fdfs
```

#### 3.修改tracker.conf和storage.conf以及client.conf(/etc/fdfs内)

tracker.conf

```
prot=22122  #默认值
base_path=/home/fastdfs/tracker #tracker的data和log目录
```

storage.conf

```
port=23000
base_path=/home/fastdfs/storage #存放storage的data和log目录
store_path0=/home/fastdfs/storage #上传文件的存放目录
tracker_server=192.168.254.131:22122 #tracker服务地址
http.server_port=8888 #文件通过http访问的端口
```

client.conf

```
tracker_server=192.168.254.131:22122
```

#### 4.安装fastdfs-nginx-module

fastDFS通过trakcer将文件放在storage服务器存储,但是同group的存储器之间需要进行文件复制,有同步延迟的问题。 当客户端把文件上传到一个storage后，再从storage集群下载文件时，此时文件没有完成storage组的同步，会导致客户端无法获取文件,fastdfs-nginx-module会把文件连接到用户上传的storage的服务器

```
tar zxf fastdfs-nginx-module_v1.16.tar.gz
cd fastdfs-nginx-module_v1.16
vim conf/mod_fastdfs.conf
--------------------------
tracker_server=192.168.254.130:22122 #tracker server的ip和port
storage_server_port=23000 #storage server的port
url_have_group_name = true #true:在文件的url上加上组名,eg:${group name}/M00/00/00/xxx
store_path0=/home/fastdfs/storage #storage路径
--------------------------
```

将mod_fastdfs.conf复制到/etc/fdfs目录下

```
cp mod_fastdfs.conf /etc/fdfs
```

#### 5.配置nginx

```
tar zxf nginx-1.8.0.tar.gz
cd nginx-1.8.0
./configure --prefix=/data/nginx --add-module=../../install_file/fastdfs-nginx-module/src
make
make install
```

在./configure时报错

```
tar pcre-7.7.tar.gz
cd pcre-7.7
./configure
make
make install
yum install -y zlib-devel
```

在make时报错

```
ln -sv /usr/include/fastcommon /usr/local/include/fastcommon
ln -sv /usr/include/fastdfs /usr/local/include/fastdfs
```

重新make即可

#### 6.修改nginx配置文件

```
cd nginx-1.8.0
vim conf/nginx.conf
----------------------
server {
    listen       80; #监听80端口
    server_name  192.168.254.131; #访问时fastdfs的ip

    location /group1/M00 { #名为/group1/M00映射的地址
        ngx_fastdfs_module; #fastdfs的地址
    }
    ...
}
----------------------
```

#### 7.启动服务

```
/usr/bin/fdfs_trackerd  /etc/fdfs/tracker.conf #启动tracker server服务
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf #启动storage server服务
/usr/local/nginx/sbin/nginx -c /usr/local/fastdfs/nginx-1.8.0/conf/nginx.conf  #启动nginx
fdfs_upload_file /etc/fdfs/client.conf /usr/local/fastdfs/fastdfs-master.zip #上传fastdfs-master.zip文件
----------------------
返回:group1/M00/00/00/wKj-g1xkJS6AVHaXAAaqd-Q4CGM947.zip
----------------------
启动tracker后检查对应端口是否监听:(启动不成功检查对应的日志 目录/logs)
netstat -anp|grep fdfs

启动storage成功后,可以查看服务是否已经登记到tracker server
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
```

通过ip(192.168.254.131)+ip(8888)+返回的地址 即可访问上传的文件

参考资料：
https://blog.csdn.net/wgp15732622312/article/details/78822218
https://github.com/happyfish100/fastdfs/wiki#tracker
https://www.cnblogs.com/Eivll0m/p/5378328.html (不知道为啥按这个里的方法启动不成功)