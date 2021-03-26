

# Spring —— 循环依赖

Spring Bean默认都是以单例的形式存在，但是也支持prototype形式，这里主要介绍单例Bean。



## 示例

```java
@Component
public class X {

    @Autowired
    private Y y;

    public X() {
        System.out.println("X construct...");
    }
}
```



```java
@Component
public class Y {

    @Autowired
    private X x;

    public Y() {
        System.out.println("Y construct...");
    }
}
```



```java
@Configuration
@ComponentScan("com.ljc")
public class AppConfig {
}
```



```java
public class Test {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        context.getBean(X.class);

        // 优雅关闭，SpringWEB中已有相关实现
        context.registerShutdownHook();
    }
```



## Bean循环依赖过程

`getSingleton`方法时获取单例Bean的方法，其中包含了对象正在创建的标记、对象的创建和初始化、对象放入单例缓冲几个重要步骤。

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {

	// 使用单例对象的高速缓存Map作为锁，保证线程同步
	synchronized (this.singletonObjects) {
		// 从单例对象的高速缓存Map中获取beanName对应的单例对象
		Object singletonObject = this.singletonObjects.get(beanName);
		// 如果单例对象获取不到
		if (singletonObject == null) {

			// 循环依赖提前标记
			beforeSingletonCreation(beanName);

			boolean newSingleton = false;

			try {
				// 从单例工厂中获取对象，如果还未创建，会创建一个实例
				singletonObject = singletonFactory.getObject();
				// 生成了新的单例对象的标记为true，表示生成了新的单例对象
				newSingleton = true;
			}

			if (newSingleton) {
				// 放生成的bean放入单例缓存中
				addSingleton(beanName, singletonObject);
			}
		}
		// 返回该单例对象
		return singletonObject;
	}
}
```



`beforeSingletonCreation(beanName)`会在实例化对象前将beanName放入到一个Set集合中，表明当前bean处于创建过程。

接着就是在执行`singletonFactory.getObject();`方法时，如果不存在对象就会创建一个对象。

下面是创建Bean的最主要方法，这里只罗列的部分较为重要的代码：

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
		throws BeanCreationException {

	BeanWrapper instanceWrapper = null;
	// 获取factoryBean实例缓存
	if (mbd.isSingleton()) {

		instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
	}
	if (instanceWrapper == null) {
		// 没有就创建实例, 这里通过newInstance创建对象
		instanceWrapper = createBeanInstance(beanName, mbd, args);
	}

	// 判断当前bean是否需要提前曝光：单例&允许循环依赖&当前bean正在创建中，检测循环依赖
	boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
			isSingletonCurrentlyInCreation(beanName));
	if (earlySingletonExposure) {
		if (logger.isTraceEnabled()) {
			logger.trace("Eagerly caching bean '" + beanName +
					"' to allow for resolving potential circular references");
		}
		// 为避免后期循环依赖，可以在bean初始化完成前将创建实例的ObjectFactory加入工厂
		addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
	}

	Object exposedObject = bean;
	try {
		// 属性渲染
		populateBean(beanName, mbd, instanceWrapper);

		// 执行初始化init方法
		exposedObject = initializeBean(beanName, exposedObject, mbd);
	}
	catch (Throwable ex) {

	}

}
```

当从`factoryBeanInstanceCache`缓存中取不到相关的Bean时，就会通过`createBeanInstance`方法创建一个Object对象（这里还不能说是一个Bean对象，因为只是通过反射实例化出了一个对象，并没有进行过属性渲染，执行初始化方法等步骤）；</br>

后面根据isSingleton() && allowCircularReferences &&isSingletonCurrentlyInCreation(beanName))三个boolean值来判断是否开启循环依赖，其中`isSingleton()` 和 `isSingletonCurrentlyInCreation(beanName))`在单例对象中都是会返回true，所以一般可以通过修改`allowCircularReferences`的值来设置是否开启循环依赖，而默认是为true，即支持循环依赖。</br>

当earlySingletonExposure为true时执行下面方法



