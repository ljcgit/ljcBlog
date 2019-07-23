## 1.rewrite
> Syntax:rewrite regex replacement [flag];
   Default:—
   Context:server,location,if
## 1.1 flag类型
+ last：停止rewrite检测
+ break：停止rewrite检测
+ redirect：返回302临时重定向，地址栏会显示跳转后的地址
+ permanent：返回301永久重定向，地址栏会显示跳转后的地址

#### 1.1.1 示例
```

server {
    listen       86;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    root /opt/app/code;
    location ~ ^/break {
        rewrite ^/break /test/ break;
    }

    location ~ ^/last {
        rewrite ^/last /test/ last;
    }

    location /test/ {
        default_type application/json;
        return 200 '{"status":"success"}';
    }
}
```
> 这里访问break显示404，last会显示test的内容。是由于last相当于重新请求了一次url请求，而break会到/opt/app/code目录下查找test文件，由于没有该文件，所以显示404。


## 2.secure_link_module
+ 制定并允许检查请求的链接的真实性以及保护资源免遭未经授权的访问
+ 限制链接生效周期

> Syntax:secure_link expression;
   Default:—
   Context:http,server,location

> Syntax:secure_link_md5 expression;
   Default:—
   Context:http,server,location

## 2.1 示例
+ 
```
#!/bin/sh
#
#Auth:
servername="203.195.133.72:87"
download_file="/download/web.jpg"
time_num=$(date -d "2019-10-01 00:00:00" +%s)
secret_num="ljc"


res=$(echo -n "${time_num}${download_file} ${secret_num}"|openssl md5 -binary | openssl base64 | tr +/ -_ | tr -d =)
echo "http://${servername}${download_file}?md5=${res}&expires=${time_num}"
```

+ test_safe_down.conf
```
server {
    listen       87;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;
    root /opt/app/code;

    location / {
        secure_link $arg_md5,$arg_expires;
        secure_link_md5 "$secure_link_expires$uri ljc";
        if ($secure_link = "") {
           return 403;
        }

        if ($secure_link = "0") {
           return 410;
        }
    }
```

## 3.geoip_module
+ 安装
> yum install nginx-module-geoip

+ 修改nginx.conf配置
> load_module "modules/ngx_http_geoip_module.so";
   load_module "modules/ngx_stream_geoip_module.so";

+ 下载相应的地域和城市数据库文件
```sh
#!/bin/sh
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz
wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
```

### 3.1 http_geoip_module使用场景
+ 区别国内外作HTTP访问规则
+ 区别国内城市地域作HTTP访问规则


## 4.Https

### 4.1 生成Ca证书
+ 确保具有openssl和http_ssl_module模块

+ 生成key密钥
> 在nginx下创建ssl_key文件,在ssl_key文件下执行 *openssl genrsa idea -out ljc.key 1024* 命令。此时该目录下就会出现ljc.key目录。

![生成key](https://upload-images.jianshu.io/upload_images/16503287-d4567a02d706af3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


+ 生成证书签名签名文件（csr文件）
> 执行 *openssl req -new -key ljc.key -out ljc.csr* 命令，并填写信息。
![csr](https://upload-images.jianshu.io/upload_images/16503287-2469011e23d6b0cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

+ 生成证书签名文件（CA文件）
> 执行 *openssl x509 -req -days 3650 -in ljc.csr -signkey ljc.key -out ljc.crt* 命令。
![CA](https://upload-images.jianshu.io/upload_images/16503287-c7cd849ee7b44078.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4.2Nginx的HTTPS语法配置
> Syntax:ssl on | off;
   Default:ssl off;
   Context:http,server

> Syntax:ssl_certificate file;
   Default:—;
   Context:http,server

> Syntax:ssl_certificate_key file;
   Default:—;
   Context:http,server

### 4.3 测试CA
+ 新建test_https.conf
```
server
{
   listen   443 ssl;    //这里是和老版本的有所不同（原来使用ssl on）
   server_name IP;
   ssl_certificate /etc/nginx/ssl_key/ljc.crt;
   ssl_certificate_key /etc/nginx/ssl_key/ljc.key;

   index index.html index.htm;
   location / {
        root /opt/app/code;
   }
}
```


## 5.Lua
+ 安装
> yum install Lua

### 5.1 Nginx调用lua模块指令
![模块指令](https://upload-images.jianshu.io/upload_images/16503287-e2737c41c6a5edb0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



