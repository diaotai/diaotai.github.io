---
title: nginx初步探索（配置文件）
date: 2018-11-18 20:06:11
tags: tools
---
> 本文是用于记录 nginx 常见的配置文件处理，包括个人在实验过程中碰到的问题。本次实验成果进行了反向代理，静态资源服务器，加 http 头，修改 http 返回的 header，封 ip 等尝试
> 待完成任务有 gzip 和 日志的开启，这两块始终不成功

# 基本结构
nginx 配置文件分为以下几个部分:
- main 模块:    一般是用于记录 nginx 的用户和用户组、nginx 允许生成的worker数， 日志存储位置等信息
- events 模块:  直接影响网络连接，记录允许每个进程最多接受多少请求，是否可以同时接受多个请求，选取哪一种连接模型，是否进行序列化等。
- http 模块:    很核心的模块，主要用于处理 http 及 https 服务，其中可以定义大量的信息：比如说 反向代理、ip控制、缓存、静态资源服务等等，绝大多数模块的使用都在这里
- server 模块: 用于定义虚拟主机的相关参数，一个 http 模块可以有多个server 参数
- location 模块: 用于定义路径的匹配及处理

# nginx 配置注意点
* root 只代表当前模块的返回值，如果子模块也存在 root，那么会覆盖父模块的 root，而不是跟在后面
* 在静态服务器中，auto_index on;会打开静态目录的选项
* \~ ... 可以使用正则表达式，正则表达式的优先级要高于普通的匹配
* 普通匹配中，会选择最长的匹配

