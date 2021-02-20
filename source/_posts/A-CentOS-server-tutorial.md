---
title: A-CentOS-server-tutorial
date: 2021-02-20 18:44:54
tags: 前端日常
cover: https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/img/20210220184714.png
---

## 从零搭建个人服务器
> 趁着腾讯云十周年新用户打折薅了一波老马的羊毛，三年单核2g 1M带宽CVM ￥288带回家，买不了吃亏买不了上当（阿里云新用户折扣已经用过了( •̀ ω •́ )✧,羊毛得换着薅,毕竟也只是个人学习使用，要求不需要太高）。

### 配置CVM
>购买了云服务器之后要先选择需要安装的操作系统镜像，这里按照推荐选择的64位CentOS 7.6版本。服务器初始化成功之后可以在腾讯云控制台看到自己的服务器信息（实例id，内外网ip等，红框内可以进行系统镜像备份、重装系统、登录等操作）

![](https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/image20200918112719.png)

登录时最好采用秘钥登录的方式，如果设置密码登录，每次登录服务器会有如下警告

![](https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/image20200918114003.png)

可以进行如下配置关闭密码登录入口，修改文件后记得保存并重启SSHD服务
![](https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/image20200918114650.png)
```
sudo systemctl restart sshd
```
秘钥建议妥善保管，以后登录的时候就只不需要输入复杂的密码了，直接上传秘钥即可

### 安装nginx

#### 1.gcc 安装
安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：
```
yum install gcc-c++
```

#### 2.PCRE pcre-devel 安装
PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：
```
yum install -y pcre pcre-devel
```

#### 3.zlib 安装
zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。

#### 4.OpenSSL 安装
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。
```
yum install -y openssl openssl-devel
```

#### 5.下载nginx并安装
使用wget命令下载（推荐）。确保系统已经安装了wget，如果没有安装，执行
```
yum install wget
wget -c https://nginx.org/download/nginx-1.18.0.tar.gz
```
nginx版本可自行选择，一般安装最新的稳定版就可以。下载完成后进行解压，建议把下载下来的压缩包剪切到/usr/local目录下进行安装(用户级的安装程序都建议安装到此目录下)

```
# 将压缩包移动至usr目录下进行解压安装
mv ./nginx-1.18.0.tar.gz /usr/local
# 进入压缩包所在目录
cd /usr/local
# 解压
tar zxvf nginx-1.18.0.tar.gz
# 进入nginx目录
cd nginx-1.18.0
# 安装三连击
./configure && make && make install 
```
> 在linux中，‘&’ 表示任务在后台执行，如要在后台运行redis-server,则执行redis-server &，
‘&&’ 表示前一条命令执行成功时，才执行后一条命令 ，如 echo '1‘ && echo '2'。
‘|’表示管道，上一条命令的输出，作为下一条命令参数，‘||’表示上一条命令执行失败后，才执行下一条命令

##### linux的复制/剪切命令用法
> 复制/拷贝：
-  cp  文件名  路径
```
cp  hello.csv  ./python/ml  //把当前目录的hello.csv拷贝到当前目的python文件夹里的ml文件夹里
```
- cp 源文件名  新文件名
```
cp  hello.txt   world.txt   //复制并改名,并存放在当前目录下  
```
- cp file1 file2 复制一个文件 
```
cp -a /tmp/dir1 . //复制一个目录到当前工作目录 
cp -a dir1 dir2 //复制一个目录
cp dir/* . //复制一个目录下的所有文件到当前工作目录
```
- 剪切/移动：mv 文件名 路径
```
mv hello.csv ./python   //把当前目录的hello.csv剪切到当前目的python文件夹里
mv  hello.txt  ../java/   //把当前目录下的文件hello.txt剪切到上一级目录的子目录java目录里
mv  hello.txt  ..     //把文件hello.txt移动到上一级目录
```

#### 6.nginx启动与停止
先进入安装目录的sbin文件夹下，
```
cd /usr/local/nginx/sbin
```
启动nginx
```
./nginx
```
nginx常用命令
```
./nginx   # 启动命令
./nginx -s stop   # 强制停止命令
./nginx -s quit   # 任务处理完再停止
./nginx -s reload # 重启
```
浏览器地址栏输入CVM的ip，如果出现nginx的欢迎页面，则表示安装成功

