---
title: 前端需要知道的nginx配置
date: 2019-05-03 15:51:27
tags:  nginx
---

 作为一个前端，或多或少都听过nginx。通俗的来说，nginx是一个高性能的HTTP和反向代理web服务器，经常用于服务端的反向代理和负载均衡。这篇文章主要介绍前端经常用到的nginx配置，诸如：
- nginx.conf文件
- nginx的启动，停止，重启
- 自定义错误页面和访问设置
- 访问权限配置
- 适配pc或移动设备
- gzip配置
- 虚拟主机配置
- 反向代理配置
- 负载均衡
- https配置

 <!--more-->
 ### nginx.conf 文件

 nginx.conf 文件是nginx的配置文件，我们修改配置，往往都是在这个文件下进行修改的。下面是对部分配置字段的讲解


 ```
# 运行用户
#user  nobody;

# nginx进程，一般设置和cpu核数一样
worker_processes  1;

# 错误日志存放目录
#error_log  logs/error.log;

# 进程pid存放位置
#pid        logs/nginx.pid;

# events模块 配置主要影响nginx服务器和用户的网络连接
events {
    worker_connections  1024; # 单个后台进程的最大并发数
}


http {
    include       mime.types; # 文件扩展名和类型映射表
    default_type  application/octet-stream;  # 默认文件类型
    
    # 设置日志模式
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    # nginx访问日志存放位置
    #access_log  logs/access.log  main;
    
    # 开启高效传输模式
    sendfile        on;


    # 减少网络报文段的数量
    #tcp_nopush     on; 

    # 保持连接的时间，也叫超时时间
    #keepalive_timeout  0;
    keepalive_timeout  65;

    # 开启gip压缩
    #gzip  on;

    server {
        listen       80; # 监听端口
        server_name  localhost; # 配置域名

        #charset koi8-r;
        #access_log  logs/host.access.log  main;

        location / {
            root   html;   # 服务器默认启动目录
            index  index.html index.htm; # 默认访问文件
        }

        error_page  404              /404.html; # 配置404页面

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html; # 错误状态码显示页面
        location = /50x.html {
            root   html;
        }
        ....
    }
}
 ```



 ### nginx的启动，停止，重启


**启动**

- 使用命令 `nginx` 直接启动
- 使用systemctl命令

```
systemctl start nginx.service
```

**停止**

```
//  从容停止
nginx -s quit

// 立即停止服务
nginx -s stop

// 杀死nginx进程
killall nginx


// systemctl停止
systemctl stop nginx.service
```

**重启nginx服务**

```
systemctl restart nginx.service
```

**重新载入配置文件**

```
nginx -s reload
```

### 自定义错误页面和访问设置

修改如下配置，地址也可以使用外部的链接地址。

```
error_page   500 502 503 504  /50x.html;
```
然后添加新建的自定义错误页面到网站根目录下，并重启服务后进行访问。


### 访问权限配置

```
location / {
    deny 1.1.1.1;  // 禁止该ip访问
    allow 1.1.1.2; // 允许该ip访问
}

location =/img {
    allow all; # 对网站下的img所有用户允许访问
}

```

我们也可以使用正则表达式设置访问权限。
- = ： 表示精确匹配后面的url
- ~ :  表示 正则匹配，但区分大小写
- ~*： 正则匹配，不区分大小写
- ^~: 表示普通字符匹配，如果该选项匹配，只匹配该选项，不匹配别的选项
- / : 通用匹配，如果没有其他匹配，任何请求都会匹配到



### 适配pc或移动设备

原理：通过内置变量 `$http_user_agent`, 可以获取客户端的useragent,就可以判断用户当前处于移动端还是pc端，进而显示对应的页面。

```
server {
    listen 80;
    server selinayu.cc;
    location / {
        root /usr/share/nginx/pc; //pc的本地路径
        if($http_user_agent ~* '(Android|webos|iphone|ipad|blackberry)') {
            // ~* 根据正则表达式不区分大小写
            root / usr/share/nginx/mobile
        }
        index index.html
    }

}
```

### gzip配置