# nginx 基本配置模板
```
########### 每个指令必须有分号结束。#################

#配置用户或者组，默认为nobody nobody。
#user administrator administrators; 
#允许生成的进程数，默认为1
#worker_processes 2; 
#指定nginx进程运行文件存放地址
#pid /nginx/pid/nginx.pid; 
#制定错误日志路径，级别。这个设置可以放入全局块，http块，server块，级别依次为：debug|info|notice|warn|error|crit|alert|emerg
error_log log/error.log debug; 

#工作模式及连接数上限
events {
#设置网路连接序列化，防止惊群现象发生，默认为on
   accept_mutex on; 
#设置一个进程是否同时接受多个网络连接，默认为off
   multi_accept on; 
#事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
#use epoll; 
#单个work进程允许的最大连接数，默认为512
   worker_connections 512; 
}

http {
#文件扩展名与文件类型映射表。设定mime类型(邮件支持类型),类型由mime.types文件定义
#include /usr/local/etc/nginx/conf/mime.types;
   include mime.types; 
#默认文件类型，默认为text/plain
   default_type application/octet-stream; 

# _note 取消服务访问日志(这个很坑爹，我最后也没有找到日志到底打在哪里了))
#access_log off;     
#自定义日志格式
   log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; 
#设置访问日志路径和格式。"log/"该路径为nginx日志的相对路径，mac下是/usr/local/var/log/。combined为日志格式的默认值
   access_log log/access.log myFormat; 
   rewrite_log on;

#允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。（sendfile系统调用不需要将数据拷贝或者映射到应用程序地址空间中去）
   sendfile on; 
#每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
   sendfile_max_chunk 100k; 

#连接超时时间，默认为75s，可以在http，server，location块。
   keepalive_timeout 65; 

# _note gzip压缩开关（同样坑爹，怎么也打不开。。。）
#gzip on;

   tcp_nodelay on;

#设定实际的服务器列表
   upstream mysvr1 {   
     server 127.0.0.1:7878;
     server 192.168.10.121:3333 backup; #热备(其它所有的非backup机器down或者忙的时候，请求backup机器))
   }
   upstream mysvr2 {
# _note weigth参数表示权值，权值越高被分配到的几率越大（传说中的加权轮询）
     server 192.168.1.11:80 weight=5;
     server 192.168.1.12:80 weight=1;
     server 192.168.1.13:80 weight=6;
   }
   upstream https-svr {
# _note 每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题（不过说起来多服务器的 session 这种东西不应该是使用 redis 或 memory吗）
     ip_hash;
     server 192.168.1.11:90;
     server 192.168.1.12:90;
   }

#error_page 404 https://www.baidu.com; #错误页

#HTTP服务器

# 静态资源一般放在nginx所在主机
   server {
       listen 80; #监听HTTP端口
       server_name 127.0.0.1; #监听地址  
       keepalive_requests 120; #单连接请求上限次数
       set $doc_root_dir "/Users/doing/IdeaProjects/edu-front-2.0"; #设置server里全局变量
       #index index.html;  #定义首页索引文件的名称
       location ~*^.+$ { #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
        # _note 静态资源目录实验成功（除了我怎么都关不掉这个功能），本功能不能提供目录的展示，不过可以提供旗下所有文件的下载
          root $doc_root_dir; #静态资源根目录  
          proxy_pass http://mysvr1; #请求转向“mysvr1”定义的服务器列表
          # _note 本处是封ip功能，试验成功，写 all 的时候会把所有ip都干掉
          #deny 127.0.0.1; #拒绝的ip
          #allow 172.18.5.54; #允许的ip           
       } 
   }

#http
   server {
       listen 80;
       server_name www.helloworld.com; # _note 监听基于域名的虚拟主机。可有多个，可以使用正则表达式和通配符，不过要玩这个需要动host文件，不然依旧没什么卵用。就方便而言用 OmegaSwitch 是更好的选择
       charset utf-8; #编码格式
       set $static_root_dir "/Users/doing/static";
       location /app1 { # _note 反向代理的路径（和upstream绑定），location后面设置映射的路径(注意，本处对应的路径在 server 上要有对应的路径，这里的映射不是把该路径映射到 server 的根路径上) 
           proxy_pass http://zp_server1;
       } 
       location /app2 {  
           proxy_pass http://zp_server2;
       } 
       location ~ ^/(images|javascript|js|css|flash|media|static)/ {  #静态文件，nginx自己处理
           root $static_root_dir;
           expires 30d; #静态资源过时间30天
       }
       location ~ /\.ht {  #禁止访问 .htxxx 文件
           deny all;
       }
       location = /do_not_delete.html { # _note 直接简单粗暴的返回状态码及内容文本（如果使用浏览器访问该接口会直接下载一个文本文件）
           return 200 "hello.";
       }

       # 指定某些路径使用https访问(使用正则表达式匹配路径+重写uri路径)
       location ~* /http* { #路径匹配规则：如localhost/http、localhost/httpsss等等
#rewrite只能对域名后边的除去传递的参数外的字符串起作用，例如www.c.com/proxy/html/api/msg?method=1&para=2只能对/proxy/html/api/msg重写。
#rewrite 规则 定向路径 重写类型;
#rewrite后面的参数是一个简单的正则。$1代表正则中的第一个()。
#$host是nginx内置全局变量，代表请求的主机名
#重写规则permanent表示返回301永久重定向
           rewrite ^/(.*)$ https://$host/$1 permanent;
       }

       #错误处理页面（可选择性配置）
#error_page 404 /404.html;
#error_page 500 502 503 504 /50x.html;

#以下是一些反向代理的配置(可选择性配置)
#proxy_redirect off;
#proxy_set_header Host $host; #proxy_set_header用于设置发送到后端服务器的request的请求头
#proxy_set_header X-Real-IP $remote_addr;
#proxy_set_header X-Forwarded-For $remote_addr; #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
#proxy_connect_timeout 90; #nginx跟后端服务器连接超时时间(代理连接超时)
#proxy_send_timeout 90; #后端服务器数据回传时间(代理发送超时)
#proxy_read_timeout 90; #连接成功后，后端服务器响应时间(代理接收超时)
#proxy_buffer_size 4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
#proxy_buffers 4 32k; #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
#proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
#proxy_temp_file_write_size 64k; #设定缓存文件夹大小，大于这个值，将从upstream服务器传

#client_max_body_size 10m; #允许客户端请求的最大单文件字节数
#client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数


   }

#https
#(1)HTTPS的固定端口号是443，不同于HTTP的80端口；
#(2)SSL标准需要引入安全证书，所以在 nginx.conf 中你需要指定证书和它对应的 key
   server {
     listen 443;
     server_name  www.hellohttps1.com www.hellohttps2.com;
     set $geek_web_root "/Users/doing/IdeaProjects/backend-geek-web";
     ssl_certificate      /usr/local/etc/nginx/ssl-key/ssl.crt; #ssl证书文件位置(常见证书文件格式为：crt/pem)
     ssl_certificate_key  /usr/local/etc/nginx/ssl-key/ssl.key; #ssl证书key位置
     location /passport {
       send_timeout 90;
       proxy_connect_timeout 50;
       proxy_send_timeout 90;
       proxy_read_timeout 90;
       proxy_pass http://https-svr;
     }
     location ~ ^/(res|lib)/ {
        root $geek_web_root; 
        expires 7d;
# _note add_header用于为后端服务器返回的response添加请求头，这里通过add_header实现CROS跨域请求服务器，该指令疑似只有用于具体的 location块中才有作用
        add_header Access-Control-Allow-Origin *; 
     }
#ssl配置参数（选择性配置）
     ssl_session_cache shared:SSL:1m;
     ssl_session_timeout 5m;
   }

   #配置访问控制：每个IP一秒钟只处理一个请求，超出的请求会被delayed
#语法：limit_req_zone  $session_variable  zone=name:size  rate=rate (为session会话状态分配一个大小为size的内存存储区，限制了每秒（分、小时）只接受rate个IP的频率)
   limit_req_zone  $binary_remote_addr zone=req_one:10m   rate=1r/s nodelay;
   location /pay {
       # 用于向请求添加http头，这里是在请求向具体的后端服务器之前加请求头，可以从后端读到（似乎一样只能用于具体的 location 块中）
        proxy_set_header Host $http_host;
        proxy_set_header X-Real_IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
#访问控制：limit_req zone=name [burst=number] [nodelay];
        limit_req zone=req_one burst=5; #burst=5表示超出的请求(被delayed)如果超过5个，那些请求会被终止（默认返回503）
        proxy_pass http://mysvr1;
   }

#可以把子配置文件放到/usr/local/etc/nginx/servers/路径下，通过include引入
   include /usr/local/etc/nginx/servers/*.conf;

} 

```

