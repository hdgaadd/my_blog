> 相信大家碰到源码时经常无从下手🙃，不知道从哪开始阅读，面对大量代码晕头转向，索性就读不下去了，又浪费了一次提升自己的机会😭。
>
> <br/>
>
> 我认为**有一种方法**，可以解决大家的困扰！**那就是通过阅读某一次开源的【PR】、【ISSUE】，从这个入口出发去阅读源码！！**
>
> <br/>
>
> 至此，我们发现自己开始从大量堆砌的源码中脱离开来😀。豁然开朗，柳暗花明又一村。

ShenYu是一个异步的，高性能的，跨语言的，响应式的 API 网关。有关ShenYu的介绍可以[戳这](https://shenyu.apache.org/zh/docs/index/)。



## 一、前瞻

OK，开始我们今天的[PR阅读](https://github.com/apache/shenyu/pull/5003)，PR提交如下图。

![Snipaste_2024-02-29_21-08-49](E:\z-relax\my_blog\2024.2.29\Snipaste_2024-02-29_21-08-49.png)

翻译过来，意思就是：原来的插件只支持 motan2 协议，并且是硬编码的。本次修改使MotanRpcExt 得到增强以支持**motan本机协议**。同时支持根据客户端注册的RPC协议自动匹配。

简单来说就是增强motan插件。

我们可以通过以上的线索来思考我们本次的阅读线索：

1. 贡献者是**做了什么**实现了增强motan插件
2. 这个motan的插件的功能是什么



## 二、探索

对于线索2，我们可以看下ShenYu项目的模块。

![Snipaste_2024-02-29_21-17-06](E:\z-relax\my_blog\2024.2.29\Snipaste_2024-02-29_21-17-06.png)

ShenYu既然是一个网关，它必须要**调用到微服务节点**，而上图中有各个微服务开源中间件的集成，grpc、dubbo等。可以知道motan插件的功能，就是提供给ShenYu网关一个**支持调用motan微服务节点**的功能。

那我们继续线索1的探索。

整体看下本次的所有提交，可以分为**两部分**，负责监听微服务节点消息的监听端、调用微服务节点的调用端。

![Snipaste_2024-02-29_21-49-15](E:\z-relax\my_blog\2024.2.29\Snipaste_2024-02-29_21-49-15.png)

那我们先看看调用端修改了什么？

```java
// 旧代码
try {
    responseFuture = (ResponseFuture) commonClient.asyncCall(metaData.getMethodName(), params, Object.class);
} catch (Throwable e) {
    LOG.error("Exception caught in MotanProxyService#genericInvoker.", e);
    return null;
}

// 新代码
try {
    Request request = MotanClientUtil.buildRequest(reference.getServiceInterface(), metaData.getMethodName(), metaData.getParameterTypes(), params, null);
    responseFuture = (ResponseFuture)commonClient.asyncCall(request, Object.class);
} catch (Throwable e) {
    LOG.error("Exception caught in MotanProxyService#genericInvoker.", e);
    return null;
}
```

很明显，新代码**新增了几个入参**：`reference.getServiceInterface()`、`metaData.getParameterTypes()`，大概是为了支持增加motan的功能，这一部分我们没什么发现，那先看看监听端。

监听端做了以下修改。和调用端同样，**新增了入参**。

![Snipaste_2024-02-29_22-15-26](E:\z-relax\my_blog\2024.2.29\Snipaste_2024-02-29_22-15-26.png)

其实可以思考下，本次PR提交是为了**支持motan协议**，而该协议相比旧协议motan2需要**新增以上的入参**，以支持其的功能。

这也解答了上文我们思考的线索1：

> 贡献者是做了什么实现了增强motan插件





好了，今天的分享就到这了。**大家能否感受到通过PR这种方式来阅读源码的乐趣呢** ！

> 博主的GitHub也有一些读者感兴趣的知识可以学习😍，[GitHub地址戳这](https://github.com/hdgaadd)
>
> <br/>
>
> **创作不易，不妨点赞、收藏、关注支持一下，各位的支持就是我创作的最大动力**❤️





