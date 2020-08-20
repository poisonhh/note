# Spring

## Bean的作用域

- singleton：单例模式，在整个Spring IOC容器中，使用singleton定义的bean只有一个实例
- prototype：原先模式，每次通过容器的getBean方法获取prototype定义的bean时，都产生一个新的bean实例

**只有在Web应用中使用Spring时，request、session、global-session作用域才有效**

- request：对于每次HTTP请求，使用request定义的bean都将产生一个新实例，即每次HTTP请求将会产生不用的bean实例
- session：同一个session共享一个bean实例
- global-session：同session作用域不同的是，所有的session共享一个bean实例

## Bean的完整生命周期

### Bean的生命周期

- Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化
- Bean实例化后对将Bean的引入和值注入到Bean的属性中
- 如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
- 如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
- 如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将Bean所在应用上下文引用传入进来
- 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法
- 如果Bean实现了InitializingBean接口，Spring就将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
- 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法
- 此时，Bean已经准备就绪，可以被应用程序使用了，他们将一直驻留在应用上下文中，知道应用上下文被销毁
- 如果Bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method声明销毁方法，该方法也会被调用

### Bean完整的生命周期

————————————初始化————————————

- BeanNameAware.setBeanName() 在创建此bean的bean工厂中设置bean的名称，在普通属性设置之后调用，在InitializinngBean.afterPropertiesSet()方法之前调用
- `BeanClassLoaderAware.setBeanClassLoader()`: 在普通属性设置之后，InitializingBean.afterPropertiesSet()之前调用
- BeanFactoryAware.setBeanFactory() : 回调提供了自己的bean实例工厂，在普通属性设置之后，在InitializingBean.afterPropertiesSet()或者自定义初始化方法之前调用
- `EnvironmentAware.setEnvironment()`: 设置environment在组件使用时调用
- `EmbeddedValueResolverAware.setEmbeddedValueResolver()`: 设置StringValueResolver 用来解决嵌入式的值域问题
- `ResourceLoaderAware.setResourceLoader()`: 在普通bean对象之后调用，在afterPropertiesSet 或者自定义的init-method 之前调用，在 ApplicationContextAware 之前调用。
- `ApplicationEventPublisherAware.setApplicationEventPublisher()`: 在普通bean属性之后调用，在初始化调用afterPropertiesSet 或者自定义初始化方法之前调用。在 ApplicationContextAware 之前调用。
- `MessageSourceAware.setMessageSource()`: 在普通bean属性之后调用，在初始化调用afterPropertiesSet 或者自定义初始化方法之前调用，在 ApplicationContextAware 之前调用。
- ApplicationContextAware.setApplicationContext(): 在普通Bean对象生成之后调用，在InitializingBean.afterPropertiesSet之前调用或者用户自定义初始化方法之前。在ResourceLoaderAware.setResourceLoader，ApplicationEventPublisherAware.setApplicationEventPublisher，MessageSourceAware之后调用。
- `ServletContextAware.setServletContext()`: 运行时设置ServletContext，在普通bean初始化后调用，在InitializingBean.afterPropertiesSet之前调用，在 ApplicationContextAware 之后调用**注：是在WebApplicationContext 运行时**
- BeanPostProcessor.postProcessBeforeInitialization() : 将此BeanPostProcessor 应用于给定的新bean实例 在任何bean初始化回调方法(像是InitializingBean.afterPropertiesSet或者自定义的初始化方法）之前调用。这个bean将要准备填充属性的值。返回的bean示例可能被普通对象包装，默认实现返回是一个bean。
- BeanPostProcessor.postProcessAfterInitialization() : 将此BeanPostProcessor 应用于给定的新bean实例 在任何bean初始化回调方法(像是InitializingBean.afterPropertiesSet或者自定义的初始化方法)之后调用。这个bean将要准备填充属性的值。返回的bean示例可能被普通对象包装
- InitializingBean.afterPropertiesSet(): 被BeanFactory在设置所有bean属性之后调用(并且满足BeanFactory 和 ApplicationContextAware)。

————————————销毁————————————

在BeanFactory 关闭的时候，Bean的生命周期会调用如下方法:

- `DestructionAwareBeanPostProcessor.postProcessBeforeDestruction()`: 在销毁之前将此BeanPostProcessor 应用于给定的bean实例。能够调用自定义回调，像是DisposableBean 的销毁和自定义销毁方法，这个回调仅仅适用于工厂中的单例bean(包括内部bean)
- 实现了自定义的destory()方法

## Spring-bean的循环依赖以及解决方式

> 1. 什么是循环依赖？
> 2. 怎么检测循环依赖
> 3. Spring怎么解决循环依赖
> 4. Spring对于循环依赖无法解决的场景
> 5. Spring解决循环依赖的方式我们能够学到什么？

### 1. 什么是循环依赖

循环依赖其实就是循环引用，也就是两个或者两个以上的bean互相持有对方，最终形成闭环。比如A依赖于B，B依赖于C，C又依赖于A。

Spring中循环依赖场景有：

- 构造器的循环依赖
- field属性的循环依赖

### 2. 怎么检测是否存在循环依赖

检测循环依赖相对比较容易，Bean在创建的时候可以给该Bean打标，如果递归调用回来发现正在创建中的话，即说明了循环依赖了。

### 3. 三种循环依赖

- 构造器的循环依赖(Spring 无法解决)

 Spring容器会将每一个正在创建的Bean 标识符放在一个“当前创建Bean池”中，Bean标识符在创建过程中将一直保持在这个池中，因此如果在创建Bean过程中发现自己已经在“当前创建Bean池”里时将抛出BeanCurrentlyInCreationException异常表示循环依赖；而对于创建完毕的Bean将从“当前创建Bean池”中清除掉。

　　Spring容器先创建单例A，A依赖B，然后将A放在“当前创建Bean池”中，此时创建B,B依赖C ,然后将B放在“当前创建Bean池”中,此时创建C，C又依赖A， 但是，此时A已经在池中，所以会报错，因为在池中的Bean都是未初始化完的，所以会依赖错误 ，（初始化完的Bean会从池中移除）

- setter方式单例，默认方式
- setter方式原型，prototype

对于“prototype”作用域Bean，Spring容器无法完成依赖注入，因为“prototype”作用域的Bean，Spring容器不进行缓存，因此无法提前暴露一个创建中的Bean。

### 3. Spring怎么解决循环依赖

Spring的循环依赖的理论依据其实是基于Java的引用传递，当我们获取到对象的引用时，对象的field或者属性是可以延后设置的（但是构造器必须是在获取引用之前）。

Spring的单例对象的初始化主要分为三步：

- createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象
- populateBean：填充属性，这一步主要是多bean的依赖属性进行填充
- initializeBean：调用spring xml中的init方法

从上面讲述的单例bean初始化步骤我们可以知道，循环依赖主要发生在第一、第二部。也就是构造器循环依赖和field循环依赖。

那么我们要解决循环引用也应该从初始化过程着手，对于单例来说，在spring容器整个生命周期内，有且只有一个对象，所以很容易想到这个对象应该存在cache中，Spring为了解决单例的循环依赖问题，使用了三级缓存。

看源码可知，三级缓存只要指:

```java
/** Cache of singleton objects: bean name --> bean instance */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);

/** Cache of singleton factories: bean name --> ObjectFactory */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);

/** Cache of early singleton objects: bean name --> bean instance */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);
```

这三级缓存分别指：

- singletonFactories：单例对象工厂的cache
- earlySingletonObjects：提前曝光的单例对象的cache
- singletonObjects：单例对象的cache

我们在创建bean的时候，首先想到的是从cache中获取这个单例的bean，这个缓存就是singletonObjects。主要调用方法就是：

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```

上面的代码需要解释两个参数：

- isSingletonCurrentlyInCreation()判断当前单例bean是否正在创建中，也就是没有初始化完成(比如A的构造器依赖了B对象所以得先去创建B对象，或者在A的populateBean过程中依赖了B对象，得先去创建B对象，这时的A就是处于创建中的状态。)
- allowEarlyReference是否允许从singletonFactories中通过getObject拿到对象

分析getSingleton()的整个过程，Spring首先从一级缓存singletonObjects中获取。如果获取不到，并且对象正在创建中，就再从二级缓存earlySingletonObjects中获取。如果还是获取不到且允许singletonFactories通过getObject()获取，就从三级缓存singletonFactory.getObject()(三级缓存)获取，如果获取到了则：

```java
this.earlySingletonObjects.put(beanName, singletonObject);
this.singletonFactories.remove(beanName);
```

从singletonFactories中移除，并放入earlySingletonObjects中。其实也就是从三级缓存移动到了二级缓存。

从上面三级缓存的分析，我们可以知道，Spring解决循环依赖的诀窍就在于singletonFactories这个三级cache。这个cache的类型是ObjectFactory，定义如下：

```java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;
}
```

这个接口在下面被引用

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

这里就是解决循环依赖的关键，这段代码发生在createBeanInstance之后，也就是说单例对象此时已经被创建出来(调用了构造器)。这个对象已经被生产出来了，虽然还不完美（还没有进行初始化的第二步和第三步），但是已经能被人认出来了（根据对象引用能定位到堆中的对象），所以Spring此时将这个对象提前曝光出来让大家认识，让大家使用。

这样做有什么好处呢？让我们来分析一下“A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象”这种循环依赖的情况。A首先完成了初始化的第一步，并且将自己提前曝光到singletonFactories中，此时进行初始化的第二步，发现自己依赖对象B，此时就尝试去get(B)，发现B还没有被create，所以走create流程，B在初始化第一步的时候发现自己依赖了对象A，于是尝试get(A)，尝试一级缓存singletonObjects(肯定没有，因为A还没初始化完全)，尝试二级缓存earlySingletonObjects（也没有），尝试三级缓存singletonFactories，由于A通过ObjectFactory将自己提前曝光了，所以B能够通过ObjectFactory.getObject拿到A对象(虽然A还没有初始化完全，但是总比没有好呀)，B拿到A对象后顺利完成了初始化阶段1、2、3，完全初始化之后将自己放入到一级缓存singletonObjects中。此时返回A中，A此时能拿到B的对象顺利完成自己的初始化阶段2、3，最终A也完成了初始化，进去了一级缓存singletonObjects中，而且更加幸运的是，由于B拿到了A的对象引用，所以B现在hold住的A对象完成了初始化。

知道了这个原理时候，肯定就知道为啥Spring不能解决“A的构造方法中依赖了B的实例对象，同时B的构造方法中依赖了A的实例对象”这类问题了！因为加入singletonFactories三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决。

### 总结

Spring通过三级缓存加上“提前曝光”机制，配合Java的对象引用原理，比较完美地解决了某些情况下的循环依赖问题！

不要使用基于构造函数的依赖注入，可以通过以下方式解决：

- 在字段上使用@Autowired注解，让Spring决定在合适的时机注入

- 用基于setter方法的依赖注入。