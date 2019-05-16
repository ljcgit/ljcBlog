# 异常的基本概念
在Java语言规范中，所有异常都是Throwable类或者其子类的实例。Throwable有两大直接子类。第一个是Error，涵盖程序不应捕获的异常。当程序触发Error时，它的执行状态已经无法恢复，需要中止线程甚至是中止虚拟机。第二子类则是Exception，涵盖程序可能需要捕获并且处理的异常。

Exceptionu有一个特使的子类RuntimeException，用来表示“程序虽然无法继续执行，但是还能抢救一下”的情况。例如：数组索引越界。

RuntimeException和Error属于Java里的非检查异常（unchecked exception）。其他异常则属于检查异常（checked exception）。在Java语法中，所有的检查异常都需要程序显式地捕获（catch），或者在方法声明中用throws关键字标注。通常情况下，程序中自定义的异常应为检查异常，以便最大化利用Java编译器的编译时检查。

异常实例的构造十分昂贵。这是由于在构造异常实例时，Java虚拟机便需要生成该异常的栈轨迹（stack trace）。该操作会逐一访问当前线程的Java栈帧，并且记录下各种调试信息，包括栈帧所指向方法的名字，方法所在的类名，文件名，以及在代码中的第几行触发该异常。

当然，在生成栈轨迹时，Java虚拟机会忽略掉异常构造器以及填充栈帧的Java方法，直接从新建异常位置开始算起。此外，Java虚拟机还会忽略标记为不可见的Java方法栈帧。

异常对应的栈轨迹并非throw语句的位置，而是新建异常的位置。

# Java虚拟机是如何捕获异常的？
在编译生成的字节码中，每个方法都附带一个异常表。异常表中的每一个条目代表一个异常处理器，并且由from指针，to指针，target指针以及所捕获的异常类型构成。这些指针的值是字节码索引，用以定位字节码。

其中，from指针和to指针标识了该异常处理器所监控的范围，例如try代码块所覆盖的范围。target指针则指向异常处理器的起始位置，例如catch代码块的起始位置。

当程序触发异常时，Java虚拟机会从上至下遍历异常表中的所有条目。当触发异常的字节码的索引值在某个异常表条目的监控范围，Java虚拟机会判断所抛出的异常和该条目想要捕获的异常是否匹配。如果匹配，Java虚拟机会将控制流转移至该条目target指针指向的字节码。

如果遍历完所有异常表条目，Java虚拟机仍未匹配到异常处理器，那么它会弹出当前方法对应的Java栈帧，并且在调用者中重复上述操作。在最坏情况下，Java虚拟机需要遍历当前线程Java栈上所有方法的异常表。

![](https://upload-images.jianshu.io/upload_images/16503287-681f01185e526ba6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

针对异常执行路径，Java编译器会生成一个或多个异常表条目，监控整个try-catch代码块，并且捕获所有种类的异常。这些异常表条目的target指针将指向另一份复制的finally代码块。并且，在这个finally代码块的最后，Java编译器会重新抛出所捕获的异常。

可见，一共有三份finally代码块。其中，前两份分别位于try代码块和catch代码块的正常执行路径出口。最后一份则作为异常处理器，监控try代码块以及catch代码块。它将捕获try代码块触发的未被catch代码块捕获的异常，以及catch代码块触发的异常。

有个小问题？如果catch代码块捕获了异常，并且出发了另一个异常，那么finally捕获并且重抛的异常是哪个呢？答案是后者。也就是所原本的异常便会被忽略掉，不易于代码调试。
