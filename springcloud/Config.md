## 1.打印出Config Server请求Git仓库的细节
``` yml
logging:
  level:
    org.springframework.cloud: debug
    org.springframework.boot: debug
```

## 2.配置内容的加解密
#### 2.1 配置JCE（jdk8）
+ 在Oracle官网下载相应的压缩文件
+ 将压缩文件中的两个jar文件复制到JDK/jre/lib/security下

#### 2.2 对称加密
+ 在服务项目中创建bootstrap.yml并添加以下内容
```yml
encrypt:
  key: foo  #对称加密密钥
```
> **以encrypt开头的配置，必须放在bootstrap.yml中，否则将无法正常加解密。

+ 测试
![对称加密.png](https://upload-images.jianshu.io/upload_images/16503287-7651b056eb80847c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

出现上面输出，说明能够正常加密解密。

#### 2.3 使用加密的内容进行存储
+ 在git上新建一个encryption.yml文件
```yml

spring:
  datasource:
    username: dbuser
    password: '{cipher}8f2dd7e09785613cff2efa0b7cd82fd5c6567ff183bcaffeef4e0cc9a3a04af8'
```
这里password使用的就是上面加密后的密码。
> 这里必须要在密钥两端加上单引号，否则该值不会被解密，而properties格式文件则无需加上单引号。

+ 访问该yml配置 http://localhost:3344/encryption-defautl.yml
![encryption.yml](https://upload-images.jianshu.io/upload_images/16503287-4314c42257ef8a73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 如果想要让Config Servce直接返回密文本身，可以将spring.cloud.config.server.encrypt.enabled=false，这时可由Config Client自行解密。




