## 一、前瞻

经过上篇源码阅读博客的实践，发现按模块阅读也能获得不少收获，而且能更加系统地阅读源码。

今天的阅读方式还是按**模块阅读**的方式，以下是Spring各个模块的组成。

![spring-overview](D:\code\z-mine\my_blog\z-others\spring-overview.png)

那今天就挑**Beans**这个模块来阅读，先思考下本次阅读的**阅读线索**：

1. Beans模块使用了什么**设计模式**
2. Beans模块里的Bean采用什么数据结构进行存储
3. Beans模块里的Bean被Spring IOC容器管理，那管理Bean的**具体实例**是谁

## 二、探索

Ok，先整体看下Beans模块的代码组织结构。

![微信截图_20240326170710](D:\code\z-mine\my_blog\20240326\微信截图_20240326170710.png)

看了组织结构，很好奇为什么spring不多创建几个文件包来分类，这个先作为我们的**阅读线索4**，等下继续探索。

**factory包**很显眼在第一个，应该就是创建Bean的工厂，而且应该使用了不少设计模式，我们按这个为入口进行探索。

![微信截图_20240326171254](D:\code\z-mine\my_blog\20240326\微信截图_20240326171254.png)

我们以**BeanFactory**接口的实现类ClassPathXmlApplicationContext，来看看factory这个工厂是如何运行的，顺便解决我们的阅读线索1、2。

以下是ClassPathXmlApplicationContext的类图。

![微信截图_20240326172028](D:\code\z-mine\my_blog\20240326\微信截图_20240326172028.png)

从类图里的链条可以知道，BeanFactory接口通过一系列的链条，生成了最终的实例ClassPathXmlApplicationContext。

BeanFactory作用在于生产Bean对象，而子类实现ClassPathXmlApplicationContext通过Class Path这种方式来生产Bean对象。

也就是采用了**工厂方法模式**，使每一个不同的子类实现都封装为了一个对象。

到这我们解决了阅读线索1。

> Beans模块使用了什么设计模式

我们再看看阅读线索2：Beans模块里的Bean采用什么数据结构进行存储？

```java
public interface BeanFactory {

	/**
	 * Return an instance, which may be shared or independent, of the specified bean.
	 * <p>This method allows a Spring BeanFactory to be used as a replacement for the
	 * Singleton or Prototype design pattern. Callers may retain references to
	 * returned objects in the case of Singleton beans.
	 * <p>Translates aliases back to the corresponding canonical bean name.
	 * <p>Will ask the parent factory if the bean cannot be found in this factory instance.
	 * @param name the name of the bean to retrieve
	 * @return an instance of the bean.
	 * Note that the return value will never be {@code null} but possibly a stub for
	 * {@code null} returned from a factory method, to be checked via {@code equals(null)}.
	 * Consider using {@link #getBeanProvider(Class)} for resolving optional dependencies.
	 * @throws NoSuchBeanDefinitionException if there is no bean with the specified name
	 * @throws BeansException if the bean could not be obtained
	 */
	Object getBean(String name) throws BeansException;
}
```

BeanFactory主要是提供获取Bean的功能，那存储Bean的应该就在类图里链条的中间。

![微信截图_20240326175627](D:\code\z-mine\my_blog\20240326\微信截图_20240326175627.png)

我们在链条的中间**AbstractApplicationContext**找到了getBen方法的实现，既然可以get，那存储Bean的数据结构也应该在里面。

```java
@Override
public final ConfigurableListableBeanFactory getBeanFactory() {
    DefaultListableBeanFactory beanFactory = this.beanFactory;
    if (beanFactory == null) {
        throw new IllegalStateException("BeanFactory not initialized or already closed - " +
                                        "call 'refresh' before accessing beans via the ApplicationContext");
    }
    return beanFactory;
}
```

最终定位到了这段代码，可以看到引用了DefaultListableBeanFactory类，我们打开这个类。

![微信截图_20240326180140](D:\code\z-mine\my_blog\20240326\微信截图_20240326180140.png)

可以看到存储Bean的**最终数据结构就是这些Map**，还采用了ConcurrentHashMap来支持并发，而Map的Key BeanDefinition就是**Bean本身**。

到这我们就解决了阅读线索2：

> Beans模块里的Bean采用什么数据结构进行存储

而阅读线索3也显而易见，管理Bean的也就是存储Bean的这些对象如上文的**DefaultListableBeanFactory**：

> Beans模块里的Bean被Spring IOC容器管理，那管理Bean的**具体实例**是谁

我们可以全局搜索下private final Map<String, BeanDefinition>，看看哪些在存储Bean对象。

![微信截图_20240326181319](D:\code\z-mine\my_blog\20240326\微信截图_20240326181319.png)

可以看到有两个，其他一个便是我们上文所探索的**DefaultListableBeanFactory**。



## 三、总结

我们再来看看中间新加入的阅读线索4，不知大家忘记了没。

我们可以对照图片1的代码组织结构，发现这些没包的类都是比较杂的，想必是Spring觉得目前这些功能类还不成一个包的体系，可能后面规模更大会统一集成起来管理。