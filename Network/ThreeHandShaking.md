+ 客户端会先服务端发送一个SYN包，同时会指出客户端序列号；
+ 服务端在收到客户端的包后，会向客户端发送一个ACK包，其值就是客户端的序列号加1，同时会携带SYN信号，指出服务端的序列号；
+ 客户端收到报文后，将服务端的序列号加1作为ACK的值返回给服务端。

![三次握手.png](https://upload-images.jianshu.io/upload_images/16503287-0bae94c2c9a4e336.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### SYN超时问题
如果客户端大量在发送SYN后就关闭客户端，就会导致服务端在较长时间内处于等待状态。
1.SYN队列满后，通过tcp_syncookies参数回发SYN_Cookie；
2.若为正常连接则Client会回发SYN_Cookie，直接建立连接。
