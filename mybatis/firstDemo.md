#### 1.在MyBatis的github上下载相应的压缩文件。
> https://github.com/mybatis/mybatis-3/releases

#### 2.创建一个SqlSessionFactory对象
```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory =
new SqlSessionFactoryBuilder().build(inputStream);
```
会根据mybatis的全局配置文件来创建一个SqlSessionFactory工厂。

+ mybatis-config.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/bookstore"/>
                <property name="username" value="root"/>
                <property name="password" value=""/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <mapper resource="BookMapper.xml"/>
    </mappers>
</configuration>
```

#### 3.创建sql映射文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BookMapper">
    <select id="selectBook" resultType="com.bean.Book">
        select * from books where bid = #{id}
    </select>
</mapper>
```
#### 4.通过SqlSessionFactory获取SqlSession对象
> SqlSession session = sqlSessionFactory.openSession();

#### 5.调用SqlSession相应的方法来执行sql语句
```java
try {
Blog blog = session.selectOne(
    "org.mybatis.example.BlogMapper.selectBlog", 101);
} finally {
    session.close();
}
```
> 一定要在使用后关闭SqlSession


