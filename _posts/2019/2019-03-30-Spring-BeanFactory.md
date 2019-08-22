---
layout: post
title: Spring Framework——BeanFatorty解析
category: Spring
tags: [Spring Framework]
---
透过源码了解容器，解惑Spring的依赖注入

## Spring Framework -- BeanFatorty
Spring 的 BeanFactory 容器
这是一个最简单的容器，它主要的功能是为依赖注入 （DI） 提供支持，这个容器接口在 org.springframework.beans.factory.BeanFactor 中被定义。BeanFactory 和相关的接口，比如BeanFactoryAware、DisposableBean、InitializingBean，仍旧保留在 Spring 中，主要目的是向后兼容已经存在的和那些 Spring 整合在一起的第三方框架。

在 Spring 中，有大量对 BeanFactory 接口的实现。其中，最常被使用的是 XmlBeanFactory 类。这个容器从一个 XML 文件中读取配置元数据，由这些元数据来生成一个被配置化的系统或者应用。

在资源宝贵的移动设备或者基于 applet 的应用当中， BeanFactory 会被优先选择。否则，一般使用的是 ApplicationContext，除非你有更好的理由选择 BeanFactory。

## getBean源码解析

### BeanFactory

``` java
public interface BeanFactory {

  Object getBean(String name) throws BeansException;

  <T> T getBean(String name, Class<T> requiredType) throws BeansException;

  Object getBean(String name, Object... args) throws BeansException;

  <T> T getBean(Class<T> requiredType) throws BeansException;

  <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

  <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);

  <T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);
  /** 工厂是否包含指定name的实例 */
  boolean containsBean(String name);
  /** 指定name的实例对象是否是单例模式的 */
  boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
  /** 指定name的实例对象是否是原型模式的 */
  boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
  /** 实例对象的类型是否匹配 */
  boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
  /** 实例对象的类型是否匹配 */
  boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
  /** 获取实例对象类型 */
  Class<?> getType(String name) throws NoSuchBeanDefinitionException;
  /** 获取别名，如果有设置别名的话 */
  String[] getAliases(String name);

}
```

从接口中的方法名可以很容易的理解每个方法的意图，最常用或者最常见的就是getBean方法，用于获取Bean的实例。BeanFactory是用于访问Spring Bean容器的根接口，是一个单纯的Bean工厂，也就是常说的IOC容器的顶层定义，各种IOC容器是在其基础上为了满足不同需求而扩展的，包括经常使用的ApplicationContext。

getBeanProvider: 返回指定bean的提供程序,从5.1开始，此接口扩展Iterable并提供Stream支持

返回ObjectProvider，提供可用性和唯一性检查

### ObjectProvider
``` java
public interface ObjectProvider<T> extends ObjectFactory<T>, Iterable<T> {

T	getIfAvailable() ;//检查可用性性，返回bean实例，不可用返回null
default T		getIfAvailable(Supplier<T> defaultSupplier); //检查可用性，如果工厂中不存在，提供一个默认回调 defaultSupplier
T	getIfUnique();//检查唯一性，返回bean实例，如果不唯一或不可用返回null
default T	getIfUnique(Supplier<T> defaultSupplier);//检查唯一性，如果没有找到唯一项，则返回默认回调提供的默认对象

...
}

```

### AbstractBeanFactory
getBean：获取bean实例，在日常使用中，通过setter方法或者注解方式就能从容器中取得对象并注入
我通过观察BeanFactory的骨架实现AbstractBeanFactory来了解容器的实例获取过程

``` java
@Override
	public Object getBean(String name) throws BeansException {
		return doGetBean(name, null, null, false);
	}
```

``` java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
		@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
    // 取得bean的name，可通过别名获取实际beanName,aliasMap.get(name)
    final String beanName = transformedBeanName(name);
    Object bean;
    // 调用父类DefaultSingletonBeanRegistry，从单例缓存中获取对象实例（请见下方DefaultSingletonBeanRegistry类）
    Object sharedInstance = getSingleton(beanName);
    // check对象是否为null
    if (sharedInstance != null && args == null) {
      ...
      bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }
    else {
      // 若从当前容器中取不到已经被实例化的bean，判断bean是否在父容器中定义，若在，则从父容器中取得实例。
  		// 循环依赖检测
      if (isPrototypeCurrentlyInCreation(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
      }
      // 从父容器中取得实例。
      ...
    }

    // Check if required type matches the type of the actual bean instance.
    if (requiredType != null && !requiredType.isInstance(bean)) {
      ...
    }
    return (T) bean;
}
```

### DefaultSingletonBeanRegistry
``` java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
  /** 单例对象缓存，{name:instance} */
  private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

  /** 正在创建的对象集合 */
  private final Set<String> singletonsCurrentlyInCreation =
			Collections.newSetFromMap(new ConcurrentHashMap<>(16));

  /** 早期的缓存对象 */
  private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

  /** 单例工厂缓存 */
  private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);


  @Override
	@Nullable
  public Object getSingleton(String beanName) {
    return getSingleton(beanName, true);
  }


  @Nullable
  protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 从缓存中获取实例对象
    Object singletonObject = this.singletonObjects.get(beanName);

    // 如果对象为空，但却在正在创建的集合中
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
      // 锁住整个缓存对象
      synchronized (this.singletonObjects) {
        // 从早起缓存中获取
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
          // 从单例工厂缓存取得
          ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);

          if (singletonFactory != null) {
            // 通过工厂创建对象
            singletonObject = singletonFactory.getObject();
            // 放入早期对象缓存中
            this.earlySingletonObjects.put(beanName, singletonObject);
            // 删除该name对应的对象工厂
            this.singletonFactories.remove(beanName);
          }
        }
      }
    }
    return singletonObject;
  }

}
```
