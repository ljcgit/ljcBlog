

# @Resource详解

阅读本文之前希望读者最好已经对整个Bean的大体Spring执行顺序已经有了一定的了解。



**示例**

定义一个接口，表示水果类，只包含一个方法代表售卖。

```java
public interface Fruit {

    void sell();
}
```

有两个具体实现类，Apple🍎和Banana🍌。



```java
@Service
public class Apple implements Fruit {


    public Apple() {
        System.out.println("Apple......");
    }

    @Override
    public void sell() {
        System.out.println("苹果2元一斤");
    }
}
```



```java
@Service
public class Banana implements Fruit {

    public Banana() {
        System.out.println("Banana...");
    }

    @Override
    public void sell() {
        System.out.println("香蕉3元一串");
    }
}
```



具体的商店类来注入Fruit接口。

```java
@Component
public class Store {

    @Resource
    private Fruit fruit;

    public void getFruit() {
        fruit.sell();
    }

}
```



测试方法：

```java
public class Test {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        context.getBean(Store.class).getFruit();


        // 优雅关闭，SpringWEB中已有相关实现
        context.registerShutdownHook();
    }
}
```



实际执行结果：

![image-20210331103704825](images/image-20210331103704825的副本2.png)

这里可以清楚的看到报了`NoUniqueBeanDefinitionException`异常，说是希望单个Bean的匹配，却找到了多个。



下面就来具体的讲下为什么。



首先，@Resource中没有设置任何属性值，统统采用的是默认的值。

> 按照Spring Bean的加载顺序，Store Bean创建的时候，BeanFactory中已经创建了Apple和Banana Bean。



<img src="/images/image-20210331110643824的副本2.png" alt="image-20210331110643824" style="zoom:50%;" />





### 1.  newInstance

第一步就是先创建出Store对象。



### 2.  解析类中的字段

Spring在实例化对象后，会调用 `applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);`方法对BeanDefinition进行完善，主要为了后续的BeanPostProcessor处理器在注入对应的字段时能获取到需要注入的类的相关信息。

而对应需要注入的一个个类而言，就是使用**ResourceElement**对象来进行保存，相关重要的构造函数如下：

```java
public ResourceElement(Member member, AnnotatedElement ae, @Nullable PropertyDescriptor pd) {
   super(member, pd);
   Resource resource = ae.getAnnotation(Resource.class);
   // 获取@Resource的name属性
   String resourceName = resource.name();
   // 获取@Resource的type属性
   Class<?> resourceType = resource.type();
   this.isDefaultName = !StringUtils.hasLength(resourceName);
   if (this.isDefaultName) {
      // 如果没有设置@Resource name属性就用字段名称作为bean name
      resourceName = this.member.getName();
      // 如果member是setter方法，则取setXXX的XXX部分为bean name
      if (this.member instanceof Method && resourceName.startsWith("set") && resourceName.length() > 3) {
         resourceName = Introspector.decapitalize(resourceName.substring(3));
      }
   }
   else if (embeddedValueResolver != null) {
      // 如果设置了@Resource name的属性，则使用EmbeddedValueResolver对象先做一次SpringEL解析得到真正的bean name
      resourceName = embeddedValueResolver.resolveStringValue(resourceName);
   }
   if (Object.class != resourceType) {
      // 确保字段或setter方法类型与resourceType一致
      checkResourceType(resourceType);
   }
   else {
      // No resource type specified... check field/method.
      resourceType = getResourceType();
   }
   this.name = (resourceName != null ? resourceName : "");
   this.lookupType = resourceType;
   String lookupValue = resource.lookup();
   // 如果使用jndi查找名字
   this.mappedName = (StringUtils.hasLength(lookupValue) ? lookupValue : resource.mappedName());
   Lazy lazy = ae.getAnnotation(Lazy.class);
   // 是否延迟注入
   this.lazyLookup = (lazy != null && lazy.value());
}
```



当name属性没有被设置时，就会执行下面的分支，根据是方法注入还是属性注入，分别设置为方法名称set后面的字符串或字段名称。



