# Java获取静态资源文件路径



​		开发中经常会有文件相关的操作，比如说文件的查找，文件的存储。但是在使用中往往会因为路径问题耽误大量的开发时间。在实际中常常通过File file = new File("c://a.txt")方式来使用，但是切换环境后可能会遇到找不到该资源的问题。Java中，可以使用getResource()方法来获取资源。




**项目工程如下：**<br/>
<center>
<img src="/images/java%20getResource.png" width="25%" height="25%" />
项目工程
</center>

## 1.Class和ClassLoader对于getResouce()的区别？

```java
public class ResourceTest {

    @Test
    public void test1() {
        // /Users/luojiacheng/code/translaction-test/target/test-classes/com/ljc/
        System.out.println(ResourceTest.class.getResource("").getPath());
        // /Users/luojiacheng/code/translaction-test/target/test-classes/
        System.out.println(ResourceTest.class.getResource("/").getPath());

        // /Users/luojiacheng/code/translaction-test/target/test-classes/
        System.out.println(ResourceTest.class.getClassLoader().getResource("").getPath());
        // null
        System.out.println(ResourceTest.class.getClassLoader().getResource("/"));
    }


}
```



Class:

getResource("")获取的是当前文件的路径；

getResouce("/")获取的是classes下的路径，在spring中相当于/WEB-INF/classes/下；



ClassLoader:

getResource("")获取的是classes下的路径，等同于Class.getResource("/")；

getResouce("/")无法获取。



## 2.getResourceAsStream()

```java
public InputStream getResourceAsStream(String name) {
    URL url = getResource(name);
    try {
        return url != null ? url.openStream() : null;
    } catch (IOException e) {
        return null;
    }
}
```



getResourceAsStream()其实就是在getReource() + new InputStream()。





## 3.Class.getResource()实现

```java
public java.net.URL getResource(String name) {
    // 获取包路径
    name = resolveName(name);
    ClassLoader cl = getClassLoader0();
    if (cl==null) {
        // A system class.
        return ClassLoader.getSystemResource(name);
    }
  	// "" -> com/.../    "/" -> ""
    return cl.getResource(name);
}
```



Class的getResouce方法实际调用就是ClassLoader的getResource方法，只不过在调用前对文件名进行了额外的处理，将“”转为了包路径，“/”转为了“”。


## 4.示例
```java

    /**
     * 无内容返回
     */
    @GetMapping("/getTextFromFile")
    public String getTextFromFile() throws IOException {
        String filePath = MyFileOperation.class.getResource("/a.txt").getPath();
        // **.jar!/BOOT-INF/classes!/a.txt
        System.out.println(filePath);
        File file = new File(filePath);
        StringBuilder s = new StringBuilder();
        if (file.exists()) {
            InputStreamReader isr = new InputStreamReader(new FileInputStream(file), "gbk");
            BufferedReader bufferedReader = new BufferedReader(isr);
            String str;
            while ((str = bufferedReader.readLine()) != null) {
                s.append(str);
            }
        }
        return s.toString();
    }
    
    
    /**
     * 正确
     */
    @GetMapping("/getTextFromFileByStream")
    public String getTextFromFileByStream() throws IOException {
        StringBuilder s = new StringBuilder();
        InputStreamReader isr = new InputStreamReader( MyFileOperation.class.getResourceAsStream("/a.txt"), "gbk");
        BufferedReader bufferedReader = new BufferedReader(isr);
        String str;
        while ((str = bufferedReader.readLine()) != null) {
            s.append(str);
        }
        return s.toString();
    }
```



