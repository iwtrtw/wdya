#### Spring 循环依赖

Spring中之所以会出现循环依赖跟Bean的生命周期有关系，在创建一个Bean的过程中如果依赖的另外一个Bean还没有创建，就会需要去创建依赖的那个Bean，而如果两个Bean相互依赖的话，就会出现循环依赖的问题。

```java
@Component
public class A {
    // A中注入了B
    @Autowired
    private B b;
}

@Component
public class B {
    // B中也注入了A
    @Autowired
    private A a;
}
```

```java
@Component
public class A {
    // A中注入了A
    @Autowired
    private A a;
}
```

Spring创建bean的过程主要可以分为三个阶段

+ 实例化：new一个对象

  >  AbstractAutowireCapableBeanFactory中的createBeanInstance方法

+ 属性注入：为实例化中new出来的对象填充属性>

  > AbstractAutowireCapableBeanFactory的populateBean方法

+ 初始化：初始化，执行aware接口中的方法，完成AOP代理

  > AbstractAutowireCapableBeanFactory的initializeBean

  

假设整个流程以A的创建为起点，创建A的过程实际上就是调用getBean方法，getBean又会去调用doGetBean方方法，这个方法有两层含义

> 创建一个新的Bean
>
> 从缓存中获取到已经被创建的对象

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly) throws BeansException {
        String beanName = this.transformedBeanName(name);
    //首先调用getSingleton(a)方法，这个方法又会调用getSingleton(beanName, true),尝试从三级缓存中获取该bean; A第一次创建，所以哪个缓存都不会命中，则返回结果为null
        Object sharedInstance = this.getSingleton(beanName);
        Object bean;
        if (sharedInstance != null && args == null) {
            
        } else {
//从缓存中获取A失败，A为单例，则调用另外一个重载方法getSingleton(beanName,  singletonFactory)。
                if (mbd.isSingleton()) {
                    sharedInstance = this.getSingleton(beanName, () -> {
                        try {
      //在createBean方法调用doCreateBean真正开始创建bean A，在A实例化后，属性注入前调用addSingletonFactory放入三级缓存singletonFactories中
                            return this.createBean(beanName, mbd, args);

     //......
    }
```

#### getSingleton(beanName, true)

第一次获取A时尝试从到一级缓存(singletonObjects单例池)中尝试去获取Bean，整个缓存分为三级（尝试从二级缓存和三级缓存获取bean的前提是该bean是正在创建当中）

+ singletonObjects：一级缓存，存储的是所有创建好了的单例Bean
+ earlySingletonObjects：完成实例化，但是还未进行属性注入及初始化的对象
+ singletonFactories：提前暴露的一个单例工厂，二级缓存中存储的就是从这个工厂中获取到的对象

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        //一级缓存中获取-->完整bean
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
            //当一级缓存中没有该bean且该bean在创建中
            synchronized (this.singletonObjects) {
                //二级缓存中-->获取未属性注入的bean
                singletonObject = this.earlySingletonObjects.get(beanName);
                if (singletonObject == null && allowEarlyReference) {
                    //三级缓存中-->获取工厂
                    ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                    if (singletonFactory != null) {
                        singletonObject = singletonFactory.getObject();
                        //三级缓存中的工厂getObject的对象-->放入二级缓存中
                        this.earlySingletonObjects.put(beanName, singletonObject);
                        this.singletonFactories.remove(beanName);
                    }
                }
            }
        }
        return singletonObject;
    }
```

第一次获取A时，一级缓存未命中（而A还没有标记为正在创建当中，所以直接返回null）此时会进入另一个getSingleton(beanName, singletonFactory)。在此方法中会调用singletonFactory.getObject()方法，然后回调createBean(beanName, mbd, args)方法去完成bean的创建并返回，最后把bean放到到一级缓存中。参数singletonFactory 是需要等待createBean(beanName, mbd, args) 方法的返回。

#### getSingleton(beanName, singletonFactory)

标记当前单例bean A正在创建当中，然后通过createBean方法返回的A最终被放到了一级缓存，也就是单例池中。createBean()方法会完成A的实例化、属性注入【发现依赖B、实例化B、属性注入、发现A(可以从三级缓存中获取)，完成bean B的创建】、完成A的创建

```java
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "Bean name must not be null");
        Map var3 = this.singletonObjects;
        synchronized(this.singletonObjects) {
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
             //......省略异常处理及日志
             //标志着这个单例Bean正在创建,将beanName放入到singletonsCurrentlyInCreation这个集合中;如果同一个单例Bean多次被创建，这里会抛出异常
                this.beforeSingletonCreation(beanName);
                boolean newSingleton = false;
                boolean recordSuppressedExceptions = this.suppressedExceptions == null;
                if (recordSuppressedExceptions) {
                    this.suppressedExceptions = new LinkedHashSet();
                }

                try {
                    //前面传入的lambda在这里会被执行，回调createBean方法真正去创建一个Bean后返回
                    singletonObject = singletonFactory.getObject();
                    newSingleton = true;
                }
                // 省略catch异常处理
                } finally {
                    if (recordSuppressedExceptions) {
                        this.suppressedExceptions = null;
                    }
                     // 创建完成后将对应的beanName从singletonsCurrentlyInCreation移除
                    this.afterSingletonCreation(beanName);
                }

                if (newSingleton) {
                     // 添加到一级缓存singletonObjects中
                    this.addSingleton(beanName, singletonObject);
                }
            }

            return singletonObject;
        }
}
```

#### createBean(beanName, mbd, args)

调用doCreateBean：实例化对象，在属性注入前将实例化的bean先放到三级缓存singletonFactory中

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
    //....

        RootBeanDefinition mbdToUse = mbd;
        Class<?> resolvedClass = this.resolveBeanClass(mbd, beanName, new Class[0]);

    //....
        try {
            //实例化bean、将实例bean放入三级缓存中、属性注入、初始化
            beanInstance = this.doCreateBean(beanName, mbdToUse, args);
            
            }
            return beanInstance;
        } 
//....
    }
```

#### doCreateBean(beanName, mbdToUse, args)

实例化bean、将允许循环依赖并正在创建当中的bean放入到三级缓存中、属性注入、执行初始化方法

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
        BeanWrapper instanceWrapper = null;
        if (mbd.isSingleton()) {
            instanceWrapper = (BeanWrapper)this.factoryBeanInstanceCache.remove(beanName);
        }

        if (instanceWrapper == null) {
            //调用bean的构造方法完成实例化（反射获取class对象，然后调用实例化方法）
            instanceWrapper = this.createBeanInstance(beanName, mbd, args);
        }

        Object bean = instanceWrapper.getWrappedInstance();
        Class<?> beanType = instanceWrapper.getWrappedClass();
        if (beanType != NullBean.class) {
            mbd.resolvedTargetType = beanType;
        }

        Object var7 = mbd.postProcessingLock;
        synchronized(mbd.postProcessingLock) {
            if (!mbd.postProcessed) {
                try {
                    this.applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
                } catch (Throwable var17) {
                    throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Post-processing of merged bean definition failed", var17);
                }

                mbd.postProcessed = true;
            }
        }
//判断该bean是否为单例、是否允许循环依赖、是否在创建当中
        boolean earlySingletonExposure = mbd.isSingleton() && this.allowCircularReferences && this.isSingletonCurrentlyInCreation(beanName);
        if (earlySingletonExposure) {
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Eagerly caching bean '" + beanName + "' to allow for resolving potential circular references");
            }
            
            //将实例化后但属性未注入的单例bean放入到三级缓存singletonFactories中，后面从三级缓存中获取bean调用singletonFactory.getObject()会回调getEarlyBeanReference获取到A
            this.addSingletonFactory(beanName, () -> {
                return this.getEarlyBeanReference(beanName, mbd, bean);
            });
        }

        Object exposedObject = bean;

        try {
            //属性注入（通过BeanPostProcessor）
            this.populateBean(beanName, mbd, instanceWrapper);
            exposedObject = this.initializeBean(beanName, exposedObject, mbd);
        } 
    

    //......
    }
```

#### addSingletonFactory

在完成Bean的实例化后，属性注入之前Spring将Bean包装成一个工厂添加进了三级缓存中. 

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(singletonFactory, "Singleton factory must not be null");
        Map var3 = this.singletonObjects;
        synchronized(this.singletonObjects) {
            if (!this.singletonObjects.containsKey(beanName)) {
                // 添加到三级缓存中
                this.singletonFactories.put(beanName, singletonFactory);
                this.earlySingletonObjects.remove(beanName);
                this.registeredSingletons.add(beanName);
            }
        }
    }
```

这里只是添加了一个工厂，后面通过这个工厂（ObjectFactory）的getObject方法可以得到一个对象，而这个对象实际上就是通过getEarlyBeanReference这个方法创建的。那么，什么时候会去调用这个工厂的getObject方法呢？这个时候就要到创建B的流程了。

当A完成了实例化并添加进了三级缓存后，就要开始为A进行属性注入populateBean，在注入时发现A依赖了B，那么这个时候Spring又会去getBean(b)，然后反射调用setter方法完成属性注入

#### populateBean(beanName, mbd, instanceWrapper)

对实例化后的A进行属性注入，发现属性包含B对象，然后尝试从缓存中获取B，第一次从单例池中获取失败，然后开始进入B的创建流程

```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    //......

            PropertyValues pvs = mbd.hasPropertyValues() ? mbd.getPropertyValues() : null;
            int resolvedAutowireMode = mbd.getResolvedAutowireMode();
            if (resolvedAutowireMode == 1 || resolvedAutowireMode == 2) {
                MutablePropertyValues newPvs = new MutablePropertyValues((PropertyValues)pvs);
                if (resolvedAutowireMode == 1) {
                    this.autowireByName(beanName, mbd, bw, newPvs);
                }

                if (resolvedAutowireMode == 2) {
                    this.autowireByType(beanName, mbd, bw, newPvs);
                }

                pvs = newPvs;
            }

            boolean hasInstAwareBpps = this.hasInstantiationAwareBeanPostProcessors();
            boolean needsDepCheck = mbd.getDependencyCheck() != 0;
            PropertyDescriptor[] filteredPds = null;
            if (hasInstAwareBpps) {
                if (pvs == null) {
                    pvs = mbd.getPropertyValues();
                }

                PropertyValues pvsToUse;
                for(Iterator var9 = this.getBeanPostProcessorCache().instantiationAware.iterator(); var9.hasNext(); pvs = pvsToUse) {
                    InstantiationAwareBeanPostProcessor bp = (InstantiationAwareBeanPostProcessor)var9.next();
                    
                    //执行属性注入
                    pvsToUse = bp.postProcessProperties((PropertyValues)pvs, bw.getWrappedInstance(), beanName);
                    if (pvsToUse == null) {
                        if (filteredPds == null) {
                            filteredPds = this.filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                        }

                        pvsToUse = bp.postProcessPropertyValues((PropertyValues)pvs, filteredPds, bw.getWrappedInstance(), beanName);
                        if (pvsToUse == null) {
                            return;
                        }
                    }
                }
            }
//......
        }
    }
