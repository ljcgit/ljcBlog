通过Enum往数据库中添加数据默认在数据库中保存的是Enum的name。如果要保存Enum的value，可以通过改变handler来实现。
```xml
    <typeHandlers>
        <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler" javaType="com.StatusEnum"></typeHandler>
    </typeHandlers>
```

## 自定义Hanlder
#### 接口：
```java
public enum StatusEnum {

    SUCCESS(200,"成功"),FAIL(404,"失败");

    private int code;
    private String msg;

     StatusEnum(){}

     StatusEnum(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public static StatusEnum getStatus(int code){
         switch (code){
             case 200 : return SUCCESS;
             default: return FAIL;
         }
    }
}
```
#### TypeHandler实现：
 ```java
/**
 * 枚举类型解析器
 */
public class MyEnumHandler implements TypeHandler<StatusEnum> {
    @Override
    public void setParameter(PreparedStatement preparedStatement, int i, StatusEnum statusEnum, JdbcType jdbcType) throws SQLException {
        preparedStatement.setString(i,Integer.toString(statusEnum.getCode()));
    }

    @Override
    public StatusEnum getResult(ResultSet resultSet, String s) throws SQLException {

        String code = resultSet.getString(s);

        return StatusEnum.getStatus(Integer.parseInt(code));
    }

    @Override
    public StatusEnum getResult(ResultSet resultSet, int i) throws SQLException {

        String code = resultSet.getString(i);

        return StatusEnum.getStatus(Integer.parseInt(code));
    }

    @Override
    public StatusEnum getResult(CallableStatement callableStatement, int i) throws SQLException {

        String code = callableStatement.getString(i);

        return StatusEnum.getStatus(Integer.parseInt(code));
    }
}
```
#### 配置属性
```xml
    <typeHandlers>
        <!--<typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler" javaType="com.StatusEnum"></typeHandler>-->
        <typeHandler handler="com.myHandler.MyEnumHandler" javaType="com.StatusEnum"/>
    </typeHandlers>
```
> 我们也可以直接在SQL语句中指定：
> \#{statusEnum,typeHandler=com.myHandler.MyEnumHandler}

