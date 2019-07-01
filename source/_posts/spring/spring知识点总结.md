---
title: spring知识点总结
date: 2019-02-13 10:01:29
tags:
categories: 
- spring
---
### 1. bean生命周期
![](http://ww1.sinaimg.cn/large/007DdbdYly1g04kbgr6vlj31uo0tzdpq.jpg)


```
Spring 容器根据配置中的 Bean Definition(定义)中对bean进行实例化

spring将值和bean的引用引入到bean对应的属性中

如果bean实现了BeanNameAware接口,spring将bean的id传递给setBeanName方法

如果bean实现了BeanFactoryAware接口,spring将调用setBeanFactory方法引入beanFactory

如果bean实现了BeanApplicationContextAware接口,spring将调用setApplicationContext方法引入applicationContext

如果bean实现了BeanPostProcessor接口,spring将调用它们的postProcessBeforeInitialization方法 @PostConstruct注解的方法是由 CommonAnnotationBeanPostProcessor类处理,该类继承InitDestroyAnnotationBeanPostProcessor
bean初始化和销毁的一个处理器

如果bean实现了InitializingBean接口,spring将调用afterPropertiesSet方法

如果bean使用init-method声明了初始化方法，该方法会被调用

如果bean实现了BeanPostProcessor接口,spring将调用它们的postProcessAfterInitialization方法

此时bean已经就绪,将一直存在上下文中,直到上下文销毁

如果bean实现了DisposableBean接口,spring将调用destroy接口方法

如果bean使用destroy-method声明了销毁方法,则该方法被调用
```

### 2. 初始化方法,销毁方法执行的先后顺序
- 初始化
(postProcessor)@PostConstruct -> (InitializingBean)afterPropertiesSet -> init-method声明的方法
- 销毁
(postProcessor)@PreDestroy -> (DisposableBean)destory -> destroy-method声明的方法

### 3. spring容器 applicationContext和beanFactory的区别
Beanfactory 和 ApplicationContext是spring的两种IoC容器
- Beanfactory是简单的低级容器,提供IoC功能的实现,可以将它理解为一个map,通过getBean方法获取bean
- ApplicationContext是更高级的容器,抽象基类AbstractApplicationContext的子类AbstractRefreshableApplicationContext依赖DefaultListableBeanFactory,所以说IoC功能是beanFactory实现
  根据applicationContext继承的接口可以发现,有如下功能
  MessageSource:管理 message,实现国际化等功能。
  ApplicationEventPublisher:事件发布。
  ResourcePatternResolver:多资源加载。
  EnvironmentCapable:系统 Environment（profile + Properties）相关。
  Lifecycle:管理生命周期。
  Closable:关闭,释放资源
  InitializingBean:自定义初始化。
  BeanNameAware:设置 beanName 的 Aware 接口。
- spring IoC 两个步骤
    a. 加载配置文件，解析成 BeanDefinition 放在 Map 里。
    b. 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出 Class 对象进行实例化，同时，如果有依赖关系，将递归调用 getBean 方法 —— 完成依赖注入。
    实现机制:工厂模式加反射机制
 对比:

| BeanFactory            | ApplicationContext                                                                                                                        |
|:-----------------------|:------------------------------------------------------------------------------------------------------------------------------------------|
| 它使用懒加载             | 启动时全部加载                                                                                                                              |
| 它使用语法显式提供资源对象 | 它自己创建和管理资源对象                                                                                                                     |
| 不支持国际化             | 支持国际化                                                                                                                                 |
| 不支持基于依赖的注解      | 支持基于依赖的注解                                                                                                                          |
| XmlBeanFactory         | ClassPathXmlApplicationContext<br>FileSystemXmlApplicationContext<br>XmlWebApplicationContext<br>ConfigServletWebServerApplicationContext |

### Spring 框架中有哪些不同类型的事件？
五种标准的事件
- 上下文更新事件（ContextRefreshedEvent）
- 上下文开始事件（ContextStartedEvent）
- 上下文停止事件（ContextStoppedEvent）
- 上下文关闭事件（ContextClosedEvent）
- 请求处理事件（RequestHandledEvent）
### 4. spring中使用的设计模式
- 代理模式<br/>
    网关的设计就是使用的代理模式<br/>
    spring中AOP是基于jdk动态代理和动态字节码生成技术实现的,
- 策略模式

### 5. aop实现原理
>    aop底层是通过动态代理机制和字节码生成技术即cglib实现.jdk动态代理实现,代理的类必须实现接口;字节码生成则是通过生成代理类的一个子类,子类覆盖父类方法,不管代理类有没有实现接口,但是对final方法无法覆写.<br/>

* jdk动态代理

    ```
    // 1. 首先实现一个InvocationHandler，方法调用会被转发到该类的invoke()方法。
    public class DynamicProxyHandler implements InvocationHandler {
        private Object proxied;
        public DynamicProxyHandler(Object proxied) {
            System.out.println("dynamic proxy handler constuctor: " + proxied.getClass());
            this.proxied = proxied;
        }
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("dynamic proxy name: " + proxy.getClass());
            System.out.println("method: " + method.getName());
            System.out.println("args: " + Arrays.toString(args));
            
            Object invokeObject = method.invoke(proxied, args);
            if (invokeObject != null) {
                System.out.println("invoke object: " + invokeObject.getClass());
            } else {
                System.out.println("invoke object is null");
            }
            return invokeObject;
        }
    }
    // 2.动态代理
    ClassLoader classLoader = Interface.class.getClassLoader();
    Class<?>[] interfaces = new Class[] { Interface.class };
    InvocationHandler handler = new DynamicProxyHandler(realObject);
    Interface proxy = (Interface) Proxy.newProxyInstance(classLoader, interfaces, handler); 
     
    ```

>    loader，指定代理对象的类加载器
    interfaces，代理对象需要实现的接口，可以同时指定多个接口
    handler，方法调用的实际处理者，代理对象的方法调用都会转发到这里（注意1)
    newProxyInstance()会返回一个实现了指定接口的代理对象，对该对象的所有方法调用都会转发给InvocationHandler.invoke()方法。理解上述代码需要对Java反射机制有一定了解。动态代理神奇的地方就是代理对象是在程序运行时产生的，而不是编译期
    对代理对象的所有接口方法调用都会转发到InvocationHandler.invoke()方法，在invoke()方法里我们可以加入任何逻辑，比如修改方法参数，加入日志功能、安全检查功能等；之后我们通过某种方式执行真正的方法体，示例中通过反射调用了Hello对象的相应方法，还可以通过RPC调用远程方法。