```

 + postProcessProperties(PropertyValues pvs, Object bean, String beanName) 

```java
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
        InjectionMetadata metadata = this.findAutowiringMetadata(beanName, bean.getClass(), pvs);

        try {
            //属性注入
            metadata.inject(bean, beanName, pvs);
            return pvs;
        } catch (BeanCreationException var6) {
            throw var6;
        } catch (Throwable var7) {
            throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", var7);
        }
    }
```

+ InjectionMetadata.inject(bean, beanName, pvs)

```java
public void inject(Object target, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
        Collection<InjectionMetadata.InjectedElement> checkedElements = this.checkedElements;
        Collection<InjectionMetadata.InjectedElement> elementsToIterate = checkedElements != null ? checkedElements : this.injectedElements;
        if (!((Collection)elementsToIterate).isEmpty()) {
            Iterator var6 = ((Collection)elementsToIterate).iterator();
            while(var6.hasNext()) {
                InjectionMetadata.InjectedElement element = (InjectionMetadata.InjectedElement)var6.next();
                //属性注入
                element.inject(target, beanName, pvs);
            }
        }

    }
```

+ AutowiredAnnotationBeanPostProcessor.inject

```java
protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
            Field field = (Field)this.member;
            Object value;
            if (this.cached) {
                value = AutowiredAnnotationBeanPostProcessor.this.resolvedCachedArgument(beanName, this.cachedFieldValue);
            } else {
                DependencyDescriptor desc = new DependencyDescriptor(field, this.required);
                desc.setContainingClass(bean.getClass());
                Set<String> autowiredBeanNames = new LinkedHashSet(1);
                Assert.state(AutowiredAnnotationBeanPostProcessor.this.beanFactory != null, "No BeanFactory available");
                TypeConverter typeConverter = AutowiredAnnotationBeanPostProcessor.this.beanFactory.getTypeConverter();

                try {
//解决依赖
                    value = AutowiredAnnotationBeanPostProcessor.this.beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
                } catch (BeansException var13) {
                    throw new UnsatisfiedDependencyException((String)null, beanName, new InjectionPoint(field), var13);
                }
//......
        }