# 内置全局变量列表
```
$args ：这个变量等于请求行中的参数，同$query_string
$content_length ： 请求头中的Content-length字段。
$content_type ： 请求头中的Content-Type字段。
$document_root ： 当前请求在root指令中指定的值。
$host ： 请求主机头字段，否则为服务器名称。
$http_user_agent ： 客户端agent信息
$http_cookie ： 客户端cookie信息
$limit_rate ： 这个变量可以限制连接速率。
$request_method ： 客户端请求的动作，通常为GET或POST。
$remote_addr ： 客户端的IP地址。
$remote_port ： 客户端的端口。
$remote_user ： 已经经过Auth Basic Module验证的用户名。
$request_filename ： 当前请求的文件路径，由root或alias指令与URI请求生成。
$scheme ： HTTP方法（如http，https）。
$server_protocol ： 请求使用的协议，通常是HTTP/1.0或HTTP/1.1。
$server_addr ： 服务器地址，在完成一次系统调用后可以确定这个值。
$server_name ： 服务器名称。
$server_port ： 请求到达服务器的端口号。
$request_uri ： 包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。
$uri ： 不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。
$document_uri ： 与$uri相同。
```


# 参考资料:
* [nginx快速入门之配置篇](https://zhuanlan.zhihu.com/p/31202053)
* [nginx beginner's guide](http://nginx.org/en/docs/beginners_guide.html)
* [nginx cookbook](印象笔记)

