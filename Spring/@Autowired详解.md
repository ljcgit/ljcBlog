# @Autowired详解

@Autowired是在Spring中新引入的注解，所属的包为`org.springframework.beans.factory.annotation;`

定义如下：

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {

   /**
    * Declares whether the annotated dependency is required.
    * <p>Defaults to {@code true}.
    */
   boolean required() default true;

}
```



@Autowired常用来作属性的注入，可以作用在构造方法、普通方法、字段、注解、参数上。

Spring中**AutowiredAnnotationBeanPostProcessor** 处理器负责处理@Autowired注解相关注入。

@Autowired和@Resourece对于字段的解析过程都是差不多了，这里就不作具体介绍了，有兴趣的可以看我另外一篇关于@Resouce的介绍。

> 两者最主要的就是用来处理的处理器不同，并且@Resource对应的处理器会先执行，所以对应的Bean也会先被加载。



在AutowiredAnnotationBeanPostProcessor处理器中，所有被@Autowired注解的类信息都会被保存到	`AutowiredMethodElement`类中，在对象真正注入的时候就会执行该类中的**inject**方法，具体实现如下：



```java
protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
			Field field = (Field) this.member;
			Object value;
			// 如果缓存，从缓存中获取
			if (this.cached) {
				// 如果  cachedFieldValue instanceof DependencyDescriptor。则调用 resolveDependency 方法重新加载。
				value = resolvedCachedArgument(beanName, this.cachedFieldValue);
			}
			else {
				// 否则调用了 resolveDependency 方法。这个在前篇讲过，在 populateBean 方法中按照类型注入的时候就是通过此方法，
				// 也就是说明了 @Autowired 和 @Inject默认是 按照类型注入的
				DependencyDescriptor desc = new DependencyDescriptor(field, this.required);
				desc.setContainingClass(bean.getClass());
				Set<String> autowiredBeanNames = new LinkedHashSet<>(1);
				Assert.state(beanFactory != null, "No BeanFactory available");
				// 转换器使用的bean工厂的转换器
				TypeConverter typeConverter = beanFactory.getTypeConverter();
				try {
					// 获取依赖的value值的工作  最终还是委托给beanFactory.resolveDependency()去完成的
					// 这个接口方法由AutowireCapableBeanFactory提供，它提供了从bean工厂里获取依赖值的能力
					value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
				}
				catch (BeansException ex) {
					throw new UnsatisfiedDependencyException(null, beanName, new InjectionPoint(field), ex);
				}
				// 把缓存值缓存起来
				synchronized (this) {
					// 如果没有缓存，则开始缓存
					if (!this.cached) {
						// 可以看到value！=null并且required=true才会进行缓存的处理
						if (value != null || this.required) {
							// 这里先缓存一下 desc，如果下面 utowiredBeanNames.size() > 1。则在上面从缓存中获取的时候会重新获取。
							this.cachedFieldValue = desc;
							// 注册依赖bean
							registerDependentBeans(beanName, autowiredBeanNames);
							// autowiredBeanNames里可能会有别名的名称,所以size可能大于1
							if (autowiredBeanNames.size() == 1) {
								// beanFactory.isTypeMatch挺重要的,因为@Autowired是按照类型注入的
								String autowiredBeanName = autowiredBeanNames.iterator().next();
								if (beanFactory.containsBean(autowiredBeanName) &&
										beanFactory.isTypeMatch(autowiredBeanName, field.getType())) {
									this.cachedFieldValue = new ShortcutDependencyDescriptor(
											desc, autowiredBeanName, field.getType());
								}
							}
						}
						else {
							this.cachedFieldValue = null;
						}
						this.cached = true;
					}
				}
			}
			if (value != null) {
				// 通过反射，给属性赋值
				ReflectionUtils.makeAccessible(field);
				field.set(bean, value);
			}
		}
