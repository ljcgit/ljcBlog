## 1 文件读取
> Syntax:sendfile on | off;
   Default:sendfile off;
   Context:http,server,location.if in location

### 1.1
> Syntax:tcp_nopush on | off;
   Default:tcp_nopush off;
   Context:http,server,location

作用：sendfile开启的情况下，提高网络包的传输效率。

### 1.2
> Syntax:tcp_nodelay on | off;
   Default:tcp_nodelay off;
   Context:http,server,location

作用：keepalive连接下，提高网络包的传输实时性。

## 2 文件压缩
> Syntax:gzip on | off;
   Default:gzip off;
   Context:http,server,location,if in location;

作用：压缩传输

### 2.1 压缩级别
> Syntax:gzip_comp_level level;
   Default:gzip_comp_level 1;
   Context:http,server,location

### 2.2 控制gzip版本
> Syntax:gzip_http_version 1.0|1.1;
   Default:gzip_http_version 1.1;
   Context:http,server,location

## 3 扩展Nginx压缩模块
+ http_gzip_static_module : 预读gzip功能
+ http_gunzip_module : 应用支持gunzip的压缩方式

## 4 实例
+ 在/etc/nginx/conf.d/下创建一个static_server.conf文件
+ 修改内容如下：
```conf
server {
    listen       81;
    server_name  localhost;

    sendfile on;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location ~ .*\.(jpg|gif|png)$ {
        #gzip on;
        #gzip_http_version 1.1;
        #gzip_comp_level 2;
        #gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root /opt/app/code/images;
    }

    location ~ .*\.(txt|xml)$ {
        #gzip on;
        #gzip_http_version 1.1;
        #gzip_comp_level 1;
        #gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root /opt/app/code/doc;
    }

    location ~ ^/download {
        #gzip_static on;
        tcp_nopush on;
        root /opt/app/code;
   }

}
```
+ 在/opt/app/code/images下放置一张图片
+ 通过“IP:81/图片名称”的方式访问图片
![测试结果](https://upload-images.jianshu.io/upload_images/16503287-202831ab81525821.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 修改static_server.conf
```
    location ~ .*\.(jpg|gif|png)$ {
        gzip on;
        gzip_http_version 1.1;
        gzip_comp_level 2;
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpg image/jpeg image/gif image/png;
        root /opt/app/code/images;
    }

```
![测试结果](https://upload-images.jianshu.io/upload_images/16503287-bb6e352a4c0c72f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 发现图片大小只少了18KB（gzip主要是对文本进行压缩）。

+ 测试文本的压缩变化，在/opt/app/code/doc防止一个txt文件，并通过网址进行访问
![测试压缩](https://upload-images.jianshu.io/upload_images/16503287-d3950303058debb3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 修改static_server.conf
```
    location ~ .*\.(txt|xml)$ {
        gzip on;
        gzip_http_version 1.1;
        gzip_comp_level 1;
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root /opt/app/code/doc;
    }
```
![QQ图片20190629161523.png](https://upload-images.jianshu.io/upload_images/16503287-39e04f5cf8e2d2f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 从237KB压缩到了1.7KB。

## 5 Nginx缓存
> Syntax:expires [modified] time;
               expires epoch | max | off;
   Default: expires off;
   Context:http,server,location,if in location

+ 示例：
```conf
    location ~ .*\.(html|htm)$ {
        expires 24h;
        root /opt/app/code;
    }
```

## 6 Nginx跨站访问
> Syntax:add_header name value [always];
   Default:—
   Context:http,server,location,if in location
+ Access-Control-Allow-Origin
+ Access-Control-Allow-Methods 


## 7 防盗链
> Syntax:valid_referers none | blocked | server_names | string ...;
   Default:—
   Context:server,location

> none:允许没有http_refer的请求访问资源
   block：允许不是http://开头的，不带协议的请求访问资源

+ 示例
```
        valid_referers none blocked 203.195.133.72;
        if ( $invalid_referer ) {
            return 403; 
        }
```


## 8 代理
> Syntax:proxy_pass URL
   Default:—
   Context:location,if in location,limit_except

+ 示例：
```
  proxy_pass http://localhost:8888;
```

## 9 负载均衡
### 9.1 示例
+ 首先监听三个端口
```
server {
    listen       91;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /opt/app/code;
        index  index.html index.htm;
    
```

+ 创建负载均衡配置文件：
```
    upstream ljc{
        server 203.195.133.72:91;
        server 203.195.133.72:92;
        server 203.195.133.72:93;
    }
server {
    listen       84;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        proxy_pass http://ljc;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
```

+ 重启服务
+ 访问IP:84，发现会以轮询的方式切换页面。

### 9.2 upstream参数
+ down:当前的server暂时不参与负载均衡
+ backup:预留的备份服务器
+ max_fails:允许请求失败的次数
+ fail_timeout:经过max_fails失败后，服务暂停的时间
+ max_conns:限制最大的接收的连接数

### 9.3调度算法
+ 轮询：按时间顺序逐一分配到不同的后端服务器
+ 加权轮询：weight值越大，分配到的访问几率越高
+ ip_hash：每个请求按访问IP的hash结果分配，这样来自同一个IP的固定访问一个后端服务器
+ url_hash:按照访问的URL的hash结果来分配请求，使每一个URL定向到同一个后端服务器
+ least_conn:最少链接数，哪个机器连接数少就分发
+ hash关键数值：hash自定义的key

> ip_hash是基于remote_addr的，所以有些代理的请求无法确定实际的Ip地址。

### 9.4 缓存
> Syntax:porxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time],
Default:—
Context:http

> Syntax:proxy_cache zone | off;
   Default:proxy_cache off;
   Context:http,server,location

> 缓存的时间
   Syntax:proxy_cache_valid [code ...] time;
   Default:—
   Context:http,server,location

> 缓存的维度
   Syntax:proxy_cache_key String;
   Default:proxy_cache_key $scheme$porxy_host$request_uri;
   Context:http,server,location

+ 示例：
```

    upstream ljc2{
        server 203.195.133.72:91;
        server 203.195.133.72:92;
        server 203.195.133.72:93;
    }
    proxy_cache_path /opt/app/cache levels=1:2 keys_zone=imooc_cache:10m max_size=10g inactive=60m use_temp_path=off;
server {
    listen       85;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        proxy_cache imooc_cache;
        proxy_pass http://ljc2;
        proxy_cache_valid 200 304 12h;   #如果返回状态是200或者是304则缓存12小时
        proxy_cache_valid any 10m;    #其余缓存10分钟
        proxy_cache_key $host$uri$is_args$args;     #设置缓存key
        add_header Nginx-Cache "$upstream_cache_status";
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;   #如何符合以上情况会转发到另一个upstream
    }

    #error_page  404              /404.html;

```


### 9.5 补充
+ 如何清理指定缓存？
1.rm -rf缓存目录内容
2.第三方扩展模块ngx_cache_purge

+ 如何让部分页面不缓存
> Syntax:proxy_no_cache string...;
   Default:—
   Context:http,server,location
