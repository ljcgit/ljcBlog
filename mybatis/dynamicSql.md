## 1.返回自增主键
> useGeneratedKeys="true" keyProperty="bid"
在标签上设置这两个属性值。useGeneratedKeys表示使用自增主键，keyProperty表示返回的主键要赋给谁。

## 2.参数处理
+ 单个参数：不做特殊处理 #{参数名}：取出参数值。
+ 多个参数：#{param1} #{param2} 
+ 命名参数：@Param()  明确指出参数名   
+ POJO
+ Map
+ TO
> 如果是Collection（List，set)类型或者是数组，也会特殊处理。也就是把传入的list或者数组封装在map中。
Collection  -> collection    List  -> list     数组  -> array

#### #{}和${}
{}：是以预编译的形式，将参数设置到sql语句中
${}：取出的值直接拼装在sql语句中

## 3 多对一级联查询
+ 先构建一方的查询
+ 自定以映射规则
```xml
    <!--自定义结果映射规则-->
    <resultMap id="MyBook" type="book">
        <id column="bid" property="bid"/>
        <result column="bname" property="bname"/>
        <result column="price" property="price"/>
        <result column="author" property="author"/>
        <result column="bnum" property="bnum"/>
        <association property="pub" column="pid" select="com.dao.PublishersMapperAnnotation.getById">
        </association>
    </resultMap>
```

## 4 一对多查询
+ 直接查询出所有数据
```xml
    <!--自定义结果映射规则-->
    <resultMap id="MyPublishers" type="com.bean.Publishers">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <collection property="books" ofType="book">
                <id column="bid" property="bid"/>
                <result column="bname" property="bname"/>
                <result column="price" property="price"/>
                <result column="author" property="author"/>
                <result column="bnum" property="bnum"/>
         </collection>
    </resultMap>

    <select id="getByIdWithBook" resultMap="MyPublishers">
        select * from publishers left join books on publishers.id = books.pid where id = #{id}
    </select>
```
+ 分步查询
 ```xml
    <resultMap id="MyPublishersStep" type="com.bean.Publishers">
        <id column="id" property="id"/>
        <result column="name" property="name"/>
        <collection property="books" column="id" select="com.dao.BookMapper.getBookByPid"/>
    </resultMap>
    <select id="getByIdWithBookStep" resultMap="MyPublishersStep">
        select * from publishers where id = #{id}
    </select>
```
```xml
    <select id="getBookByPid" resultType="book">
        select * from books where pid = #{pid}
    </select>
```
> collection传递多个值：采用map形式{key1:value1,key2:value2}

## 5 鉴别器
```xml
    <!--自定义结果映射规则+ 鉴别器-->
    <resultMap id="MyBookDis" type="book">
        <id column="bid" property="bid"/>
        <result column="bname" property="bname"/>
        <result column="price" property="price"/>
        <result column="author" property="author"/>
        <result column="bnum" property="bnum"/>
        <discriminator javaType="int" column="bnum">
            <case value="1" resultType="book">
                <association property="pub" column="pid" select="com.dao.PublishersMapperAnnotation.getById">
                </association>
            </case>
            <case value="2" resultType="book">
                <result column="bname" property="author"/>
            </case>

        </discriminator>
    </resultMap>
```

## 6 用中文进行数据库查询时无效
在url后面加上useUnicode=true&characterEncoding=utf-8

## 7 动态sql语句 —— if
```xml
    <select id="getBooksByConditionIf" resultType="book">
        select * from books where
        <if test="bname!=null">
             bname = #{bname}
        </if>
        <if test="bnum == 1">
             bnum = #{bnum}
        </if>
    </select>
```
## 8 动态sql语句——where
会自动去掉首部的and或者or

## 9 动态sql语句——trim
+ prefix=""：前缀 trim标签体是整个字符串拼串后的结果 prefix给拼串后的整个字符串加一个前缀
+ prefixOverrides=""：前缀覆盖 去掉整个字符串前面多余的字符
+ suffix=""：后缀 suffix给拼串后的整个字符串加一个前缀
+ suffixOverrides=""：后缀覆盖 去掉整个字符串后面多余的字符

## 10 动态sql语句——choose
```xml
    <select id="getBooksByConditionChoose" resultType="book">
        select * from books
        <where>
            <choose>
                <when test="bname!=null">
                    bname = #{bname}
                </when>
                <when test="bnum == 1">
                    bnum = #{bnum}
                </when>
                <otherwise>
                    1=1
                </otherwise>
            </choose>
        </where>

    </select>
```

## 11 动态sql语句——set
```xml
    <update id="updateBook">
        update books
        <set>
            <if test="bname!=null">
                bname=#{bname},
            </if>
            <if test="price!=null">
                price=#{price},
            </if>
            <if test="author!=null">
                author=#{author},
            </if>
            <if test="bnum!=null">
                bnum=#{bnum}
            </if>
        </set>
        where bid = #{bid}
    </update>
```

## 12 动态sql语句——foreach
```xml
    <select id="getBookByConditionForeach" resultType="book">
        select * from books 
        <foreach collection="list" item="bid" separator="," open="where bid in (" close=")">
            #{bid}
        </foreach>
    </select>
```
+ collection：指定要遍历的集合
       list类型的参数会特殊处理封装在map中，map的key就叫list
+ item：将当前遍历出的元素赋值给指定的变量
+ separator：每个元素之间的分隔符
+ open：遍历出所有结果拼接一个开始的字符
+ close：遍历出所有的结果拼接一个结束的字符
+ \#{item}：就能取处变量的值也就是当前遍历出的元素

## 13 foreach批量插入
```xml
    <insert id="insertBooks">
        insert into books(bname,price,author,bnum) values
        <foreach collection="books" item="book" separator="," >
            (#{book.bname},#{book.price},#{book.author},#{book.bnum})
        </foreach>
    </insert>
```

## 14 内置参数
+ _parameter：代表整个参数
            单个参数：_parameter就是这个参数
            多个参数：参数会被封装为一个map；_parameter就是代表这个map

## 15 bind标签
```xml
        <bind name="_bname" value="'%'+bname+'%'"/>
```
可以动态绑定某个值

## 16 sql标签
+ sql抽取，
+ include来引用已经抽取的sql
+ include还可以自定义一些property，sql标签内部就能使用自定义属性（${})