```



仔细看inject方法实现会发现，其实也是调用了BeanFactory的**resolveDependency**方法来获取到需要注入的bean，如果看过@Resource的同学就会发现这和@Resource不设置任何属性时所执行的方法是一样的。

这就会导致和@Resouce同样的问题，即在一个接口有多个实现类的情况下，就会抛出**NoUniqueBeanDefinitionException**的异常。

```java
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
```



那么如何解决多实现类的情况呢？

在Spring获取某类型的所有相关类时，会有一个判断，如下；

```java
for (String candidate : candidateNames) {
   // 如果beanName与candidateName所对应的Bean对象不是同一个且candidate可以自动注入
   if (!isSelfReference(beanName, candidate) && isAutowireCandidate(candidate, descriptor)) {
      // 添加一个条目在result中:一个bean实例(如果可用)或仅一个已解析的类型
      addCandidateEntry(result, candidate, descriptor, requiredType);
   }
}
```



其中**isAutowireCandidate(candidate, descriptor)**判断条件就可以用来控制匹配的类数量，其中主要的判断过程如下所示：

```java
public boolean isAutowireCandidate(BeanDefinitionHolder bdHolder, DependencyDescriptor descriptor) {
   boolean match = super.isAutowireCandidate(bdHolder, descriptor);
   if (match) {
      // 比较name是否一致
      match = checkQualifiers(bdHolder, descriptor.getAnnotations());
      if (match) {
         MethodParameter methodParam = descriptor.getMethodParameter();
         if (methodParam != null) {
            Method method = methodParam.getMethod();
            if (method == null || void.class == method.getReturnType()) {
               match = checkQualifiers(bdHolder, methodParam.getMethodAnnotations());
            }
         }
      }
   }
   return match;
}
```



而该方法又调用了**checkQualifiers**方法，看到这里就明白了我们需要引入**@Qualifier**注解了；

> ```java
> @Autowired
> @Qualifier(value = "apple")
> private Fruit fruit;
> ```

```java
protected boolean checkQualifiers(BeanDefinitionHolder bdHolder, Annotation[] annotationsToSearch) {
   if (ObjectUtils.isEmpty(annotationsToSearch)) {
      return true;
   }
   SimpleTypeConverter typeConverter = new SimpleTypeConverter();
   for (Annotation annotation : annotationsToSearch) {
      //获取每个注解类型
      Class<? extends Annotation> type = annotation.annotationType();
      boolean checkMeta = true;
      boolean fallbackToMeta = false;
      if (isQualifier(type)) {
         // 判断是否是Qualifier注解，一种是spring内部的Qualifier，一种是javax.inject.Qualifier
         // 判断value是否和beanName一致
         if (!checkQualifier(bdHolder, annotation, typeConverter)) {
            //检查bean定义里是否有这个注解
            fallbackToMeta = true;
         }
         else {
            checkMeta = false;
         }
      }
      if (checkMeta) {
         boolean foundMeta = false;
         for (Annotation metaAnn : type.getAnnotations()) {
            Class<? extends Annotation> metaType = metaAnn.annotationType();
            if (isQualifier(metaType)) {
               foundMeta = true;
               // Only accept fallback match if @Qualifier annotation has a value...
               // Otherwise it is just a marker for a custom qualifier annotation.
               if ((fallbackToMeta && StringUtils.isEmpty(AnnotationUtils.getValue(metaAnn))) ||
                     !checkQualifier(bdHolder, metaAnn, typeConverter)) {
                  return false;
               }
            }
         }
         if (fallbackToMeta && !foundMeta) {
            return false;
         }
      }
   }
   return true;
}
```



在大多数情况下，当`@Qualifier`注解中的value值和对应的`BeanDefinitionHolder`中beanName一致时就会返回true，也就会将相应的bean信息添加到`matchingBeans`集合中，进而保证了注入的bean的唯一性（因为beanName是唯一的）。



## 总结

**@Autowired**和**@Resource**（不设置任何属性）最基本的实现都是一样的（最后都调用BeanFactory的**resolveDependency**方法）。而@Autowired是不带有name和type属性的，也就无法向@Resource一样直接指明注入bean的相关信息，但是可以借助**@Qualifier**来达到@Resouce(name)同样的目的（当然@Resource也可以和@Qualifier结合使用，不过没有必要）。