```

+ DefaultListableBeanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter)

```java
public Object resolveDependency(DependencyDescriptor descriptor, @Nullable String requestingBeanName, @Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {
        descriptor.initParameterNameDiscovery(this.getParameterNameDiscoverer());
        if (Optional.class == descriptor.getDependencyType()) {
            return this.createOptionalDependency(descriptor, requestingBeanName);
        } else if (ObjectFactory.class != descriptor.getDependencyType() && ObjectProvider.class != descriptor.getDependencyType()) {
            if (javaxInjectProviderClass == descriptor.getDependencyType()) {
                return (new DefaultListableBeanFactory.Jsr330Factory()).createDependencyProvider(descriptor, requestingBeanName);
            } else {
                Object result = this.getAutowireCandidateResolver().getLazyResolutionProxyIfNecessary(descriptor, requestingBeanName);
                if (result == null) {
                    //解决依赖
                    result = this.doResolveDependency(descriptor, requestingBeanName, autowiredBeanNames, typeConverter);
                }

                return result;
            }
        } else {
            return new DefaultListableBeanFactory.DependencyObjectProvider(descriptor, requestingBeanName);
        }
    }
```

+ DefaultListableBeanFactory.doResolveDependency(descriptor, requestingBeanName, autowiredBeanNames, typeConverter)

```java
    @Nullable
public Object doResolveDependency(DependencyDescriptor descriptor, @Nullable String beanName, @Nullable Set<String> autowiredBeanNames, @Nullable TypeConverter typeConverter) throws BeansException {
        InjectionPoint previousInjectionPoint = ConstructorResolver.setCurrentInjectionPoint(descriptor);

        //......
                                Entry<String, Object> entry = (Entry)matchingBeans.entrySet().iterator().next();
                                autowiredBeanName = (String)entry.getKey();
                                instanceCandidate = entry.getValue();
                            }

                            if (autowiredBeanNames != null) {
                                autowiredBeanNames.add(autowiredBeanName);
                            }

                            if (instanceCandidate instanceof Class) {
//属性B注入类型为class类型，则尝试从工厂中获取B对象
                                instanceCandidate = descriptor.resolveCandidate(autowiredBeanName, type, this);
                            }
    }
