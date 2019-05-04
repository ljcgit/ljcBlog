## 1.properties
可以使用properties来引入外部properties配置。
+  resource:引入类路径下的资源
+ url：引入网络路径或者磁盘路径下的资源

**dbconfig.properties**
> jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/bookstore
jdbc.username=root
jdbc.password=

**引入资源文件**
```xml
    <properties resource="dbconfig.properties"></properties>
```

**使用**
```xml
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value=""/>
            </dataSource>
```

## 2. settings

## 3. typeAliases
为某个java类起别名
+ type：指定要其别名的类型全类名；默认别名就是类名小写
+ alias：指定新的别名
```xml
 <typeAlias type="com.bean.Book" alias="book"/>
 ```

**批量起别名**
```xml
  <package name="com.bean"/>
 ```

name：指定包名（为当前包以及下面所有的后代包的每一个类都起一个默认别名（类名小写）

**注解形式**
> @Alias(value = "ljc")


## 4. environments
default指定使用某种环境。
id：当前环境的唯一标识
transactionManager：事务管理器
       type：事务管理器的类型
dataSource：数据源

## 5.mapper
注册一个sql映射
resource：引入类路径下的sql映射文件
url：引用网络路径或者磁盘路径下的sql映射文件
class：引用（注册）接口
    1.有sql映射文件，映射文件名必须和接口同名，并且放在与接口同一目录下
     2.没有sql映射文件，所有的sql都是利用注解写在接口上

