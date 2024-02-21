> 相信我们大家碰到源码一开始都是比较无从下手的🙃，不知道从哪开始阅读、阅读了能收获什么。我认为**有一种方法**，可以解决上面的问题，让我们既知道了从哪开始阅读源码，又能够从阅读源码中获得收获！

> 那就是通过阅读某一次**开源commit**、某一次**ISSUE的提出**，从这个入口出发，去探索源码。

> 通过上面的方法，我们可以从堆砌的源码中脱离开来，如脱缰野马开心地去阅读其中让我们最有兴趣的部分，都说兴趣是最好的老师👩‍🏫，兴趣老师会引导我们走向正确的道路，让我们获取我们需要的收获，最后甚至还会送给你一朵小红花🌹当作奖励🤭。

## 前瞻

那就让我们从这次开源提交开始：[commit链接](https://github.com/apache/shenyu/pull/3134)

![微信截图_20240221104153](D:\code\z-mine\my_blog\2024.2.21\微信截图_20240221104153.png)

翻译过来就是

> 优化同步数据时数据初始化的代码。
> 如果数据同步需要数据初始化，则实现`DataChangedInit`.

大致意思就是实现**DataChangedInit**去优化原先的数据初始化，让我们看看怎么个优化法呢，DataChangedInit是怎么起到作用的！

## 探索

```java
public interface DataChangedInit extends CommandLineRunner {

}
```

可以看到底层接口**DataChangedInit**实现了**CommandLineRunner**，顾名思义作用是在项目启动时执行。

![微信截图_20240221112434](D:\code\z-mine\my_blog\2024.2.21\微信截图_20240221112434.png)

看下父类的实现。可以看到执行的时候判断**notExist**方法，若notExist为false的话就执行syncDataService的**异步代码**。可以猜到上图的各个子类实现主要就是实现**notExist**方法来判断是否使用**同步代码**。我们注意下**SyncDataService**后面需要提及到。

```java
public abstract class AbstractDataChangedInit implements DataChangedInit {

    /**
     * SyncDataService, sync all data.
     */
    @Resource
    private SyncDataService syncDataService;

    @Override
    public void run(final String... args) throws Exception {
        if (notExist()) {
            syncDataService.syncAll(DataEventTypeEnum.REFRESH);
        }
    }

    /**
     * check exist.
     *
     * @return boolean.
     */
    protected abstract boolean notExist();
}
```

子类代码：

```java
public class ZookeeperDataChangedInit extends AbstractDataChangedInit {

    private final ZkClient zkClient;

    /**
     * Instantiates a new Zookeeper data changed init.
     *
     * @param zkClient        the zk client
     */
    public ZookeeperDataChangedInit(final ZkClient zkClient) {
        this.zkClient = zkClient;
    }

    @Override
    protected boolean notExist() {
        return !zkClient.exists(DefaultPathConstants.PLUGIN_PARENT)
                && !zkClient.exists(DefaultPathConstants.APP_AUTH_PARENT)
                && !zkClient.exists(DefaultPathConstants.META_DATA);
    }
}
```

## 总结

那我们好奇本次优化的点究竟是什么呢，我们看下**旧代码的实现**可以找到答案，其实本次优化是为了后续可扩展性，把**重复的代码抽象起来**。

![微信截图_20240221113427](D:\code\z-mine\my_blog\2024.2.21\微信截图_20240221113427.png)

可以看到ZooKeeper、Nacos、Concus的数据同步都有重复的入参**SyncDataService**，且这3个实现类实现**相同的功能**竟然没有一个父类来承接。

我们回顾AbstractDataChangedInit提到的**SyncDataService**，这步就是一个优化了，把重复的SyncDataService入参抽象到父类AbstractDataChangedInit，同时让实现相关功能的类都有相同的父类**DataChangedInit**来进行归类。