```

+ DependencyDescriptor.resolveCandidate(autowiredBeanName, type, this)

```java
public Object resolveCandidate(String beanName, Class<?> requiredType, BeanFactory beanFactory) throws BeansException {
        return beanFactory.getBean(beanName);
}
```

**至此，尝试从工厂中获取B对象，发现缓存中不存在B，然后开始B的创建流程：实例化、添加到三级缓存、属性注入......B在属性注入时发现依赖A，然后尝试从缓存中获取A，这一次可以从三级缓存singletonFactories中获取到**

因为B需要注入A，所以在创建B的时候，又会去调用getBean(a)，这个时候就又回到之前的流程了，但是不同的是，之前的getBean是为了创建Bean，而此时再调用getBean不是为了创建了，而是要从缓存中获取，因为之前A在实例化后已经将其放入了三级缓存singletonFactories中，所以此时getBean(a)的流程就是这样子了

```java
 protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        Object singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null && this.isSingletonCurrentlyInCreation(beanName)) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                Map var4 = this.singletonObjects;
                synchronized(this.singletonObjects) {
                    singletonObject = this.singletonObjects.get(beanName);
                    if (singletonObject == null) {
                        singletonObject = this.earlySingletonObjects.get(beanName);
                        if (singletonObject == null) {
//通过三级缓存把A对应的工厂拿出来
                            ObjectFactory<?> singletonFactory = (ObjectFactory)this.singletonFactories.get(beanName);
                            if (singletonFactory != null) {
//通过工厂获取A，singletonFactory.getObject()会回调getEarlyBeanReference获取到A
                                singletonObject = singletonFactory.getObject();
                                this.earlySingletonObjects.put(beanName, singletonObject);
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

#### getEarlyBeanReference

返回一个非完整的bean，调用后置处理器的getEarlyBeanReference，而真正实现了这个方法的后置处理器只有一个，就是通过@EnableAspectJAutoProxy注解导入的AnnotationAwareAspectJAutoProxyCreator。

```java
//默认实现 
default Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
        return bean;
}
```



```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
        Object exposedObject = bean;
        SmartInstantiationAwareBeanPostProcessor bp;
        if (!mbd.isSynthetic() && this.hasInstantiationAwareBeanPostProcessors()) {
            for(Iterator var5 = this.getBeanPostProcessorCache().smartInstantiationAware.iterator(); var5.hasNext(); exposedObject = bp.getEarlyBeanReference(exposedObject, beanName)) {
                bp = (SmartInstantiationAwareBeanPostProcessor)var5.next();
            }
        }
```

也就是说如果在不考虑AOP的情况下，上面的代码等价于下面，即直接把实例化阶段的对象返回了（三级缓存的意义何在？)

```java
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
        Object exposedObject = bean;
        return exposedObject;
}
```

如果在开启AOP的情况下,会返回一个代理对象,getEarlyBeanReference将返回一个代理后的对象，而不是实例化阶段创建的对象，这样就意味着B中注入的A将是一个代理对象而不是A的实例化阶段创建后的对象。

```java
public Object getEarlyBeanReference(Object bean, String beanName) {
        Object cacheKey = this.getCacheKey(bean.getClass(), beanName);
        this.earlyProxyReferences.put(cacheKey, bean);
        return this.wrapIfNecessary(bean, beanName, cacheKey)
}
```





