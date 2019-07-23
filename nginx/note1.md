##  1.关闭iptables中的规则
+ iptables -t nat -F

##  2.停用selinux
+ 查看是否停用：getenforce
+ 设置停用：setenforce 0

## 3.安装gcc
+ yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake

## 4.
+ yum -y install wegt httpd-tools vim


## 5.ngnix概述
Ngnix是一个开源且高性能、可靠的HTTP中间件、代理服务。

## 6.nginx优点
### 6.1 IO多路复用epoll
多个描述符的I/O操作都能在一个线程内并发交替地顺序完成，这就叫I/O多路复用，这里的“复用”指的是复用同一个线程。

### 6.2 轻量级
+ 功能模块少
+ 代码模块化

### 6.3CPU亲和（affinity）
是一种把CPU核心和Nginx工作进程绑定方式，把每个worker进程固定在一个CPU上执行，减少切换cpu的cache miss，获得更好的性能。

### 6.4 sendfile

## 7. Centos安装nginx
### 7.1在/etc/yum.repos.d下创建ngnix.repo文件
+ vim /etc/yum.repos.d/ngnix.repo
 ``` 
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
```
这里$releasever为centos版本（7）

### 7.2 安装nginx
+ yum install nginx

> 查看版本：nging -v

## 8.展示nginx在linux安装了哪些包？
+ rpm -ql nginx

## 9.ngnix安装目录讲解
### 9.1
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /etc/logrotate.d/nginx | 配置文件 | Nginx日志轮转，用于logrotate服务的日志切割 | 

### 9.2
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /etc/nginx、/etc/nginx/conf.d、  /etc/nginx/conf.d/default.conf、 /etc/nginx/nginx.conf| 目录，配置文件 | Nginx主配置文件 |

### 9.3
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /etc/nginx/fastcgi_params、/etc/nginx/scgi_params、/etc/nginx/uwsgi_params | 配置文件 | cgi配置相关，fastcgi配置 | 

### 9.4
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /etc/nginx/koi-utf、/etc/nginx/koi-win、/etc/nginx/win-utf | 配置文件 | 编码转换映射转化文件 | 

### 9.5
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /etc/nginx/mime.types | 配置文件 | 设置http协议的Content-Type与扩展名对应关系 |

### 9.6
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /usr/lib/systemd/system/nginx-debug.service、/usr/lib/systemd/system/nginx.service 、/etc/sysconfig/nginx 、/etc/sysconfig/nginx-debug |  配置文件 | 用于配置出系统守护进程管理器管理方式 |

### 9.7 
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /usr/lib64/nginx/modules、/etc/nginx/modules | 目录 | Nginx模块目录 | 

### 9.8
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /usr/sbin/nginx、/usr/sbin/nginx-debug | 命令 | Nginx服务的启动管理的终端命令 |


### 9.9
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /usr/share/doc/nginx-1.16.0、/usr/share/doc/nginx-1.16.0/COPYRIGHT、/usr/share/man/man8/nginx.8.gz | 文件、目录 | Nginx的手册和帮助文件 |

### 9.10
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /var/cache/nginx | 目录 | Nginx的缓存目录 |

### 9.11
| 路径 | 类型 | 作用 |
| :-: | :-: | :-: |
| /var/log/nginx | 目录 | Nginx的日志目录 |

## 10 Nginx默认配置语法


## 11 Nginx启动命令
+ /usr/sbin/nginx -c /etc/nginx/nginx.conf
/usr/sbin/nginx为启动管理的终端命令，/etc/nginx/nginx.conf为主配置文件。

##  12 Nginx重启命令
+ systemctl restart nginx.service
+ systemctl reload nginx.service


## 13 Nginx变量
+ HTTP请求变量：arg_PARAMETER，http_HEADER，sent_http_HEADER

## 14 Nginx模块
### 14.1 _stub_status_module
| 编译选项 | 作用 |
| :-: | :-: |
| --with-http_stub_status_module | Nginx的客户端状态 |

>  Systax:stub_status;
    Default:—;
    Context:server,location

+ 进入default.conf文件
> vim default.conf
+ 添加location
>     location /mystatus {
>        stub_status;
>     }
+  测试配置是否成功
> nginx -tc /etc/nginx/nginx.conf

> 如果出现以下类似内容，则表示成功：
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

+ 重启ngnix.conf
> nginx -s reload -c /etc/nginx/nginx.conf

+ 访问http://IP地址/mystatus
> Active connections: 2 
server accepts handled requests
 8 8 28 
Reading: 0 Writing: 1 Waiting: 1

8：表示接收握手的总次数
8：表示所处理的连接数
28：表示总的请求数
握手和处理的连接数相等，表示没有丢失。

### 14.2
| 编译选项 | 作用 |
| :-: | :-: |
| --with-http_random_index_module | 目录中选择一个随机主页 | 

> Systax：random_index on|off;
    Default：random_index off;
    Context:location

+ 将default.conf里面的location替换为：
```
  location / {
         root /usr/share/nginx/html;
         random_index on;
         #index index.html index.htm
}
```

### 14.3 --with-http_sub_module
| 编译选项 | 作用 |
| :-: | :-: |
| --with-http_sub_module | HTTP内容替换 | 

> Systax:sub_filter string replacement;
    Default:—
    Context:http,server,location

> Systax:sub_filter_last_modified on|off;
    Default:sub_filter_last_modified off;
    Context:http,server,location

>  是否只匹配html中的第一个匹配的：
    Systax:sub_filter_once on|off;
    Default:sub_filter_once on;
    Context:http,server,location


## 15 Nginx的请求限制
+ 连接频率限制：-limit_conn_module
+ 请求频率限制：-limit_req_module

### 15.1连接限制
> Syntax：limit_conn_zone key zone=name:size
   Default:—
   Context:http

> Syntax：limit_conn zone number;
   Default：—
   Context：http,server,location

### 15.2请求限制
> Syntax：limit_req_zone key zone=name:size rate=rate;
   Default:—
   Context:http

> Syntax：limit_req zone=name [burst=number] [nodelay];
   Default:—
   Context:http,server,location


## 16访问控制
### 16.1基于认证的访问控制
> Syntax：auth_basic string | off;
   Default：auth_basic off;
   Context：http,server,location,limit_except

> Syntax：auth_basic_user_file file;
   Default：——
   Context：http,server,location,limit_except

+ 下载htpasswd
> rpm -qf /usr/bin/htpasswd

> yum install httpd-tools 

+ 添加账号文件
> [root@VM_0_16_centos nginx]# htpasswd -c ./auth_conf 用户名


+ 修改default.conf
    location ~ ^/yellow.html {
        root /usr/share/nginx/html;
        auth_basic "Auth access test!input your password!";
        auth_basic_user_file /etc/nginx/auth_conf;      
        index index.html index.hml;
    }


## 16.1 
> deny
> allow