- 动态字节码生成 cglib
   ```
   // 1. 首先实现一个MethodInterceptor，方法调用会被转发到该类的intercept()方法。
   class MyMethodInterceptor implements MethodInterceptor{
     ...
       @Override
       public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
           logger.info("You said: " + Arrays.toString(args));
           return proxy.invokeSuper(obj, args);
       }
   }
   // 2. 然后在需要使用HelloConcrete的时候，通过CGLIB动态代理获取代理对象。
   Enhancer enhancer = new Enhancer();
   enhancer.setSuperclass(HelloConcrete.class);
   enhancer.setCallback(new MyMethodInterceptor());
    
   HelloConcrete hello = (HelloConcrete)enhancer.create();
   System.out.println(hello.sayHello("I love you!"));
   ```
### 6. sping如何解决循转依赖
循环依赖现象:A 依赖 B，B 依赖 C，C 依赖 A,然后A初始化时发现依赖B,则创建B;B初始化发现依赖C,则创建C;C发现依赖A,又创建A,形成无限循环.如下图所示:
![](http://ww1.sinaimg.cn/large/007DdbdYly1g08ez3lvs3j308k06xdfv.jpg)
Spring 循环依赖的场景有两种：
* 构造器的循环依赖。
* field 属性的循环依赖。

> 对于构造器的循环依赖，Spring 是无法解决的，只能抛出 BeanCurrentlyInCreationException 异常表示循环依赖.
Spring 只解决 scope 为 singleton 的循环依赖。对于scope 为 prototype 的 bean ，Spring 无法解决，直接抛出 BeanCurrentlyInCreationException 异常
###### 从AbstractBeanFactory doGetBean方法开始,
`
Object sharedInstance = getSingleton(beanName);
`
```
@Override
public Object getSingleton(String beanName) {
    return getSingleton(beanName, true);
}

/**
 * Return the (raw) singleton object registered under the given name.
 * <p>Checks already instantiated singletons and also allows for an early
 * reference to a currently created singleton (resolving a circular reference).
 * @param beanName the name of the bean to look for
 * @param allowEarlyReference whether early references should be created or not
 * @return the registered singleton object, or {@code null} if none found
 */
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
getSingleton会从三个缓存中查,分别是:`singletonObjects、earlySingletonObjects、singletonFactories`
```
/**
 * Cache of singleton objects: bean name to bean instance.
 *
 * 存放的是单例 bean 的映射。
 *
 * 对应关系为 bean name --> bean instance
 */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

/**
 * Cache of singleton factories: bean name to ObjectFactory.
 *
 * 存放的是【早期】的单例 bean 的映射。
 *
 * 对应关系也是 bean name --> bean instance。
 *
 * 它与 {@link #singletonObjects} 的区别区别在，于 earlySingletonObjects 中存放的 bean 不一定是完整的。
 *
 * 从 {@link #getSingleton(String)} 方法中，中我们可以了解，bean 在创建过程中就已经加入到 earlySingletonObjects 中了，
 * 所以当在 bean 的创建过程中就可以通过 getBean() 方法获取。
 * 这个 Map 也是解决【循环依赖】的关键所在。
 **/
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

/**
  * Cache of early singleton objects: bean name to bean instance.
  *
  * 存放的是 early objects 的映射，可以理解为创建单例 bean 的 factory 。
  *
  * 对应关系是 bean name --> object
  */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```
这个三级缓存:
- 第一级为 singletonObjects
- 第二级为 earlySingletonObjects
- 第三级为 singletonFactories
> 首先，从一级缓存 singletonObjects 获取。
  如果，没有且当前指定的 beanName 正在创建，就再从二级缓存 earlySingletonObjects 中获取。
  如果，还是没有获取到且允许 singletonFactories 通过 #getObject() 获取，则从三级缓存 singletonFactories 获取。如果获取到，则通过其 #getObject() 方法，获取对象，并将其加入到二级缓存 earlySingletonObjects 中，并从三级缓存 singletonFactories 删除

**关键点：添加至三级缓存**
使用bean的无参构造函数创建对象，并将其添加至三级缓存，这个时候的bean没有属性填充，但是bean已经存在了。

**每一层缓存是 三级缓存中获取到 会添加至二级缓存,然后删除三级缓存,最终只会在一级缓存中存在**

>最终循环依赖 Spring 解决的过程：
- 首先 A 完成初始化第一步并将自己提前曝光出来（通过 ObjectFactory 将自己提前曝光），在初始化的时候，发现自己依赖对象 B，此时就会去尝试 get(B)，这个时候发现 B 还没有被创建出来
- 然后 B 就走创建流程，在 B 初始化的时候，同样发现自己依赖 C，C 也没有被创建出来
- 这个时候 C 又开始初始化进程，但是在初始化的过程中发现自己依赖 A，于是尝试 get(A)，这个时候由于 A 已经添加至缓存中（一般都是添加至三级缓存 singletonFactories ），通过 ObjectFactory 提前曝光，所以可以通过 ObjectFactory#getObject() 方法来拿到 A 对象，C 拿到 A 对象后顺利完成初始化，然后将自己添加到一级缓存中
- 回到 B ，B 也可以拿到 C 对象，完成初始化，A 可以顺利拿到 B 完成初始化。到这里整个链路就已经完成了初始化过程了







