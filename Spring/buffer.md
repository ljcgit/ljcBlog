# Spring为什么需要使用三级缓存？



有如下一个场景，X类中注入了Y类，Y类注入了X类，按照正常的类加载顺序，X类在注入Y的时候其实还没完全初始化完成，而注入的Y此时就开始实例化，发现也需要用到X，这是就去获取X，但是X并没有初始化完成，这个时候就会出现X在等Y初始化完成，而Y又在等X初始化完成，就进入了一个互相等待的情况。



Spring是如何避免这种情况的呢？那就是**三级缓存**。



先来说说Spring有哪三级缓存。

+ 一级缓存 singletonObject  ： 单例对象集合；
+ 二级缓存 earlySingletonObjects ：早期单例对象集合，也就是对象还没有完全初始化；
+ 三级缓存 singletonFactories ：单例对象工厂集合，里面存储的是对象封装后的ObjectFactory。



**Spring实现流程：**

1. X实例化完成；
2. 检测到支持循环依赖，将X实例化的对象封装成ObjectFactory放入三级缓存中；
3. populateBean渲染X对象，对属性Y进行注入；
4. 获取Y对象，getSingleton返回null；
5. 对Y进行实例化；
6. 将Y实例化后的对象封装成ObjectFactory放入三级缓存；
7. populateBean渲染Y对象，对属性X进行注入；
8. 获取X对象，getSingleton返回之前三级缓存中存储的对象，并将对象放入二级缓存；
9. 返回二级缓存中的X；
10. Y初始化完成，返回Y；
11. X初始化完成。



## 为什么一定要使用三级缓存（singletonFactories）？

可以思考下，如果只有两级缓存，在进行对象注入的时候，给属性注入的就是原始对象。但是，在AOP场景下，对象可能是一个代理对象，这个时候注入就应该是一个代理对象而不是原始对象。

三级缓存也就是这个原因存储的是一个ObjectFactory，而不是Object；

当循环依赖过程中，对象第二次进入getBean流程时就需要用到三级缓存中对象，就会调用getObject方法返回一个封装后的对象，而对象既可能是原始对象，也可能是代理对象，再将这个对象返回给需要注入的类。

getObject会对对象进行一系列的包装然后返回。Spring中利用lambda表达式的形式声明，具体实现如下：

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
   // 默认最终公开的对象是bean,通过createBeanInstance创建出来的普通对象
   Object exposedObject = bean;
   // mbd的systhetic属性：设置此bean定义是否是"synthetic"，一般是指只有AOP相关的pointCut配置或者Advice配置才会将 synthetic设置为true
   // 如果mdb不是synthetic且此工厂拥有InstantiationAwareBeanPostProcessor
   if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
      // 遍历工厂内的所有后处理器
      for (BeanPostProcessor bp : getBeanPostProcessors()) {
         // 如果bp是SmartInstantiationAwareBeanPostProcessor实例
         if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
            SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
            // 让exposedObject经过每个SmartInstantiationAwareBeanPostProcessor的包装
            exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
         }
      }
   }
   // 返回最终经过层次包装后的对象
   return exposedObject;
}
```

Spring调用注册的BeanPostProcessor的**getEarlyBeanReference**方法来对对象进行包装，最后返回经过包装的对象。

`AnnotationAwareAspectJAutoProxyCreator`处理器是Spring用来进行AOP处理的，继承了**SmartInstantiationAwareBeanPostProcessor**类，**getEarlyBeanReference**方法如下：

```java
public Object getEarlyBeanReference(Object bean, String beanName) {
   Object cacheKey = getCacheKey(bean.getClass(), beanName);
   this.earlyProxyReferences.put(cacheKey, bean);
   return wrapIfNecessary(bean, beanName, cacheKey);
}
```



```java
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
   // 如果已经处理过，直接返回
   if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
      return bean;
   }
   // 这里advisedBeans缓存了已经进行了代理的bean，如果缓存中存在，则可以直接返回
   if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
      return bean;
   }
   // 这里isInfrastructureClass()用于判断当前bean是否为Spring系统自带的bean，自带的bean是
   // 不用进行代理的；shouldSkip()则用于判断当前bean是否应该被略过
   if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
      // 对当前bean进行缓存
      this.advisedBeans.put(cacheKey, Boolean.FALSE);
      return bean;
   }

   // 获取符合对应类的Advisor，如果没有类没有符合的aop就会返回空集合
   Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
   // 对当前bean的代理状态进行缓存
   // 生成对应的代理对象
   if (specificInterceptors != DO_NOT_PROXY) {
      // 对当前bean的代理状态进行缓存
      this.advisedBeans.put(cacheKey, Boolean.TRUE);
      // 根据获取到的Advices和Advisors为当前bean生成代理对象
      Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
      // 缓存生成的代理bean的类型，并且返回生成的代理bean
      this.proxyTypes.put(cacheKey, proxy.getClass());
      return proxy;
   }

   // 正常无需代理的对象直接返回bean
   this.advisedBeans.put(cacheKey, Boolean.FALSE);
   return bean;
}
```

wrapIfNecessary方法和普通AOP在初始化方法中执行**postProcessAfterInitialization**是一样的，会先获取到类对应的Advisor，然后根据策略生成对应的代理工厂和代理类，当没有Advisor时，也就不会进行代理处理。

> 当有AOP场景下的类具有循环依赖的情况时，就会将代理对象的生成提前到初始化之前，对象注入的时候。



**PS ： AOP处理器postProcessAfterInitialization实现**

```java
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
   if (bean != null) {
      // 获取当前bean的key：如果beanName不为空，则以beanName为key，如果为FactoryBean类型，
      // 前面还会添加&符号，如果beanName为空，则以当前bean对应的class为key
      Object cacheKey = getCacheKey(bean.getClass(), beanName);
      // 判断当前bean是否正在被代理，如果正在被代理则不进行封装
      if (this.earlyProxyReferences.remove(cacheKey) != bean) {
         // 如果它需要被代理，则需要封装指定的bean
         return wrapIfNecessary(bean, beanName, cacheKey);
      }
   }
   return bean;
}
```





