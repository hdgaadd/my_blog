![6](D:\code\z-mine\my_blog\2024.3.12\6.png)
> *面试官：上一期博客问了你Redis主从，现在考考你Redis集群吧*
>
> <br/>
>
> *面试官：不用慌尽管说，错了也没关系😊。。。*
>
> <br/>
>
> **每日分享大厂面试演练，感兴趣就关注我吧**❤️







## 面试官：知道Redis集群和Redis主从有什么区别

它们两者都是Redis高可用的解决方案，但偏向点不同。

Redis主从对数据的完整性更看重，主、从服务器都**保存完整**的一个数据库；而Redis集群则对抗压能力更看重，整个集群的数据库**整合起来**是一个完整的数据库。要看业务做出不同的选择。

所以在功能上他们也有不同，Redis主从有哨兵，而Redis集群有分片。

![微信截图_20240312153614](D:\code\z-mine\my_blog\2024.3.12\微信截图_20240312153614.png)

## 面试官：那Redis集群怎么实现负载均衡的

主要是通过我刚刚提到的**分片**功能。

集群的话通过分片的方式来保存数据库中的键值对，集群的整个数据库被分为**16384**个槽，而集群中的每个节点可以处理这里面的0个或最多16384个槽。

这样的话，对键值对的增删改查就可以**负载均衡**地下落到各个集群的节点中。

## 面试官：要是热点数据都是某个Redis节点的槽，负载均衡不是没用了吗

噢噢，这种情况的话，Redis集群有**重新分片**的功能。

重新分片操作可以将任意数量已经指派给某个节点的槽改为指派给另一个节点，相关槽所属的键值对也会从源节点被移动到目标节点。

更重要的是，重新分片是可以**在线进行**的，集群不需要下线，源节点和目标节点也都可以继续处理命令请求。

像面试官说的这种情况，可以在线上环境紧急把相关热点槽**指派**给其他节点处理。

![微信截图_20240312153746](D:\code\z-mine\my_blog\2024.3.12\微信截图_20240312153746.png)

## 面试官：还有其他方法吗

emmmm我想想，还可以对这些槽负责的Redis节点配置从服务器。

主节点用于处理槽，而从服务器可以用来分担**读的压力**，如果主节点失效了，从节点会成为主节点代替它继续工作。

![微信截图_20240312153812](D:\code\z-mine\my_blog\2024.3.12\微信截图_20240312153812.png)

## 面试官：集群里没有哨兵，那他们怎么选举主节点的

选举的话是集群中**自带的机制**，不需要哨兵的支持。

不过和**哨兵**选举领头哨兵一样是先到先得，而且他们投票的对象是**集群中的其他节点**。

集群里每个负责处理槽的主节点都有一次投票的机会，而第一个向主节点要求投票的从节点将获得主节点的投票。获得半数支持的从节点就可以成为主节点。

## 面试官：最后问你一个，集群里节点怎么进行故障检测的

**集群节点之间**的故障检测和Redis主从中的**哨兵**检测很类似，都是通过**PING消息**来检测的。。。



> *面试官抓抓脑袋，继续看你的简历......*
>
> <br/>
>
> *得想想考点你不懂的😰*

## 未完待续。。。。。。

好了，今天的分享就先到这，我们下期**大厂面试演练**继续。

> **创作不易，不妨点赞、收藏、关注支持一下，各位的支持就是我创作的最大动力**❤️