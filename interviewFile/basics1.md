## 1. JDK和JRE有什么区别？
JRE：JRE是Java Runtime Environment的缩写，顾名思义是java运行时环境，包含了java虚拟机，java基础类库。是使用java语言编写的程序运行所需要的软件环境，是提供给想运行java程序的用户使用的，还有所有的Java类库的class文件，都在lib目录下，并且都打包成了jar。

至于在Windows上的虚拟机是哪个文件呢？就是<JRE安装目录>/bin/client中的jvm.dll。

JDK：Jdk是Java Development Kit的缩写，顾名思义是java开发工具包，是程序员使用java语言编写java程序所需的开发工具包，是提供给程序员使用的。JDK包含了JRE，同时还包含了编译java源码的编译器javac，还包含了很多java程序调试和分析的工具：jconsole，jvisualvm等工具软件，还包含了java程序编写所需的文档和demo例子程序。

如果你需要运行java程序，只需安装JRE就可以了。如果你需要编写java程序，需要安装JDK。

## 2.==和equals的区别是什么？
### ==：
对于基本类型和引用类型==的作用效果是不同的，如下所示：
+ 基本类型：比较的是值是否相同；
+ 引用类型：比较的是引用是否相同。所以，除非是同一个new出来的对象，他们的比较后的结果为true，否则比较后结果为false。  **JAVA当中所有的类都是继承于Object这个基类的，在Object中的基类中定义了一个equals的方法，这个方法的初始行为是比较对象的内存地 址，但在一些类库当中这个方法被覆盖掉了，如String,Integer,Date在这些类当中equals有其自身的实现，而不再是比较类在堆内存中的存放地址了。**
```java
String x = "string";
String y = "string";
String z = new String("string");
System.out.println(x==y); // true
System.out.println(x==z); // false
System.out.println(x.equals(y)); // true
System.out.println(x.equals(z)); // true
```
代码解读：程序在运行的时候会创建一个字符串缓冲池，当使用y=”string"这样的表达式创建字符串的时候，程序首先会在这个String缓冲池中寻找相同值的对象，由于x先被放到了缓冲池中，所以在y被创建的时候，程序找到了具有相同值的x，将x的引用交给了y。而z使用的new操作符，它告诉程序我需要一个新的，于是创建了一个新对象到内存中。

修改下程序：
```java
String x = "string";
String z = new String("string");
z=z.intern();
System.out.println(x==z); //true
System.out.println(x.equals(z)); // true
```
java.lang.String的intern()方法"string".intern()方法的返回值还是字符串"string"，表面上看起来好像这个方 法没什么用处。但实际上，它做了个小动作：检查字符串池里是否存在"string"这么一个字符串，如果存在，就返回池里的字符串；如果不存在，该方法会 把"string"添加到字符串池中，然后再返回它的引用。

### equals:
equals 本质上就是 ==，只不过 String 和 Integer 等重写了 equals 方法，把它变成了值比较。

总结 ：== 对于基本类型来说是值比较，对于引用类型来说是比较的是引用；而 equals 默认情况下是引用比较，只是很多类重写了 equals 方法，比如 String、Integer 等把它变成了值比较，所以一般情况下 equals 比较的是值是否相等。

## 3.两个对象的hashCode()相同，则equals()也一定为 true，对吗？
不对。
```java
        String str1 = "通话";
        String str2 = "重地";
        System.out.println(String.format("str1：%d | str2：%d",  str1.hashCode(),str2.hashCode()));
        System.out.println(str1.equals(str2));
```
执行结果：
> str1：1179395 | str2：1179395
false

