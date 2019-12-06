## 1.ChannelInitializer
ChannelInitializer是一种特殊的ChannelInboundHandler，可以通过一种简单的方式（调用initChannel方法）来初始化Channel。

> 注意：当initChannel被执行完后，会将当前的handler从Pipeline中移除。

## 2.执行流程
![执行流程](https://upload-images.jianshu.io/upload_images/16503287-711176974445f461.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