```java
   if (this.isDefaultName) {
      // 如果没有设置@Resource name属性就用字段名称作为bean name
      resourceName = this.member.getName();
      // 如果member是setter方法，则取setXXX的XXX部分为bean name
      if (this.member instanceof Method && resourceName.startsWith("set") && resourceName.length() > 3) {
         resourceName = Introspector.decapitalize(resourceName.substring(3));
      }
   }
```



下面是具体的ResourceElement类对象中的各属性。

![image-20210331103357922](/images/image-20210331103357922.png)



然后可以测试下在@Resource注解中加入name属性；

> @Resource(name = "apple")



得到的是下面的对象。

![image-20210331104307762](/images/image-20210331104307762的副本2.png)



**这里可以看出name属性会有明显的不同。**



> 这里name属性和lookupType属性其实可以对应于@Resource中的name和type属性。



### 3. populateBean

第三步就是类属性的注入。

执行到对应的处理器（@Resource是通过`CommonAnnotationBeanPostProcessor`处理器进行处理的）进行属性注入的时候，`autowireResource`就会用来获取对应属性需要注入的对象。

```java
protected Object autowireResource(BeanFactory factory, LookupElement element, @Nullable String requestingBeanName)
      throws NoSuchBeanDefinitionException {

   // 自动装配的对象
   Object resource;
   // 自动装配的名字
   Set<String> autowiredBeanNames;
   // 依赖的属性名
   String name = element.name;

   if (factory instanceof AutowireCapableBeanFactory) {
      AutowireCapableBeanFactory beanFactory = (AutowireCapableBeanFactory) factory;
      DependencyDescriptor descriptor = element.getDependencyDescriptor();
      // 判断是否设置了name属性的值
      if (this.fallbackToDefaultTypeMatch && element.isDefaultName && !factory.containsBean(name)) {
         autowiredBeanNames = new LinkedHashSet<>();
         resource = beanFactory.resolveDependency(descriptor, requestingBeanName, autowiredBeanNames, null);
         if (resource == null) {
            throw new NoSuchBeanDefinitionException(element.getLookupType(), "No resolvable resource object");
         }
      }
      else {
         resource = beanFactory.resolveBeanByName(name, descriptor);
         autowiredBeanNames = Collections.singleton(name);
      }
   }
   else {
      resource = factory.getBean(name, element.lookupType);
      autowiredBeanNames = Collections.singleton(name);
   }

   if (factory instanceof ConfigurableBeanFactory) {
      ConfigurableBeanFactory beanFactory = (ConfigurableBeanFactory) factory;
      for (String autowiredBeanName : autowiredBeanNames) {
         if (requestingBeanName != null && beanFactory.containsBean(autowiredBeanName)) {
            //注册依赖关系
            beanFactory.registerDependentBean(autowiredBeanName, requestingBeanName);
         }
      }
   }

   return resource;
}
```

这里的element对象就是我们之前解析保存的**ResourceElement**。

然后最重要的，上面强调过的name属性在不同Resource注解中解析出来的ResourceElement是会不同的，这里就会有不同的处理方式。

当`isDefaultName`为false时，说明name属性被设置了值，此时执行的如下逻辑；

```java
     resource = beanFactory.resolveBeanByName(name, descriptor);
     autowiredBeanNames = Collections.singleton(name);
```

`resolveBeanByName`方法就是通过设置的名称来进行解析对应的Bean对象。接着就能看到经常使用到的getBean方法，从BeanFactory中拿到Bean对象。

```java
public Object resolveBeanByName(String name, DependencyDescriptor descriptor) {
   InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
   try {
      // 获取bean
      return getBean(name, descriptor.getDependencyType());
   }
   finally {
      // 为目标工厂方法提供依赖描述符
      ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
   }
}
```

当Resource注解中name没有设置值，即使用的是字段名称作为beanName，执行的就是

```java
         resource = beanFactory.resolveDependency(descriptor, requestingBeanName, autowiredBeanNames, null);
```



resolveDependency方法中实际工作的方法就是doResolveDependency：

