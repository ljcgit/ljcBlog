## 1. rmi（remote method invocation）

> 只针对java

client：stud 桩

server：skeleton 骨架



## 2. RPC（Remote Procedure Call）远程过程调用

> 跨语言

- 定义一个接口说明文件：描述了对象（结构体）、对象成员、接口方法等一系列信息。
- 通过RPC框架所提供的编译器，将接口说明文件编译成具体语言文件。
- 在客户端与服务器端分别使用RPC编译器所生成的文件，即可像调用本地方法一样调用远程方法。

> 相对于Rest形式可以减少网络传输。



## 3. Protocol Buffers（Google）

是一种语言独立，平台独立，可拓展的。 

> 由于支持多种语言，所以所支持的类型是各语言的交集。



## 4. 装饰模式

IO



## 5. java.nio

java.io中最为核心的一个概念是流（Stream），面向流的编程。Java中，一个流要么是输入流，要么是输出流，不可能同时既是输入流又是输出流。



java.nio中拥有3个核心概念：Selector（选择器），Channel（通道），Buffer（缓冲）。在java.nio中，我们是面向块（block）或者缓冲区（buffer）编程的。Buffer本身就是一块内存，底层实现上就是个数组，数据的读写都是Buffer来实现的。所有数据的读写都必须经过Buffer。



除了数组之外，Buffer还提供了对于数据的结构化访问方式，并且可以追踪到系统的读写过程。



Java中的7种原生数据类型（无Boolean）都有各自对应的Buffer类型，比如IntBuffer，LongBuffer，CharBuffer，ByteBuffer。



由于Channel是双向的，因此他能更好的反映出底层操作系统的真实情况；在Linux系统中，底层操作系统的通道就是双向的。



## 6. position limit capacity 状态属性

#### 6.1 capacity

buffer所包含的元素个数，值不为负数且不可变。



#### 6.2 limit 

buffer中第一个不能被读和写元素的索引。



#### 6.3 position

buffer中下一个将要被读和写的元素索引。



> 0 <= mark <= position <= limit <= capacity



## 7 字符编码

unicode是一种编码方式，而UTF则是一种存储方式；而UTF-8是Unicode的实现方式之一。



## 8 传统socket编程缺点

- 线程数量过多
- 线程上下文切换开销
- 没有数据传输时线程的空等



## 9 Reactor模式的角色构成

#### 9.1 Handle

**句柄或者描述符**。本质上是一种资源，是有操作系用提供的；该资源表示一个个事件，比如说文件描述符，或是针对网络编程中的Socket描述符。事件既可以来自于外部，也可以来自于内部；外部事件比如说客户端的连接请求，客户端发送过来数据等；内部事件比如说操作系统产生的定时器事件等。它本质上就是一个文件描述符。Handle是事件产生的发源地。

#### 9.2 Synchronous Event Demultiplexer（同步事件分离器）

本身是一个系统调用，用于等待事件的发生（事件可能是一个，也可能是多个）。调用方在调用它的时候会被阻塞，一直阻塞到同步事件分离器上有事件产生为止。对于Linux来说，同步事件分离器指的就是常用的I/O多路复用机制，比如说select、poll、epoll等。在Java NIO领域中，同步事件分离器对应的组件就是Selector；对应的阻塞方法就是select方法。



#### 9.3 Event Handler （事件处理器）

本身由多个回调方法构成，这些回调方法构成了与应用相关的对于某个事件的反馈机制。Netty相比于Java NIO来说，在事件处理器这个角色上进行了一个升级，它为我们开发者提供了大量的回调方法，供我们在特定事件产生时实现相应的回调方法进行业务逻辑的处理。



#### 9.4 Concrete Event Handler（具体事件处理器）

是事件处理器的实现。它本身实现了事件处理器所提供的各个回调方法，从而实现了特定于业务的逻辑。它本身就是我们所编写的一个个的处理器实现。



#### 9.5 Initiation Dispatcher（初始分发器）

实际上就是Reactor角色。它本身定义了一些规范，这些规范用于控制事件的调度方式，同时又提供了应用进行事件处理器的注册、删除等设施。他本身是整个事件处理器的核心所在，Initiation Dispatcher会通过同步事件分离器来等待事件的发生。一旦事件发生，Initiation Dispatcher首先会分离出每一个事件，然后调用事件处理器，最后调用相关的回调方法来处理这些事件。 



## 10 Reactor模式的流程

1.当应用向Initiation Dispatcher注册具体的事件处理器时，应用会标识出该事件处理器希望Initiation Dispatcher在某个事件发生时向其通知的该事件，该事件与Handle关联；

2.Initiation Dispatcher会要求每个事件处理器向其传递内部的Handle。该Handle向操作系统标识了事件处理器；

3.当所有的事件处理器组册完毕后，应用会调用handle_events方法来启动Initiation Dispatcher的事件循环。这时，Initiation Dispatcher会将每个注册的事件处理器的Handle合并起来，并使用同步事件分离器等待这些事件的发生。比如说，TCP层会使用select同步事件分离器操作来等待客户端发送的数据到达连接的socket handle上；

4.当与某个事件源对应的Handle变为ready状态时（比如说，TCP socket变为等待读状态时），同步事件分离器就会通知Initiation Dispatcher；

