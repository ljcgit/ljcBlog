# 重识Spring的BeanPostProcessor处理器



`BeanPostProcessor`是Spring用来实现快速拓展最为核心的接口，现在常说的AOP就是通过实现BeanPostProcessor接口来进行拓展。

下面就先看下BeanPostProcessor到底长什么样？🧐



## BeanPostProcessor接口

```java
public interface BeanPostProcessor {

	 //在bean init之前执行
   @Nullable
   default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      return bean;
   }

   // 在bean执行完init方法之后执行
   @Nullable
   default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      return bean;
   }

}
```



BeanPostProcessor中就两个方法，**postProcessBeforeInitialization**和**postProcessAfterInitialization**，分别会在Spring执行invokeInitMethods方法（包含了afterPropertiesSet和自定义init方法）前后进行执行。



> 这里要特别注意BeanPostProcessor中方法的最后一个单词是**Initialization**，这个很容易和**postProcessBeforeInstantiation**/**postProcessAfterInstantiation**方法混淆，两者的调用时机是不一样的，**Initialization**是在初始化前后，而**Instantiation**是在实例化前后。



下面介绍下BeanPostProcessor用途比较重要的两个实现：**CommonAnnotationBeanPostProcessor**和**AutowiredAnnotationBeanPostProcessor**。

Spring会在添加<context:component-scan > XML配置以及@ComponentScan()注解时创建默认的CommonAnnotationBeanPostProcessor和AutowiredAnnotationBeanPostProcessor处理器。



## CommonAnnotationBeanPostProcessor

<img src="/images/image-20210327153638683.png" alt="image-20210327153638683" style="zoom:50%;" />



最上层实现就是`BeanPostProcessor`接口。`CommonAnnotationBeanPostProcessor`支持常见的Java注解，特别是JSR-250注解，所以大部分的注解都会关于“资源”的构建、销毁和使用。

因为实现了InitDestroyAnnotationBeanPostProcessor接口，所以支持`@PostConstruct`，`@PreDestroy`关于对象初始化和销毁的注解。



支持注解如下：

+ **@Resource**
+ @PostConstruct
+ @PreDestroy
+ @WebServiceRef



```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {

   /**
    * 在bean实例化之前调用
    */
   @Nullable
   default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
      return null;
   }

   /**
    * 在bean实例化之后调用
    */
   default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
      return true;
   }

   /**
    * 当使用注解的时候，通过这个方法来完成属性的注入
    */
   @Nullable
   default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
         throws BeansException {

      return null;
   }

   /**
    * 属性注入后执行的方法，在5.1版本被废弃
    */
   @Deprecated
   @Nullable
   default PropertyValues postProcessPropertyValues(
         PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {

      return pvs;
   }

}
```



### 如何解析

`CommonAnnotationBeanPostProcessor`类中有如下方法会在Bean初始化后执行，用来填充BeanDefinition中的初始化方法以及@Resource对应的字段定义；

```java
@Override
public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
   // 处理@PostConstruct和@PreDestroy注解
   super.postProcessMergedBeanDefinition(beanDefinition, beanType, beanName);
   // 处理@Resouce注解对应的属性以及方法
   InjectionMetadata metadata = findResourceMetadata(beanName, beanType, null);
   metadata.checkConfigMembers(beanDefinition);
}
```



`postProcessMergedBeanDefinition`实际调用的是父类`InitDestroyAnnotationBeanPostProcessor`中的方法；



```java
@Override
public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
   // 获取生命周期方法
   LifecycleMetadata metadata = findLifecycleMetadata(beanType);
   // 验证相关方法
   metadata.checkConfigMembers(beanDefinition);
}
```

findLifecycleMetadata方法中会调用**buildLifecycleMetadata()**方法来构建对应生命周期方法的LifecycleElement（这里主要是@PostConstruct或者@PreDestroy注解对应的方法）。

```java
private LifecycleMetadata buildLifecycleMetadata(final Class<?> clazz) {
   if (!AnnotationUtils.isCandidateClass(clazz, Arrays.asList(this.initAnnotationType, this.destroyAnnotationType))) {
      return this.emptyLifecycleMetadata;
   }

   // @PostConstruct方法
   List<LifecycleElement> initMethods = new ArrayList<>();
   // @PreDestroy方法
   List<LifecycleElement> destroyMethods = new ArrayList<>();
  
   Class<?> targetClass = clazz;

   do {
      final List<LifecycleElement> currInitMethods = new ArrayList<>();
      final List<LifecycleElement> currDestroyMethods = new ArrayList<>();
      // 遍历当前类以及父类所有方法
      ReflectionUtils.doWithLocalMethods(targetClass, method -> {
         // 当前方法的注解中包含initAnnotationType注解时（@PostConstruct）
         if (this.initAnnotationType != null && method.isAnnotationPresent(this.initAnnotationType)) {
            // 如果有，把它封装成LifecycleElement对象，存储起来
            LifecycleElement element = new LifecycleElement(method);
            // 将创建好的元素添加到集合中
            currInitMethods.add(element);
            if (logger.isTraceEnabled()) {
               logger.trace("Found init method on class [" + clazz.getName() + "]: " + method);
            }
         }
         // 当前方法的注解中包含destroyAnnotationType注解（PreDestroy）
         if (this.destroyAnnotationType != null && method.isAnnotationPresent(this.destroyAnnotationType)) {
            // 如果有，把它封装成LifecycleElement对象，存储起来
            currDestroyMethods.add(new LifecycleElement(method));
            if (logger.isTraceEnabled()) {
               logger.trace("Found destroy method on class [" + clazz.getName() + "]: " + method);
            }
         }
      });

      initMethods.addAll(0, currInitMethods);
      destroyMethods.addAll(currDestroyMethods);
      // 获取父类class对象
      targetClass = targetClass.getSuperclass();
   }
   while (targetClass != null && targetClass != Object.class);

   return (initMethods.isEmpty() && destroyMethods.isEmpty() ? this.emptyLifecycleMetadata :
         new LifecycleMetadata(clazz, initMethods, destroyMethods));
}
```

`buildLifecycleMetadata`方法会封装类中对应生命周期方法的元数据，通过`getDeclaredMethods()`获取类中的所有方法，然后判断是否使用了**@PostConstruct**或者**@PreDestroy**注解，是就会以`LifecycleMetadata`的形式放入到`lifecycleMetadataCache`中，并且一层一层获取父类相关内容。

而`findResourceMetadata`方法同样会调用`buildResourceMetadata`方法来构建@Resource注解对应的属性或方法的ResourceElement。

也就是存在如下数据存放方式：

> InjectionMetadata -> Collection<InjectedElement> injectedElements  -> ResourceElement
>
> LifecycleMetadata -> Collection<LifecycleElement> initMethods / destroyMethods ->  LifecycleElement
>
> 

```java
private InjectionMetadata buildResourceMetadata(final Class<?> clazz) {
   // 判断当前的类是否是Resource或javax.xml.ws.WebServiceRef类
   if (!AnnotationUtils.isCandidateClass(clazz, resourceAnnotationTypes)) {
      return InjectionMetadata.EMPTY;
   }

   List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();
   Class<?> targetClass = clazz;

   do {
      final List<InjectionMetadata.InjectedElement> currElements = new ArrayList<>();

      // @Resource属性注入
      // 查询是否有webService,ejb,Resource的属性注解，但是不支持静态属性
      ReflectionUtils.doWithLocalFields(targetClass, field -> {
         if (webServiceRefClass != null && field.isAnnotationPresent(webServiceRefClass)) {
            if (Modifier.isStatic(field.getModifiers())) {
               throw new IllegalStateException("@WebServiceRef annotation is not supported on static fields");
            }
            currElements.add(new WebServiceRefElement(field, field, null));
         }
         else if (ejbClass != null && field.isAnnotationPresent(ejbClass)) {
            if (Modifier.isStatic(field.getModifiers())) {
               throw new IllegalStateException("@EJB annotation is not supported on static fields");
            }
            currElements.add(new EjbRefElement(field, field, null));
         }
         else if (field.isAnnotationPresent(Resource.class)) {
            //注意静态字段不支持
            if (Modifier.isStatic(field.getModifiers())) {
               throw new IllegalStateException("@Resource annotation is not supported on static fields");
            }
            //如果不想注入某一类型对象 可以将其加入ignoredResourceTypes中
            if (!this.ignoredResourceTypes.contains(field.getType().getName())) {
               //字段会封装到ResourceElement
               currElements.add(new ResourceElement(field, field, null));
            }
         }
      });

      // @Resource方法注入
      ReflectionUtils.doWithLocalMethods(targetClass, method -> {
        
         Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
         if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
            return;
         }
         //如果重写了父类的方法，则使用子类的
         if (method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
            if (webServiceRefClass != null && bridgedMethod.isAnnotationPresent(webServiceRefClass)) {
               // 静态字段不支持
               if (Modifier.isStatic(method.getModifiers())) {
                  throw new IllegalStateException("@WebServiceRef annotation is not supported on static methods");
               }
               if (method.getParameterCount() != 1) {
                  throw new IllegalStateException("@WebServiceRef annotation requires a single-arg method: " + method);
               }
               PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
               currElements.add(new WebServiceRefElement(method, bridgedMethod, pd));
            }
            else if (ejbClass != null && bridgedMethod.isAnnotationPresent(ejbClass)) {
               if (Modifier.isStatic(method.getModifiers())) {
                  throw new IllegalStateException("@EJB annotation is not supported on static methods");
               }
               if (method.getParameterCount() != 1) {
                  throw new IllegalStateException("@EJB annotation requires a single-arg method: " + method);
               }
               PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
               currElements.add(new EjbRefElement(method, bridgedMethod, pd));
            }
            else if (bridgedMethod.isAnnotationPresent(Resource.class)) {
               // 不支持静态方法
               if (Modifier.isStatic(method.getModifiers())) {
                  throw new IllegalStateException("@Resource annotation is not supported on static methods");
               }
               Class<?>[] paramTypes = method.getParameterTypes();
               if (paramTypes.length != 1) {
                  throw new IllegalStateException("@Resource annotation requires a single-arg method: " + method);
               }
               if (!this.ignoredResourceTypes.contains(paramTypes[0].getName())) {
                  PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
                  currElements.add(new ResourceElement(method, bridgedMethod, pd));
               }
            }
         }
      });
     
      // 父类的都放在第一位，所以父类是最先完成依赖注入的
      elements.addAll(0, currElements);
      targetClass = targetClass.getSuperclass();
   }
   while (targetClass != null && targetClass != Object.class);

   return InjectionMetadata.forElements(elements, clazz);
}
```



`buildResourceMetadata`方法和`buildLifecycleMetadata`实现基本一样，只不过是加了字段的遍历以及最后生成的元数据不一样罢了。



PS：

这里获取类字段和方法的函数，Spring也是进行了充分的优化；

```java
private static Field[] getDeclaredFields(Class<?> clazz) {

   Assert.notNull(clazz, "Class must not be null");

   Field[] result = declaredFieldsCache.get(clazz);

   if (result == null) {
      try {
         // 获取clazz所有定义的属性
         result = clazz.getDeclaredFields();
         // EMPTY_FIELD_ARRAY和result在没有字段情况下都是空数组，所以用一个固定的空数组来表示，减少新对象的内存占用
         declaredFieldsCache.put(clazz, (result.length == 0 ? EMPTY_FIELD_ARRAY : result));
      }
      catch (Throwable ex) {
         throw new IllegalStateException("Failed to introspect Class [" + clazz.getName() +
               "] from ClassLoader [" + clazz.getClassLoader() + "]", ex);
      }
   }
   // 返回clazz的属性数组
   return result;
}
```

首先，Spring将所有对应的方法或字段都放入到了Map缓存中。

此外，下面这行代码；

>  declaredFieldsCache.put(clazz, (result.length == 0 ? EMPTY_FIELD_ARRAY : result));

在缓存中还没有对应类的字段或方法时，通过Class类的`getDeclaredFields`方法先获取到，然后放入map集合中，主要是这里会对获取的数组长度做个判断，当为0时，实际放入的是EMPTY_FIELD_ARRAY对象，而EMPTY_FIELD_ARRAY定义如下；

```java
private static final Field[] EMPTY_FIELD_ARRAY = new Field[0];
```

**EMPTY_FIELD_ARRAY**实际就是一个长度为0的数组，实际上**EMPTY_FIELD_ARRAY**和类字段或方法不存在时代表的含义是相同的，而**getDeclaredFields**和**getDeclaredMethods**方法每次都会返回一个新数组，所以Spring利用**EMPTY_FIELD_ARRAY**作了代替，来让result数组较早的回收，就是为了减少内存的消耗。



## AutowiredAnnotationBeanPostProcessor

<img src="/images/image-20210327153254058.png" alt="image-20210327153254058" style="zoom:50%;" />

**AutowiredAnnotationBeanPostProcessor**从类名的定义上就能看出，该处理器主要适合**@Autowired**注解相关的。该处理器和CommonAnnotationBeanPostProcessor处理器执行的时间都是一样的，只是会后于CommonAnnotationBeanPostProcessor处理器执行。

这里只介绍相关的几个重要方法。

```java
private InjectionMetadata buildAutowiringMetadata(final Class<?> clazz) {
   if (!AnnotationUtils.isCandidateClass(clazz, this.autowiredAnnotationTypes)) {
      return InjectionMetadata.EMPTY;
   }

   List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();
   Class<?> targetClass = clazz;

   do {
      final List<InjectionMetadata.InjectedElement> currElements = new ArrayList<>();

      // 遍历类中的每个属性，判断属性是否包含指定的属性(通过 findAutowiredAnnotation 方法)
      // 如果存在则保存，这里注意，属性保存的类型是 AutowiredFieldElement
      ReflectionUtils.doWithLocalFields(targetClass, field -> {
         MergedAnnotation<?> ann = findAutowiredAnnotation(field);
         if (ann != null) {
            //Autowired注解不支持静态方法
            if (Modifier.isStatic(field.getModifiers())) {
               if (logger.isInfoEnabled()) {
                  logger.info("Autowired annotation is not supported on static fields: " + field);
               }
               return;
            }
            //查看是否是required的
            boolean required = determineRequiredStatus(ann);
            currElements.add(new AutowiredFieldElement(field, required));
         }
      });


      // 遍历类中的每个方法，判断属性是否包含指定的属性(通过 findAutowiredAnnotation 方法)
      // 如果存在则保存，这里注意，方法保存的类型是 AutowiredMethodElement
      ReflectionUtils.doWithLocalMethods(targetClass, method -> {
         Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
         if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
            return;
         }
         MergedAnnotation<?> ann = findAutowiredAnnotation(bridgedMethod);
         if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
            if (Modifier.isStatic(method.getModifiers())) {
               if (logger.isInfoEnabled()) {
                  logger.info("Autowired annotation is not supported on static methods: " + method);
               }
               return;
            }
            // 如果方法没有入参，输出日志，不做任何处理
            if (method.getParameterCount() == 0) {
               if (logger.isInfoEnabled()) {
                  logger.info("Autowired annotation should only be used on methods with parameters: " +
                        method);
               }
            }
            boolean required = determineRequiredStatus(ann);
            PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
            // AutowiredMethodElement里封装了一个PropertyDescriptor（比字段多了一个参数）
            currElements.add(new AutowiredMethodElement(method, required, pd));
         }
      });

      // 父类的都放在第一位，所以父类是最先完成依赖注入的
      elements.addAll(0, currElements);
      targetClass = targetClass.getSuperclass();
   }
   while (targetClass != null && targetClass != Object.class);

   // InjectionMetadata就是对clazz和elements的一个包装而已
   return InjectionMetadata.forElements(elements, clazz);
}
```



@Autowired处理器和@Resource处理器相比，方法类似，都是获取到所有的字段或者方法，然后遍历进行相应的判断处理。@Autowired处理器相对而言，包装的类型发生了变化（AutowiredMethodElement，AutowiredMethodElement），并且，**在@Autowired下，所有对象的注入都是从父类开始的**，也就是说在同注解的属性注入时，父类的属性注入会**先于**子类的属性注入。





两个处理器在populateBean对象注入的时候，执行的逻辑都是如下所示：

```java
@Override
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
   // 这里获取到的是之前解析保存的Metadata
   InjectionMetadata metadata = findResourceMetadata(beanName, bean.getClass(), pvs);
   try {
      metadata.inject(bean, beanName, pvs);
   }
   catch (Throwable ex) {
      throw new BeanCreationException(beanName, "Injection of resource dependencies failed", ex);
   }
   return pvs;
}
```

`findResourceMetadata`从之前解析类时所保存在BeanPostProcessor中的`injectionMetadataCache `Map集合中获取到InjectionMetadata对象，然后执行inject方法，循环遍历注入。

```java
public void inject(Object target, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
   Collection<InjectedElement> checkedElements = this.checkedElements;
   Collection<InjectedElement> elementsToIterate =
         (checkedElements != null ? checkedElements : this.injectedElements);
   if (!elementsToIterate.isEmpty()) {
      for (InjectedElement element : elementsToIterate) {
         if (logger.isTraceEnabled()) {
            logger.trace("Processing injected element of bean '" + beanName + "': " + element);
         }
         element.inject(target, beanName, pvs);
      }
   }
}
```





## 拓展：BeanPostProcessor执行顺序

- 实现了**PriorityOrdered**接口 > **Ordered**接口 > 两者都没实现；
- 实现了**MergedBeanDefinitionPostProcessor**一定最后执行；
- 相同情况下，还是按照**PriorityOrdered** > **Ordered**相对顺序。



相同情况下比较如下：

```Java
private int doCompare(@Nullable Object o1, @Nullable Object o2, @Nullable OrderSourceProvider sourceProvider) {
   // 判断o1是否实现了PriorityOrdered接口
   boolean p1 = (o1 instanceof PriorityOrdered);
   // 判断o2是否实现了PriorityOrdered接口
   boolean p2 = (o2 instanceof PriorityOrdered);
   // 如果o1实现了PriorityOrdered接口，o2没有，则o1排前面
   if (p1 && !p2) {
      return -1;
   }
   // 如果o2实现了PriorityOrdered接口，而o1没有，o2排前面
   else if (p2 && !p1) {
      return 1;
   }
   // 如果o1和o2都实现或者都没有实现PriorityOrdered接口
   // 拿到o1的order值，如果没有实现Ordered接口，值为Ordered.LOWEST_PRECEDENCE
   int i1 = getOrder(o1, sourceProvider);
   // 拿到o2的order值，如果没有实现Ordered接口，值为Ordered.LOWEST_PRECEDENCE
   int i2 = getOrder(o2, sourceProvider);
   // 通过order值排序(order值越小，优先级越高)
   return Integer.compare(i1, i2);
}
```