```java
public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName,
      @Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {

   //设置新得当前切入点对象，得到旧的当前切入点对象
   InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);
   
       // ...省略

      // 查找相关所有类型匹配的bean
      Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
      //如果没有候选bean对象
      if (matchingBeans.isEmpty()) {
         //如果descriptor需要注入
         if (isRequired(descriptor)) {
            //抛出NoSuchBeanDefinitionException或BeanNotOfRequiredTypeException以解决不可 解决的依赖关系
            raiseNoMatchingBeanFound(type, descriptor.getResolvableType(), descriptor);
         }
         //返回null，表示么有找到候选Bean对象
         return null;
      }

      //定义用于存储唯一的候选Bean名变量
      String autowiredBeanName;
      //定义用于存储唯一的候选Bean对象变量
      Object instanceCandidate;

      //如果候选Bean对象Map不止有一个
      if (matchingBeans.size() > 1) {
         //确定candidates中可以自动注入的最佳候选Bean名称
         autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor);

         if (autowiredBeanName == null) {
            // 如果有多个匹配结果，抛出异常
            if (isRequired(descriptor) || !indicatesMultipleBeans(type)) {
               //让descriptor尝试选择其中一个实例，默认实现是抛出NoUniqueBeanDefinitionException.
               return descriptor.resolveNotUnique(descriptor.getResolvableType(), matchingBeans);
            }
            else {
               return null;
            }
         }
         instanceCandidate = matchingBeans.get(autowiredBeanName);
      }
      else {
         //获取machingBeans唯一的元素
         Map.Entry<String, Object> entry = matchingBeans.entrySet().iterator().next();
         autowiredBeanName = entry.getKey();
         instanceCandidate = entry.getValue();
      }

      // ...省略
  
      //返回最佳候选Bean对象【result】
      return result;
   }
   finally {
      //设置上一个切入点对象
      ConstructorResolver.setCurrentInjectionPoint(previousInjectionPoint);
   }
}
```



doResolveDependency方法中会获取所有和指定type相同的所有bean集合，**当发现bean集合中的元素个数超过1时就抛出了上面出现的错误了**，如果只有一个那自然而然就是拿那个bean来返回。

这里抛出的异常其实不是直接抛出的，而是调用的DependencyDescriptor中的resolveNotUnique方法，而该方法默认实现就是会直接抛出一个NoUniqueBeanDefinitionException异常，这也是Spring留的一个可扩展的地方。

我们可以通过继承DependencyDescriptor，然后重写该方法来自定义选择一个最优的bean（比如说选择第一个），然后在BeanPostProcessor中选择我们的子类作为实现，这样在@Resource不指定任何属性的情况下，有多个实现类的bean也不会抛出异常。

![image-20210331135223970](/images/image-20210331135223970的副本2.png)



**拓展**

- 当只指定type属性时；

> @Resource(type = Apple.class)

![image-20210331142358004](/images/image-20210331142358004的副本2.png)



设置type属性后，isDefaultName的值还是为true，所以执行的还是resolveDependency方法。但是由于添加了类型的限制，所以也就不会匹配到多个Bean，而产生异常。



- 既指定了name属性，又指定了type类型，但是是不同的类；

> @Resource(name = "banana", type = Apple.class)

![image-20210331144128910](/images/image-20210331144128910的副本2.png)

name属性被设置为banana，isDefaultName变为false，执行resolveBeanByName方法。

但是由于找不到对应beanName为banana，但是类型又为Apple.class的bean，还是会抛出异常。

![image-20210331144853902](/images/image-20210331144853902的副本2.png)

###  

## 总结

多种@Resource不同使用情况所执行方法如下所示；

![image-20210331150628161](/images/image-20210331150628161的副本2.png)

- 当@Resource不设置任何属性值时，会走**resolveDependency**方法，获取到所有类型匹配的bean来进行选择；
- 只指定了type属性时，会找到唯一的一个类型匹配的bean；
- 只指定了name属性，会执行**getBean**方法，根据指定的name来获取bean；
- 既指定了name属性，又指定了type属性，就会获取同时满足了两个条件的bean。