JDK中关于hashCode的规定：
>  hashCode 
public int hashCode()返回该对象的哈希码值。支持此方法是为了提高哈希表（例如 [Java](http://lib.csdn.net/base/java "Java 知识库").util.Hashtable 提供的哈希表）的性能。
hashCode 的常规协定是： 
        在 Java 应用程序执行期间，在对同一对象多次调用 hashCode 方法时，必须一致地返回相同的整数，前提是将对象进行 equals 比较时所用的信息没有被修改。从某一应用程序的一次执行到同一应用程序的另一次执行，该整数无需保持一致。 
如果根据 equals(Object) 方法，两个对象是相等的，那么对这两个对象中的每个对象调用 hashCode 方法都必须生成相同的整数结果。 
如果根据 equals(java.lang.Object) 方法，两个对象不相等，那么对这两个对象中的任一对象上调用 hashCode 方法不 要求一定生成不同的整数结果。但是，程序员应该意识到，为不相等的对象生成不同整数结果可以提高哈希表的性能。 
实际上，由 Object 类定义的 hashCode 方法确实会针对不同的对象返回不同的整数。（这一般是通过将该对象的内部地址转换成一个整数来实现的，但是 JavaTM 编程语言不需要这种实现技巧。） 

JDK规定当你调用equals方法比较两个对象相等时，他们调用hashcode方法时，都应该返回相同的整数值，也就是hashcode相等。记住，是应该相同。为什么应该？下面这段红色字体说了，必须重写hashcode方法维护协定！如果你不重写，那么就不能保证hashcode返回相同结果。
换句话说：重写equals方法时请必须重写hashcode方法，以保证equals方法相等时两个对象hashcode返回相同的值。如果没有重写hashcode方法的话，hashcode的值就不一定相等了。

## 4.final在java中有什么作用？
+ final修饰的类叫最终类，该类不能被继承。
+ final修饰的方法不能被重写。
+ final修饰的变量叫常量，常量必须初始化，初始化之后值就不能被修改。

**final的一些重点知识：**
1.final成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误。
2.不能够对final变量再次赋值。
3.在匿名类中所有变量都必须是final变量。
4.final和abstract这两个关键字是反相关的，final类就不可能是abstract的。
5.final方法在编译阶段绑定，称为静态绑定。
6.对于集合对象声明为final指定是引用不能被更改，但是你可以向其中增加，删除或者修改内容。

## 5.java 中的 Math.round(-1.5) 等于多少？
等于 -1，因为在数轴上取值时，中间值（0.5）向右取整，所以正 0.5 是往上取整，负 0.5 是直接舍弃。
**Math类的几个常见方法：**
+ ceil()：返回大于等于（>=）给定参数的最小整数。
+ floor()：返回小于等于（<=）给定参数的最大整数。
+ rint()：返回与参数最接近的整数。返回类型为double。
+ round()：6它表示四舍五入，算法为Math.floor(x+0.5)，即将原来的数字加上0.5后再向下取整。

## 6.String 属于基础的数据类型吗？
Strng不属于基础类型，基础类型有8种：byte，boolean，char，short，int，float，long，double。String属于对象。

## 7.java 中操作字符串都有哪些类？它们之间有什么区别？
操作字符串的类有：String、StringBuffer、StringBuilder。
String 和 StringBuffer、StringBuilder 的区别在于 String 声明的是不可变的对象，每次操作都会生成新的 String 对象，然后将指针指向新的 String 对象，而 StringBuffer、StringBuilder 可以在原有对象的基础上进行操作，所以在经常改变字符串内容的情况下最好不要使用 String。
StringBuffer 和 StringBuilder 最大的区别在于，StringBuffer 是线程安全的，而 StringBuilder 是非线程安全的，但 StringBuilder 的性能却高于 StringBuffer，所以在单线程环境下推荐使用 StringBuilder，多线程环境下推荐使用 StringBuffer。

## 8.String str="i"与 String str=new String("i")一样吗？
不一样，因为内存的分配方式不一样。String str="i"的方式，java 虚拟机会将其分配到常量池中；而 String str=new String("i") 则会被分到堆内存中。

## 9.接口和抽象类有什么区别？
+ 实现：抽象类的子类使用extends来继承；接口必须使用implements来实现接口。
+ 构造函数：抽象类可以有构造函数；接口不能有。
+ main方法：抽象类可以有main方法，并且我们能运行她；接口不能有main方法。
+ 实现数量：类可以实现很多个接口；但只能继承一个抽象类。
+ 访问修饰符：接口中的方法默认使用public修饰；抽象类中的方法可以是任意访问修饰符。

## 10.面向对象的特征？
+ 封装
+ 抽象
+ 继承
+ 多态

## 11.int 和 Integer 有什么区别？
Integer是int的包装类，int的初值为0，Integer的初值为null。
```java
        int i = 128;
        Integer i2 = 128;
        Integer i3 = new Integer(128);
        //Integer会自动拆箱为int，所以为true
        System.out.println(i == i2);
        System.out.println(i == i3);
        System.out.println("**************");
        Integer i5 = 127;//java在编译的时候,被翻译成-> Integer i5 = Integer.valueOf(127);
        Integer i6 = 127;
        System.out.println(i5 == i6);//true
        /*Integer i5 = 128;
        Integer i6 = 128;
        System.out.println(i5 == i6);//false
*/        Integer ii5 = new Integer(127);
        System.out.println(i5 == ii5); //false
        Integer i7 = new Integer(128);
        Integer i8 = new Integer(123);
        System.out.println(i7 == i8);  //false
```
首先，i和i2，i3比较都会输出true，这是因为Integer和int比都会自动拆箱。
i5和i6第一次比较输出的是true，而第二次输出的就是false，这是为什么呢？在java编译的时候，Integer i5=127会被翻译为Integer.valueOf(127)。以下是valueOf方法的源码：
```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
```java
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```
从源代码可以看出-128到127之间的值，会直接返回缓存中对应的值，并不会new，所以第二次比较为false。
> **总结：**
> + 无论如何，Integer和new Integer都不会相等。不会经历拆箱过程，类似i2和i3，i5和ii5的比较。
> + 如果两个都是非new出来的Integer，如果数在-128到127之间，则为true，否则为false。
> + int和Integer（无论是通过什么创建），都为true，因为会把Integer自动拆箱为int再去比。

## 12.重载和重写的区别?
方法的重载和重写都是实现多态的方式，区别在于前者实现的是编译时的多态性，而后者实现的是运行时的多态性。重载发生在一个类中，同名的方法如果有不同的参数列表（参数类型不同、参数个数不同或者二者都不同）则视为重载；重写发生在子类与父类之间，重写要求子类被重写方法与父类被重写方法有相同的参数列表，有兼容的返回类型，比父类被重写方法更好访问，不能比父类被重写方法声明更多的异常（里氏代换原则）。重载对返回类型没有特殊的要求，不能根据返回类型进行区分。

## 13.session 与 cookie 区别？
+ session是保存在服务器端，理论上是没有大小限制，只要内存够大。
+ 浏览器第一次访问服务器的时候后创建一个session对象并返回一个JSESSIONID=ID的值，创建一个Cookie对象key为JISSIONID，value为ID的值是，将这个Cookie协会浏览器。
+ 浏览器在第二次访问服务器的时候携带Cooike信息JSESSIONID=ID的值，如果该JESSIONID的session已经销毁，那么会重新创建一个新的session再返回一个新的JSESSIONID通过Cookie返回到浏览器。
+ 针对一个web项目，一个浏览器是共享一个session，就算有两个web项目部署在同一个服务器上，针对两个项目的session是不同的。
+ session是基于Cookie技术实现，重启浏览器后再次访问原有的连接依然会创建一个新的session，因为Cookie在关闭浏览器后就会消失，但是原来服务器的Session还在，只有等到了销毁的时间会自动销毁。
+ 如果浏览器端禁用了Cookie，那么每次访问都会创建一个新的Session，但是我们可以通过服务器端程序重写URL即可，如果页面多连接多，会增加不必要的工作量，
   那可以强制让你用户开启接收Cookie后再让其访问即可。

**说说Cookie和Session的区别？**
   
   1、Cookie和Session都是会话技术，Cookie是运行在客户端，Session是运行在服务器端。
   
   2、Cookie有大小限制以及浏览器在存cookie的个数也有限制，Session是没有大小限制和服务器的内存大小有关。

   3、Cookie有安全隐患，通过拦截或本地文件找得到你的cookie后可以进行攻击。

   4、Session是保存在服务器端上会存在一段时间才会消失，如果session过多会增加服务器的压力。

## 14.HTTP 请求的 GET 与 POST 方式的区别？
+ GET在浏览器回退时是无害的，而POST会再次提交请求。
+ GET产生的URL地址可以被Bookmark，而POST不可以。
+ GET请求会被浏览器主动cache，而POST不会，除非手动设置。
+ GET请求只能进行url编码，而POST支持多种编码方式。
+ GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
+ GET请求在URL中传送的参数是有长度限制的，而POST么有。
+ 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
+ GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
+ GET参数通过URL传递，POST放在Request body中。

GET和POST是什么？HTTP协议中的两种发送请求的方法。
HTTP是什么？HTTP是基于TCP/IP的关于数据如何在万维网中如何通信的协议。
HTTP的底层是TCP/IP。所以GET和POST的底层也是TCP/IP，也就是说，GET/POST都是TCP链接。GET和POST能做的事情是一样一样的。你要给GET加上request body，给POST带上url参数，技术上是完全行的通的。 


GET和POST还有一个重大区别，简单的说：

GET产生一个TCP数据包；POST产生两个TCP数据包。

对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；

而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。

也就是说，GET只需要汽车跑一趟就把货送到了，而POST得跑两趟，第一趟，先去和服务器打个招呼“嗨，我等下要送一批货来，你们打开门迎接我”，然后再回头把货送过去。

因为POST需要两步，时间上消耗的要多一点，看起来GET比POST更有效。因此Yahoo团队有推荐用GET替换POST来优化网站性能。但这是一个坑！跳入需谨慎。为什么？

1. GET与POST都有自己的语义，不能随便混用。
2. 据研究，在网络环境好的情况下，发一次包的时间和发两次包的时间差别基本可以无视。而在网络环境差的情况下，两次包的TCP在验证数据包完整性上，有非常大的优点。
3. 并不是所有浏览器都会在POST中发送两次包，Firefox就只发送一次。

[原文链接](https://www.cnblogs.com/logsharing/p/8448446.html#!comments)