5.Initiation Dispatcher会触发事件处理器的回调方法，从而响应这个处于ready状态的Handle。当事件发生时，Initiation Dispatcher会将被事件源激活的Handle作为「key」来寻找并分发恰当的事件处理器回调方法。

6.Initiation Dispatcher会回调事件处理器的handle_events回调方法来执行特定于应用的功能（开发者自己所编写的功能），从而响应这个事件。所发生的事件类型可以作为该方法参数并被该方法内部使用来执行额外的特定于服务的分离与分发。



## 11.EventLoop

- 一个EventLoopGroup当中会包含一个或多个EventLoop。
- 一个EventLoop在它的整个生命周期中都只会与唯一一个Thread进行绑定。
- 所有由EventLoop所处理的各种I/O事件都将在它所关联的那个Thread上进行处理。
- 一个Channel在它的整个生命周期中只会注册在一个EventLoop上。
- 一个EventLoop在运行过程当中，会被分配给一个或者多个Channel。





## 12.SimpleChannelInboundHandler和ChannelInboundHandlerAdapter的区别？

SimpleChannelInboundHandler在执行完channelRead0方法后会自动释放资源。



## 13.Channel的write和ChannelHandlerContext的write区别？

channel中的write方法会经过所有的handler，而context只会经过从下一个handler开始的handler。



## 14.

- ChannelHandlerContext与ChannelHandler之间的关联绑定关系都不会发生改变的，因此对其进行缓存是没有任何问题的。
- 对于与Channel的同名方法来说，ChannelHandlerContext的方法将会产生更短的事件流，所以我们应该在可能的情况下利用这个特性来提升应用性能。



## 15.netty 缓冲区

#### 15.1 heap buffer

最常见的类型，ByteBuf将数据存储到JVM的堆空间中，并且将实际的数据存放到byte array中来实现。

- 优点：由于数据是存储到JVM的堆中，因此可以快速的创建与释放，并且它提供了直接访问内部字节数组的方法。
- 缺点：每次读写数据时，都需要先将数据复制到直接缓冲区中在进行网络传输（必须通过直接缓冲区进行）。



#### 15.2 direct buffer

在堆之外直接分配内存空间，直接缓冲区并不会占用堆的容量空间，因为它是由操作系统在本地内存进行的数据分配。

- 优点：在使用Socket进行数据传递时，性能非常好，因为数据直接位于操作系统的本地内存中，所以不需要从JVM将数据复制到直接缓冲区中。
- 缺点：因为direct buffer是直接在操作系统内存中，所以内存空间的分配和释放要比堆空间更加复杂，而且速度要慢一些。

> Netty通过提供内存池来解决这个问题。直接缓冲区并不支持通过字节数组的方式来访问数据。

> 重点：对于后端的业务消息的编解码来说，推荐使用HeapByteBuf；对于I/O通信线程在读写缓冲区时，推荐使用DirectByteBuf。

#### 15.3 composite buffer（复合缓冲区）



## 16 JDK的ByteBUffer与Netty的ByteBuf之间的区别

- Netty的ByteBuf采用了读写索引分离的策略（readIndex与writeIndex），一个初始化（里面尚未有任何数据）的ByteBuf的readIndex和writeIndex值都为0.
- 当读索引与写索引处于同一个位置时，如果我们继续读取，那么就会抛出IndexOutOfBoundsException。
- 对于ByteBuf的任何读写操作都会分别单独维护读索引和写索引。



## 17 AtomicIntegerFieldUpdater要点总结：

- 更新器更新的必须是int类型变量，不能是其包装类型。
- 更新器更新的必须是volatile类型变量。确保线程之间共享变量时的立即可见性。
- 变量不能是static的，必须是实例变量。因为Unsafe.objectFieldOffset()方法不支持静态变量（cas操作上是通过对象实例的偏移量来直接进行赋值的。
- 更新器只能修改它可见范围内的变量，因为更新器是通过反射来得到这个变量，如果变量不可见就会报错。



## 18 Netty处理器重要概念：

- Netty的处理器可以分为两类：入站处理器和出站处理器。
- 入站处理器的顶层是ChannelInboundHandler，出站处理器的顶层是ChannelOutboundHandler。
- 数据处理时常用的各种编解码器本质上都是处理器。
- 编解码器：无论我们向网络中写入的数据是什么类型，数据在网络中传输时，其都是以字节流的形式呈现的；将数据由原本的形式转换为字节流的操作称为编码（encode），将数据由字节转换为它原本的格式或者其他格式的操作称为解码（decode），编解码统一称为codec。
- 编码：本质上是一种出站处理器，因此，编码一定是一种ChannelOutboundHandler。
- 解码：本质上是一种入站处理器，因此，解码一定是一种ChannelInboundHandler。
- 在Netty中，编码器通常以XXXEncoder命名，解码器通常以XXXDecoder命名。



## 19 关于Netty编解码器的重要结论

- 编码器其所接收的消息类型必须要与待处理的参数类型一致，否则该编码器不会被执行。
- 在解码器进行数据解码时，一定要记得判断缓冲（ByteBuf）中的数据是否足够，否则将会产生一些问题。















