# Nginx安装部署

## 虚拟机安装

略

## CentOS系统安装

略

## Nginx的安装

* 常用版本：

  * Nginx开源版

  * Nginx Plus商业版

  * openresty

  * Tengine

* **编译安装**：

  `./configure -prefix=/usr/local/nginx`

  `make`

  ``make install`

* 报错总结：

  ```
  checking for OS 
  + Linux 3.10.0-693.el7.x86_64
  x86_64 checking for C compiler ... not found 
  
  ./configure: error: C compiler cc is not found
  ```

  > 解决方案——
  >
  > 安装gcc
  >
  > `yum install -y gcc`

	```
	./configure: error: the HTTP rewrite module requires the **PCRE** library. You can either disable the module by using --without-http_rewrite_module option, or install the PCRE library into the system, or build the PCRE library statically from the source with nginx by using --with-pcre=<path> option.
	```

	> 解决方案——
	>
	> 根据报错的内容（PCRE）选择下载依赖
	>
	> `yum install -y pcre pcre-devel`

## 启动Nginx

进入安装的目录`/usr/local/nginx/sbin`

> ./nginx 启动
>
> ./nginx -s stop 快速停止
>
> ./nginx -s quick 优雅关闭，在退出前完成已经接受的连接请求
>
> ./nginx -s reload 重新加载配置

## 关于防火墙

* 关闭防火墙

  `systemctl stop firwalld.service`

* 禁止防火墙开机启动

  `systemctl disable firewalld.service`

* 放行端口

  `firewall-cmd --zone=public --add-port=80/tcp --permanent`

* 重启防火墙

  `firewall-cmd --reload`

* 安装程系统服务

  1. 创建服务脚本

     `vi /usr/lib/systemd/system/nginx.service`

  2. 服务脚本内容

     ```
     [Unit] 
     
     Description=nginx - web server 
     
     After=network.target remote-fs.target nss-lookup.target 
     
     [Service] 
     
     Type=forking 
     
     PIDFile=/usr/local/nginx/logs/nginx.pid 
     
     ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf 
     
     ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf 
     
     ExecReload=/usr/local/nginx/sbin/nginx -s reload 
     
     ExecStop=/usr/local/nginx/sbin/nginx -s stop 
     
     ExecQuit=/usr/local/nginx/sbin/nginx -s quit 
     
     PrivateTmp=true 
     
     [Install] 
     
     WantedBy=multi-user.target
     ```

  3. 重新加载系统服务

     `systemctl daemon-reload`

  4. 启动服务

     `systemctl start nginx.service`

* 开机启动

  `systemctl enable nginx.service`

# Nginx基础使用

## 目录结构

进入Nginx的主目录——

>client_body_temp
>
>conf
>
>fastcgi_temp
>
>html
>
>logs
>
>proxy_temp
>
>sbin
>
>scgi_temp
>
>uwsgi_temp

其中后缀为\_temp的文件，主要用来存放运行过程中的临时文件

* conf

  用来存放配置相关的文件

* html

  用来存放静态文件的默认目录html、css等

* sbin

  nginx的主程序

## 基本运行原理

![image-20220424190047049](..\Picture\Nginx\基本运行原理.png)

## Nginx最小配置

* worker_processes

  默认为1

  表示开启一个业务进程

* worker_connections

  默认为1024

  单个业务进程可接受连接数

* include 

  默认为mine.types

  引入http mime类型

* default_type 

  默认为applicaiton/octet-stream

  如果mine类型没匹配上，默认使用二进制流的方式传输

* sendfile

  默认为on

  使用linux的sendfile(socket, file, len)高效网络传输，也就是数据0拷贝

  ![image-20220424190506093](..\Picture\Nginx\未开启sendfile.png)

![image-20220424190615174](..\Picture\JWT\Nginx\开启sendfile.png)

* keepalive_timeout

  默认为65

* server

  虚拟主机配置

  默认为——

  ```
  server {
  
  	listen 80; 监听端口号 
  
  	server_name localhost; 主机名 
  
  	location / { 匹配路径 
  
  		root html; 文件根目录 
  
  		index index.html index.htm; 默认页名称 
  
  	}
  
  	error_page 500 502 503 504 /50x.html; 报错编码对应页面 
  
  	location = /50x.html { 
  
  		root html; 
  
  	} 
  
  } 
  ```

  

