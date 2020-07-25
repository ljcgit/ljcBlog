# 问题：String是否是线程安全的？

> 对于这个问题，自己以前不知道为什么，就认为String、StringBuffer、StringBuilder中线程安全就只有StringBuffer。最近还是search了好久，就来总结一下。

首先，看看String类的实现：
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```
String内部存储使用的是一个char数组，只不过数组使用final修饰，同时String类也使用了final修饰。
数组用final修饰保证了数组的值是唯一确定的，也就是不可修改，并且数组是char类型，也是Java的基本类型，这样还避免了一些引用对象内部发生变化。
类用final修改代表该类不能被继承，也就不会有子类复写导致不安全。
所以对于String是线程安全的。
还有一点能证明String是线程安全的，String的所有对象都存在于常量池中。
但是，在一些使用场景，String也会引发线程不安全，比如：
```java
public class test1 {

    static String s = "";

    public static void main(String[] args) throws InterruptedException {
        Thread a = new Thread(test1::add,"a");
        Thread b = new Thread(test1::add,"b");
        Thread c = new Thread(test1::add,"c");
        Thread d = new Thread(test1::add,"d");
        a.start();
        b.start();
        c.start();
        d.start();

        a.join();
        b.join();
        c.join();
        d.join();

        System.out.println(s.length());
    }

    public static void add(){
        Thread thread = Thread.currentThread();
        String t = thread.getName();
        for(int i = 0;i < 1000; i++){
            s = s + t;
        }
    }
}

```
这个就是简单模拟4个线程同时修改一个String对象的情况，这个例子中，最后s的长度理想状态下应该是4000，但是实际却不到。


**String本身的确是线程安全的，但是在具有使用中，一些场景还是可能会引发线程不一致的情况，所以多线程下可以使用StringBuffer，所有的方法都会加锁来实现。**