gzip 是一种网页压缩技术，可使用gzip检测工具对比压缩前后网站的变化。

```
http {
    gzip on;
}
```

### 虚拟主机配置

**基于端口号配置虚拟主机**

原理就是nginx 监听多个端口，根据不同的端口号，来区分不同的网站。
```
server{
        listen 8080;
        server_name localhost;
        root   html;
        index  index.html index.htm;
}
server{
        listen 8001;
        server_name localhost;
        root   html;
        index  index.html index.htm;
}
```

**基于ip设置虚拟主机**

和基于端口配置的类似，只是server_name 选项，需要配置改成ip


**基于域名设置虚拟主机**

基于域名配置，需要对域名进行解析到对应的ip上。至于nginx的配置修改也是修改server_name，配置对应的域名

```
server {
    listen 80
    server_name nginx.selinayu.cc
}
server {
    listen 80
    server_name nginx2.selinayu.cc
    location / {

    }
}
```

 ### 反向代理配置


**正向代理**

简单来说：你想访问目标服务器，但是没有权限，代理服务器有权限访问目标服务器，并你有访问代理服务器的权限。这时，你可以访问代理服务器，代理服务器再访问真实服务器,进而返回内容给客户端。
正向代理 代理的是客户端。我们常用的翻墙工具就是正向代理。

**反向代理**

反向代理 代理的是服务器。现在基本所有的大型网站的页面都用了反向代理。客户端发送的请求，想要访问server服务器上的内容，发送的请求被发送到代理服务器上，代理服务器在把请求发送到自己设置好的内部服务器上。这时，代理服务器代理的是服务器，对客户端提供一个统一的代理入口。

好处：
- 安全性：使用反向代理客户端用户只能通过外来网来访问服务器，用户并不知道真实访问的服务器是哪一台
- 功能性： 反向代理主要用途是为多个服务器提供负载均衡，缓存等功能。

```
server {
    listen 80;
    server_name wwww.baidu.com
    location / {
        proxy_pass ip; # 服务器ip
    }
}
```
上面的配置，就是访问www.baidu.com后，会被反向代理到对应的ip。

### 负载均衡

负载均衡，首先需要两台或以上可以提供相同服务的web服务器。

```
# http下添加如下代码
upstream  backserver { # backserver 自定义名称
    server ip1 + 端口1; # 提供相同服务的服务器
    server ip2 + 端口2;
}

# server 下添加代码
location / {
    proxy_pass http://backserver;
}
```

重启nginx，就可以实现负载均衡了。负载均衡有四种模式：

**1.默认轮询**

默认情况，上面的配置就是默认轮询方式，请求会随机派发到你配置的服务器上。

**2.权重分配**

weight 的值越高被派发请求的概率也就越高，可以根据服务器配置的不同来设置。
```
upstream backserver {
    server ip1+端口1 weight=1;
    server ip2+端口2 weight=2;
}
```

**3.哈希分配**

根据客户端ip来分配服务器，比如第一次访问请求被派发给了ip1服务器，那之后的请求都会发送到这台服务器上，可以解决session共享的问题。
```
upstream backserver {
    ip_hash;
    server ip1 + 端口1; 
    server ip2 + 端口2;
}
```
**4.最少连接分配**

根据添加的服务器判断哪台服务器分的连接最少就把请求给谁。
```
upstream backserver {
    least_conn;
    server ip1 + 端口1; 
    server ip2 + 端口2;
}
```

### https配置
需要ssl证书

```
    server {
        listen       443 ssl;
        server_name  selinayu.cc;

        keepalive_timeout 100; # 长连接 100s

        ssl_certificate      /etc/nginx/ssl_key/lcy.crt;  # 证书路径
        ssl_certificate_key  /etc/nginx/ssl_key/lcy.key; # 私钥文件

        ssl_session_cache    shared:SSL:1m; # 设置1M的缓存
        ssl_session_timeout  5m; # 过期时间 5 分钟


        location / {
            root   html;
            index  index.html index.htm;
        }
    }

    server {
        listen 80;
        server_name selinayu.cc;

        # 当访问http时会重定向到https
        return 301 https://$server_name$request_uri;
    }
```