> ​		addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
   Assert.notNull(singletonFactory, "Singleton factory must not be null");
   synchronized (this.singletonObjects) {
      // 如果单例对象的高速缓存【beam名称-bean实例】没有beanName的对象
      if (!this.singletonObjects.containsKey(beanName)) {
         // 将beanName,singletonFactory放到单例工厂的缓存【bean名称 - ObjectFactory】
         this.singletonFactories.put(beanName, singletonFactory);
         // 从早期单例对象的高速缓存【bean名称-bean实例】 移除beanName的相关缓存对象
         this.earlySingletonObjects.remove(beanName);
         // 将beanName添加已注册的单例集中
         this.registeredSingletons.add(beanName);
      }
   }
}
```

该方法会将一个`singletonFactory`放入到`singletonFactories`Map中，然后从earlySingletonObjects删除。这里就是常说的对象的**提前暴露**。



> 这个**singletonFactory**其实就是之前实例化生成的对象经过几层包装生成的ObjectFactory对象，这里之所以不直接将生成的bean对象放入到Map中，是因为该对象还不是最终的Bean，后续还要经历一些生命周期，例如AOP增强，最终生成一个代理对象。



添加到Map后就会调用`populateBean`方法，这里就是产生依赖注入环节的地方。🤤🤤🤤🤤



```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {

	boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();

	boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

	PropertyDescriptor[] filteredPds = null;

	if (hasInstAwareBpps) {
		if (pvs == null) {

			pvs = mbd.getPropertyValues();
		}
		for (BeanPostProcessor bp : getBeanPostProcessors()) {

			if (bp instanceof InstantiationAwareBeanPostProcessor) {

				InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;

				// 这里注入了依赖的对象
				PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
				
				if (pvsToUse == null) {

					if (filteredPds == null) {

						filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
					}

					pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);

					if (pvsToUse == null) {
						return;
					}
				}
				pvs = pvsToUse;
			}
		}
	}

}
```

方法中两次遍历了BeanPostProcessor集合，第二次遍历时`AnnotationBeanPostProcessor`处理器执行了postProcessProperties，获取到了所有通过Autowired注解标记的类，然后调用inject方法完成属性的注入。

下图展示了获取到了类中依赖的Y对象：

![依赖注入](/images/image-20210326085519511.png)



这时就开始走创建Y Bean对象的流程；同样会走上面的几个步骤到达`populateBean`方法，检索到有X对象需要注入，还是调用inject方法获取的X对象；

只不过这时通过`getSingleton`就能直接获取到X对象。

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
	// 先去单例缓存中获取单例对象
	Object singletonObject = this.singletonObjects.get(beanName);
	// 如果单例对象缓存中没有，并且该beanName对应的单例bean正在创建中
	if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
		// 再去三级缓存中获取，三级缓存中存储的都是已经newInstance过的，但是还未进行属性填充的
		singletonObject = this.earlySingletonObjects.get(beanName);
		// 如果在三级缓存中也没有，并且允许创建早期单例对象引用
		if (singletonObject == null && allowEarlyReference) {
			synchronized (this.singletonObjects) {
				
				singletonObject = this.singletonObjects.get(beanName);
				if (singletonObject == null) {
					singletonObject = this.earlySingletonObjects.get(beanName);
					if (singletonObject == null) {
						// 当某些方法需要提前初始化的时候则会调用addSingletonFactory方法将对应的ObjectFactory初始化策略存储在singletonFactories
						ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
						if (singletonFactory != null) {
							// 如果存在单例对象工厂，则通过工厂创建一个单例对象
							singletonObject = singletonFactory.getObject();
							// 记录在缓存中，二级缓存和三级缓存的对象不能同时存在
							this.earlySingletonObjects.put(beanName, singletonObject);
							// 从二级缓存中移除
							this.singletonFactories.remove(beanName);
						}
					}
				}
			}
		}
	}
	return singletonObject;
}
```



上面的方法是获取Bean时所会执行的，大部分**allowEarlyReference**的值是为true。

`getSingleton`方法在循环依赖对象创建时会多次调用。Bean对象创建第一次调用时，`getSingleton`方法由于单例缓存中不存在，并且 `isSingletonCurrentlyInCreation`  集合中还没有对应的beanName，所以会返回null；当依赖的对象属性注入当前对象的时候（X—> Y —>X)，相当于于同一个beanName的对象第二次执行了`getSingleton`方法，此时单例缓存中还是不存在该类的单例，但是`isSingletonCurrentlyInCreation`集合中已存在对应的beanName，对象已经处于创建流程中了，所以就可以通过之前提前暴露的ObjectFactory来获取已经实例化（还未属性注入完成）的对象，并放入到早期单例缓存（`earlySingletonObjects`）中，返回ObjectFactory中的Object。



## 三级缓存的作用

```java
   // 单例缓存 一级缓存
   public final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

   // ObjectFactory缓存 二级缓存
   public final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

	 // 早期单例缓存 三级缓存
   public final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);
```



- 一级缓存：主要是存储了bean 单例，是经过一系列加工完成的Bean；
- 二级缓存：存放了实例化对象封装后的ObjectFactory对象，还未进行属性注入，用来暴露给相关的依赖对象；
- 三级缓存：早期的单例对象，还未完成完整的属性注入等。





## Spring循环依赖不适用场景

- `prototype`之间的循环依赖

```java
@Component
@Scope("prototype")
public class X {

    @Autowired
    private Y y;
}
```



```java
@Component
@Scope("prototype")
public class Y {

    @Autowired
    private X x;

    public Y() {
        System.out.println("Y construct...");
    }
}
```



两个prototype类型的对象循环依赖就会导致下面的情况，无限反复依赖下去；

> X -> Y
>
> ​          -> X
>
> ​                  ->Y
>
> ​                         ->X

最后控制台就会产生BeanCurrentlyInCreationException异常信息。

![循环依赖异常](/images/image-20210326135115120.png)



- 通过构造函数注入

这种肯定也是不行的。在属性循环注入的时候，对象还不是一个完整的Bean，而构造器注入则要求必须是一个完整的Bean。