![](https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/img/20210219174135.png)

上面的命令需要切换到sbin目录下才能生效，会显得比较麻烦，可以考录将nginx添加到系统服务。
进入到/etc/init.d/目录
目录下创建nginx脚本，内容为：
```
#!/bin/bash
#
# chkconfig: - 85 15
# description: Nginx is a World Wide Web server.
# processname: nginx

nginx=/usr/local/nginx/sbin/nginx
conf=/usr/local/nginx/conf/nginx.conf
case $1 in
start)
echo -n "Starting Nginx"
$nginx -c $conf
echo " done"
;;
stop)
echo -n "Stopping Nginx"
stop)
echo -n "Stopping Nginx"
$nginx -s stop || echo -n "nginx not running"
echo " done"
;;
test)
$nginx -t -c $conf
;;
reload)
echo -n "Reloading Nginx"
ps auxww | grep nginx | grep master | awk '{print $2}' | xargs kill -HUP
echo " done"
;;
restart)
$0 stop
$0 start
;;
show)
ps -aux|grep nginx
;;
*)
echo -n "Usage: $0 {start|restart|reload|stop|test|show}"
;;
esac
```
> 注意，如果是本地新建的文件，保存编辑的时候一定要保存为UNIX格式

文件创建完成后，使用**chmod +x /etc/init.d/nginx**命令来设置执行权限，然后使用**chkconfig --add nginx**将其注册成服务，最后使用**chkconfig nginx on**命令设置nginx为开机启动。这样就可以使用下面的全局命令来控制nginx的启停了。

```
service nginx start
service nginx stop 
service nginx restart 
service nginx reload
```

#### 7.配置nginx
首先配置https，通过腾讯云的<u>[SSL证书管理控制台](https://console.cloud.tencent.com/ssl)</u>申请SSL证书，或者通过<u>[certbot](https://certbot.eff.org/)</u>来注册SSL证书。后者是傻瓜式的，按照官方文档来操作就好。这里介绍下前者的操作方法。
![](https://cdn.jsdelivr.net/gh/daxiahu/ImageBed@master/img/20210220163249.png)
在控制台中点击下载将证书文件下载到本地并解压，根据自己部署的服务器类型选择对应的证书，这里我用的nginx，所以选择nginx证书。
nginx文件夹里面有两个文件：
- **1_husang.top_bundle.crt**   证书文件
- **2_husang.top.key**  私钥文件

将已获取到的 **1_husang.top_bundle.crt** 证书文件和 **2_husang.top.key** 私钥文件从本地目录拷贝到轻量应用服务器 Nginx 默认配置文件目录中（nginx主安装目录下的 ./conf/ 即为 Nginx 默认配置文件目录）。
nginx的默认配置文件为nginx.conf

> 配置https模块
```
# HTTPS server
    
    server {
        listen       443 ssl;
        server_name  husang.top;  #填写你的证书绑定的域名

        ssl_certificate      1_husang.top_bundle.crt;   #填写你的证书文件名称
        ssl_certificate_key  2_husang.top.key;  #填写您的私钥文件名称

        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;     # 可参考此 SSL 协议进行配置

        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;    #可按照此加密套件配置，写法遵循 openssl 标准
        ssl_prefer_server_ciphers  on;

        location / {
            root   html;
            index  index.html index.htm;
        }

        location /game {
            alias   /home/www/daxigua/;
            index  index.html index.htm;
        }
    }
```

> 将http请求重定向到https
```
 server {
        listen       80;
        server_name  husang.top;
        location / {
            root   html;
            index  index.html index.htm;
        }
        return 301 https://$host$request_uri; 
    }
```
配置更改完成后，重启nginx才能生效，输入命令：
```
service nginx reload
```
此时发现输入http的链接后会自动跳转至https的安全链接。

> 至此，一个简单的个人web服务器已经搭建完毕了，将打包后的静态文件上传就可以正常访问啦，当然也可以配合CI/CD工具实现自动构建部署。如果是企业级的，还需要考虑负载均衡、缓存等等，这就涉及得比较多了，有兴趣的同鞋可以自己google，感谢大家的阅读(～￣▽￣)～