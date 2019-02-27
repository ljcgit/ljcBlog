首先看下面一段代码：
```java
    public static void main(String[] args) {
    	boolean flag=2;  //无法直接通过编译
    	if(flag) System.out.println("Hello,Java!");
    	if(flag==true) System.out.println("Hello,JVM!");
    }
```
## Java虚拟机中的boolean类型
+ Java语言规范中，boolean值类型只有“true”和“false”两种。而这两种符号在虚拟机中是不能被直接使用。
+ Java虚拟机规范中，boolean类型会被映射成int类型，即1和0。

## Java基本类型
除了上面的boolean类型，还有byte,short,char,int,long,float,double。
> NaN:在类型取值范围之外的值一般都称之为NaN。  
   NaN有一个有趣的特性：除了!=始终都返回true之外，所有其他比较结果都会返回false。
  举例：NaN<1.0F返回false，NaN>=1.0F返回flase，只有NaN!=1.0F返回true。

## Java基本类型大小
Java虚拟机每调用一个Java方法，便会创建一个栈帧（这里以解释器使用的解释栈帧为例）。

这种栈帧有两个主要的组成部分，分别数局部变量区，以及字节码的操作数栈。这里的局部变量是广义的，除了普遍意义下的局部变量之外，它还包含实例方法的“this指针”以及方法所接收的参数。

在Java虚拟机规范中，局部变量区等价于一个数组，并且可以用正整数来索引。除了long，double值需要用两个数组单元来存储之外，其他基本类型以及引用类型的值均占用一个数组单元。也就是说boolean，byte，char，shory这四个类型，在栈上占用的空间和int是一样的，和引用类型也是一样的。

当然，这种情况仅存在于局部变量，而并不会出现在存储于堆中的字段或者数组元素上。对于byte，char以及short这三种类型的字段或者数组单元，它们在堆上占用的空间分别为一字节，两字节，也就是说，跟这些类型的值域相吻合。

因此，当我们将一个int类型的值，存储到这些类型的字段或数组时，相当于做了依次隐式的掩码操作，即把高位截取掉。

Java虚拟机的算数运算几乎全部依赖于操作数栈。也就是说，我们需要将堆中的boolean，byte，char以及short加载到操作数栈上，而后将栈上的值当成int类型来运算。

在最上面的例子中，可以输出HelloJava，而不能输出HelloJVM，因为第一个if语句是在值不为0的情况下输出，而第二个if语句是对true和flag进行比较输出，所以第二个输出语句不会执行。

当将例子中的2改为3呢？两条语句都会输出，因为3在保存到堆时，因为指明了boolean类型，只取最低位1，所以和true进行比较时结果也为真。


