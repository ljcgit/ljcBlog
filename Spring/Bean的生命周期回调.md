# Bean的生命周期回调

​     Bean的生命周期回调主要是在Bean对象创建和销毁的过程中会执行一些init和destroy方法，了解不同形式的init和destroy声明方法，以及它们之间的先后执行顺序会有助于我们理解Bean的生命周期过程。下文主要介绍了生命周期回调的各种使用方式，以及在Spring中是如何实现的。



**常用的三种形式：**

- `InitializingBean` 和` DisposableBean`接口，分别实现`afterPropertiesSet` 和 `destroy`方法；
- JSR-250下的`@PostConstruct` 和 `@PreDestroy`注解；
- 在Spring xml文件中声明`init-method`  和 `destroy-method` 属性。



> 不推荐使用InitializingBean、DisposableBean接口，因为可能会导致和Spring高耦和。




多种形式共同作用于一个Bean时，按照如下顺序执行：

- @PostConstruct注解指定的方法；
- InitializingBean接口实现的afterPropertiesSet()方法；
- init-method属性指定的方法；
- @PreDestroy注解指定的方法；
- DisposableBean接口实现的destroy方法；
- destroy-method属性指定的方法。



示例：

```java
public class X implements InitializingBean, DisposableBean {

    @PostConstruct
    public void postConstruct() {
        System.out.println("PostConstruct.....");
    }

    public X() {
        System.out.println("X construct...");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("PreDestroy.....");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("afterPropertiesSet...");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("destroy...");
    }

    public void init() {
        System.out.println("init...");
    }

    public void destroyMethod() {
        System.out.println("destroyMethod...");
    }
}
```

输出顺序：

> X construct...
> PostConstruct.....
> afterPropertiesSet...
> init...
> PreDestroy.....
> destroy...
> destroyMethod...



源码实现方法如下，这里只罗列了相关方法中的部分代码：

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
		throws BeanCreationException {

	// 这个beanWrapper是用来持有创建出来的bean对象的
	BeanWrapper instanceWrapper = null;

	if (instanceWrapper == null) {
		// 没有就创建实例, 这里一般通过newInstance创建对象
		instanceWrapper = createBeanInstance(beanName, mbd, args);
	}

	Object exposedObject = bean;
	try {
    // 进行对象属性的渲染
		populateBean(beanName, mbd, instanceWrapper);
		// 初始化对象
		exposedObject = initializeBean(beanName, exposedObject, mbd);
	}
	catch (Throwable ex) {
		if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
			throw (BeanCreationException) ex;
		}
		else {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
		}
	}

	return exposedObject;
}
```

​     可以从上面看出首先进行的是对象的实例化操作`createBeanInstance`，相当于new生成了一个对象，然后在`populateBean`中进行对象属性的渲染，包括属性的依赖注入等都是在该方法中执行的，接着就是执行了`initializeBean`方法来初始化对象。

​	initializeBean方法内部主要代码如下：

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
  Object wrappedBean = bean;
  if (mbd == null || !mbd.isSynthetic()) {
     // 1 @PostConstruct注解所对应的方法会这里进行调用
     wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
  }
  try {
     // 2-3 调用bean的afterPropertiesSet方法和自定义init方法
     invokeInitMethods(beanName, wrappedBean, mbd);
  }
  catch (Throwable ex) {
  }
  //如果mbd为null || mbd不是"synthetic"
  if (mbd == null || !mbd.isSynthetic()) {
     wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
  }
  return wrappedBean;
 }
```



在initializeBean方法中，首先会遍历所有的BeanPostProcessor，执行对应的`postProcessBeforeInitialization`方法；

```java
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
      throws BeansException {

   Object result = existingBean;
   for (BeanPostProcessor processor : getBeanPostProcessors()) {
      Object current = processor.postProcessBeforeInitialization(result, beanName);
      if (current == null) {
         return result;
      }
      result = current;
   }
   return result;
}
```

默认的`postProcessBeforeInitialization`方法是直接返回原始bean对象，不做任何处理；

当执行到`CommonAnnotationBeanPostProcessor`处理器时，会扫描到`@PostConstruct`注解并执行相应的方法；

```java
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
   LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
   try {
      metadata.invokeInitMethods(bean, beanName);
   }
   catch (InvocationTargetException ex) {
      throw new BeanCreationException(beanName, "Invocation of init method failed", ex.getTargetException());
   }
   catch (Throwable ex) {
      throw new BeanCreationException(beanName, "Failed to invoke init method", ex);
   }
   return bean;
}
```



> 这里输出PostConstruct.....



initializeBean方法中后续会执行一个invokeInitMethods来执行init相关的方法；

```java
protected void invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
   throws Throwable {
  boolean isInitializingBean = (bean instanceof InitializingBean);
  if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
   // 如果安全管理器不为null
   if (System.getSecurityManager() != null) {
   }
   else {
    // 调用bean的afterPropertiesSet方法
    ((InitializingBean) bean).afterPropertiesSet();
   }
  }
  if (mbd != null && bean.getClass() != NullBean.class) {
   String initMethodName = mbd.getInitMethodName();
   if (StringUtils.hasLength(initMethodName) &&
     !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
     !mbd.isExternallyManagedInitMethod(initMethodName)) {
    // 在bean上调用指定的自定义init方法
    invokeCustomInitMethod(beanName, bean, mbd);
   }
  }
 }
```



按照顺序执行，就能看出这里会先执行afterPropertiesSet方法，然后是自定义的init方法。

> 执行完该方法会输出：
>
> afterPropertiesSet...
> init...



对象的销毁会进入destory方法，主要代码如下；

```java
public void destroy() {
	if (!CollectionUtils.isEmpty(this.beanPostProcessors)) {
		// 调用@PreDestroy方法
		for (DestructionAwareBeanPostProcessor processor : this.beanPostProcessors) {
			processor.postProcessBeforeDestruction(this.bean, this.beanName);
		}
	}

	if (this.invokeDisposableBean) {

		try {
			if (System.getSecurityManager() != null) {

			}
			else {
				// 执行Disposable Bean接口的destroy方法
				((DisposableBean) this.bean).destroy();
			}
		}
		catch (Throwable ex) {

		}
	}

	if (this.destroyMethod != null) {
		// 执行destroy-method方法
		invokeCustomDestroyMethod(this.destroyMethod);
	}

}
```



可以明显的看出执行顺序以及方式和初始化都是差不多的。

> 执行完destroy方法，会依次输出：
>
> PreDestroy.....
> destroy...
> destroyMethod...
