## nginx安装

nginx安装有两种方式：1.安装包 2.yum

### 安装包方式

#### 1.安装所需插件

**gcc**

```
# gcc是linux下的编译器
# 查看是否安装，版本
gcc -v

# 安装gcc
yum -y install gcc
```

**pcre、pcre-devel**

```
# pcre是一个perl库，包括perl兼容的正则表达式库，nginx的http模块使用pcre来解析正则表达式，所以需要安装pcre库
yum install -y pcre pcre-devel
```

**zlib**

```
# zlib库提供了很多种压缩和解压缩方式nginx使用zlib对http包的内容进行gzip
yum install -y zlib zlib-devel
```

**openssl**

```
# openssl是web安全通信的基石
yum install -y openssl openssl-devel
```

#### 2.安装nginx

```
# 1.下载nginx安装包，或者自己手动上传
wget http://nginx.org/download/nginx-1.9.9.tar.gz

# 解压
tar -zxvf nginx-1.9.9.tar.gz

# 安装
mkdir /usr/local/nginx
./configure --prefix=/usr/local/nginx
make && make install

# pwd 查看当前目录

# 修改nginx配置
cd /usr/local/nginx
vi conf/nginx.conf

# 启动服务
cd ../sbin
./nginx

# 重启服务
./nginx -s reload

# 浏览器访问
http://ip:port

# 如果访问不到，可能是因为防火墙
service firewalld status
service firewalld stop # 重启后，防火墙还是会是打开状态
# 禁止开机启动
chkconfig firewalld off
```

### 设置开机自启

```
# 创建nginx.service文件
cd /lib/systemd/system
vi nginx.service

# 文件内容
[Unit]
Description=nginx 
After=network.target 
  
[Service] 
Type=forking 
ExecStart=/usr/local/nginx/sbin/nginx # 填写自己的nginx安装目录
ExecReload=/usr/local/nginx/sbin/nginx reload
ExecStop=/usr/local/nginx/sbin/nginx quit
PrivateTmp=true 
  
[Install] 
WantedBy=multi-user.target

# 保存文件，执行
systemctl enable nginx.service
```

