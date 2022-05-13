xxl-job版本：2.3.0

工作中配置xxl-job遇到的问题：
接口是通过注解的方式写的：

```
@XxlJob("pptvIncrement")
public ReturnT<String> increment(String param){}
```

在配置到xxl-job时，执行任务报错：

```
ERROR [com.xxl.job.core.server.EmbedServer$EmbedHttpServerHandler] - >>>>>>>>>>> xxl-job provider netty_http server caught exception
java.lang.NoSuchMethodError: io.netty.handler.codec.http.FullHttpRequest.uri()Ljava/lang/String;
        at com.xxl.job.core.server.EmbedServer$EmbedHttpServerHandler.channelRead0(EmbedServer.java:156)
        at com.xxl.job.core.server.EmbedServer$EmbedHttpServerHandler.channelRead0(EmbedServer.java:138)
        at io.netty.channel.SimpleChannelInboundHandler.channelRead(SimpleChannelInboundHandler.java:105)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:308)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:294)
        at io.netty.handler.codec.MessageToMessageDecoder.channelRead(MessageToMessageDecoder.java:103)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:308)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:294)
        at io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:244)
        at io.netty.channel.CombinedChannelDuplexHandler.channelRead(CombinedChannelDuplexHandler.java:147)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:308)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:294)
        at io.netty.handler.timeout.IdleStateHandler.channelRead(IdleStateHandler.java:254)
        at io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:308)
        at io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:294)
        at io.netty.channel.DefaultChannelPipeline.fireChannelRead(DefaultChannelPipeline.java:846)
        at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:131)
        at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:511)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:468)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:382)
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:354)
        at io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:111)
        at io.netty.util.concurrent.DefaultThreadFactory$DefaultRunnableDecorator.run(DefaultThreadFactory.java:137)
        at java.lang.Thread.run(Thread.java:745)
```

从网上查了各种解释，最终确定是因为netty版本不一致导致的：
xxl-job配置的netty版本是4.1.48，而我们项目引入的netty版本是4.0.28.Final，所以我们可以采取升级项目的netty版本的方式。

还有一种方式，直接以接口的方式暴露出来，这样接口所在的项目无需引入xxl-job，就不会存在netty冲突的问题了，然后在控制台直接按如下配置即可：

总结：
xxljob的配置还是很灵活的，如果执行的任务需要传参，就可以在任务参数一栏传入自己需要的参数。
————————————————
版权声明：本文为CSDN博主「种下星星的日子」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/hongwei15732623364/article/details/108282159