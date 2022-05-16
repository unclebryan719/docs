---
typora-copy-images-to: upload
---

## Spring源码总结

### IOC

#### Bean生命周期

```mermaid
graph LR
Bean创建-->初始化-->销毁
```

##### Bean的创建

- 单实例Bean：容器启动时创建
- 多实例Bean：第一次获取时创建

##### Bean初始化

###### 属性赋值

```java
// 给属性赋值
populateBean(beanName, mbd, instanceWrapper);
```

###### 初始化之前

​	调用BeanPostProcessor的postProcessBeforeInitialization方法

###### 自定义初始化方法

- 使用JSR250@PostConstruct注解，标注在初始化方法上（由于该注解的解析是通过BeanPostProcessor解析执行的，所以这种方式的初始化方法先执行）
- 实现InitializingBean接口，重写afterPropertiesSet方法

- 通过@Bean指定init方法

  初始化执行顺序

  ```mermaid
  graph LR
  PostConstruct注解-->InitializingBean-->Bean注解的init方法
  ```

  

###### 初始化之后

​	调用BeanPostProcessor的postProcessAfterInitialization方法

##### Bean销毁

###### 销毁的方式

- 使用JSR250@PreDestroy注解，标注在销毁方法上

- 实现DisposableBean接口，重新destroy方法

- 通过@Bean指定destroy方法

  销毁执行的顺序

  ```mermaid
  graph LR
  PreDestroy注解-->DisposableBean-->Bean注解的destroy方法
  ```

  

容器关闭时销毁，多实例Bean不会销毁，容器不管理多实例Bean



#### BeanPostProcessor

应用：

解析注解，并实现注解的功能，比如@Autowired、@Async等



### AOP

#### 核心注解

> `@EnableAspectJAutoProxy`

Tips: EnableXXX注解，关注该注解给容器注册了什么组件，研究组件是如何工作的

```java
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {
  ...
}
```



```java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	/**
	 * Register, escalate, and configure the AspectJ auto proxy creator based on the value
	 * of the @{@link EnableAspectJAutoProxy#proxyTargetClass()} attribute on the importing
	 * {@code @Configuration} class.
	 */
	@Override
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
    
    ......
  }
```

给容器中注册一个名字为`org.springframework.aop.config.internalAutoProxyCreator`,类型为`AnnotationAwareAspectJAutoProxyCreator`的bean的定义信息，加入到`BeanDefinitionRegistry`中

```java
public static final String AUTO_PROXY_CREATOR_BEAN_NAME =
      "org.springframework.aop.config.internalAutoProxyCreator";
```

```java
@Nullable
	private static BeanDefinition registerOrEscalateApcAsRequired(
			Class<?> cls, BeanDefinitionRegistry registry, @Nullable Object source) {
		......
		RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
		beanDefinition.setSource(source);
		beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
		beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
		return beanDefinition;
	}
```

![image-20220403205011466](https://tva1.sinaimg.cn/large/e6c9d24ely1h0wtp4nnzwj21la0bmmz8.jpg)

#### 核心类

> `AnnotationAwareAspectJAutoProxyCreator`

`AnnotationAwareAspectJAutoProxyCreator`继承关系

- 实现了`BeanPostProcessor`接口
- 实现了`BeanFactoryAware`接口

![AnnotationAwareAspectJAutoProxyCreator](https://tva1.sinaimg.cn/large/e6c9d24ely1h0wu37fs2qj227a0tgaex.jpg)



BeanPostProcessor创建流程

```mermaid
sequenceDiagram 
	autonumber
	Actor-->>AnnotationConfigApplicationContext: register
  Note right of Actor: 注册配置类 
	Actor-->>AnnotationConfigApplicationContext: refresh 
	Note right of Actor: 刷新容器
	AnnotationConfigApplicationContext-->>AbstractApplicationContext: refresh
	AbstractApplicationContext-->>PostProcessorRegistrationDelegate: registerBeanPostProcessors
	Note right of AbstractApplicationContext: 注册bean的后置处理器，拦截bean的创建
	PostProcessorRegistrationDelegate-->>DefaultListableBeanFactory: getBeanNamesForType
	Note right of PostProcessorRegistrationDelegate: 根据类型获取BeanPostProcessor的名字
	PostProcessorRegistrationDelegate-->>AbstractBeanFactory: getBean
	Note right of PostProcessorRegistrationDelegate: 根据BeanPostProcessor的名字获取Bean
  AbstractBeanFactory-->>AbstractBeanFactory: doGetBean
  AbstractBeanFactory-->>DefaultSingletonBeanRegistry: getSingleton
  DefaultSingletonBeanRegistry-->>DefaultSingletonBeanRegistry:singletonObjects
	Note right of DefaultSingletonBeanRegistry: 首次调用为空，调用下面的方法
	AbstractBeanFactory-->>AbstractAutowireCapableBeanFactory: createBean
	AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory: createBeanInstance
	Note right of AbstractBeanFactory: 创建Bean
	AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory: populateBean
	Note right of AbstractAutowireCapableBeanFactory: 给属性赋值
	AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory: initializeBean
	Note right of AbstractAutowireCapableBeanFactory: 创建初始化bean
	AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory: invokeAwareMethods 
	Note right of AbstractAutowireCapableBeanFactory: 处理实现Aware接口的Bean（设置BeanName、ClassLoader、BeanFactory）
	AbstractAutowireCapableBeanFactory-->>AnnotationAwareAspectJAutoProxyCreator: initBeanFactory
	Note right of AbstractAutowireCapableBeanFactory: 初始化Bean工厂
	AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory: applyBeanPostProcessorsBeforeInitialization 
	Note right of AbstractAutowireCapableBeanFactory: 执行所有BeanPostProcessor前置处理方法
	AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory: invokeInitMethods
	Note right of AbstractAutowireCapableBeanFactory: 执行bean的初始化方法
	AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory: applyBeanPostProcessorsAfterInitialization  
	Note right of AbstractAutowireCapableBeanFactory: 执行所有BeanPostProcessor后置处理方法
	AbstractAutowireCapableBeanFactory-->>PostProcessorRegistrationDelegate: addBeanPostProcessor 
	Note left of AbstractAutowireCapableBeanFactory: 在bean工厂中加入BeanPostProcessor实例
```



#### 代理对象的创建流程

> `createBean`创建代理对象或者真正的Bean对象

```mermaid
sequenceDiagram
autonumber
Actor-->>AbstractAutowireCapableBeanFactory:createBean

AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory:resolveBeforeInstantiation

AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory:applyBeanPostProcessorsBeforeInstantiation

AbstractAutowireCapableBeanFactory-->>AbstractAutoProxyCreator:postProcessBeforeInstantiation

AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory:applyBeanPostProcessorsAfterInitialization

AbstractAutowireCapableBeanFactory-->>AbstractAutoProxyCreator:postProcessAfterInitialization

AbstractAutoProxyCreator-->>AbstractAutoProxyCreator:wrapIfNecessary
Note right of AbstractAutoProxyCreator: 获取当前Bean可用的增强器，即切面方法

AbstractAutoProxyCreator-->>AbstractAutoProxyCreator: getAdvicesAndAdvisorsForBean
Note right of AbstractAutoProxyCreator: 获取可用的增强器并排序，返回一个拦截器数组

AbstractAdvisorAutoProxyCreator-->>AbstractAdvisorAutoProxyCreator:findEligibleAdvisors
Note right of AbstractAdvisorAutoProxyCreator: 找到可用的增强器

AspectJAwareAdvisorAutoProxyCreator-->>AbstractAdvisorAutoProxyCreator:findCandidateAdvisors
Note right of AspectJAwareAdvisorAutoProxyCreator: 找到所有候选的增强器

AbstractAdvisorAutoProxyCreator-->>AbstractAdvisorAutoProxyCreator:findAdvisorsThatCanApply
Note right of AbstractAdvisorAutoProxyCreator: 过滤增强器并返回

AbstractAutoProxyCreator-->>AbstractAutoProxyCreator:createProxy
Note right of AbstractAutoProxyCreator: 创建代理对象

ProxyFactory-->>DefaultAopProxyFactory:createAopProxy
Note right of ProxyFactory: 用代理工厂创建代理对象

DefaultAopProxyFactory-->>JdkDynamicAopProxy:JdkDynamicAopProxy
Note right of DefaultAopProxyFactory:创建JDK代理

DefaultAopProxyFactory-->>ObjenesisCglibAopProxy:ObjenesisCglibAopProxy
Note right of DefaultAopProxyFactory:创建Cglib代理

AbstractAutowireCapableBeanFactory-->>AbstractAutowireCapableBeanFactory: doCreateBean
Note right of AbstractAutowireCapableBeanFactory: 创建真正的Bean对象
```



#### 代理对象的执行流程

```mermaid
sequenceDiagram
autonumber
Actor-->>代理对象:xxx方法

代理对象-->>DynamicAdvisedInterceptor:intercept
Note right of 代理对象: 拦截代理对象的目标方法（DynamicAdvisedInterceptor是CglibAopProxy的内部类）

DynamicAdvisedInterceptor-->>AdvisedSupport:getInterceptorsAndDynamicInterceptionAdvice
Note right of DynamicAdvisedInterceptor: 根据代理工厂获取目标方法的拦截器链

DynamicAdvisedInterceptor-->>MethodProxy:invoke
Note right of DynamicAdvisedInterceptor: 如果没有拦截器链，则直接执行目标方法

DynamicAdvisedInterceptor-->>CglibMethodInvocation:CglibMethodInvocation:proceed
Note right of DynamicAdvisedInterceptor: 如果存在拦截器链，则创建一个CglibMethodInvocation对象并调用proceed方法

```





### Spring事务

> 1. 在配置类中开启事务管理 @EnableTransactionManagement
> 2. 在配置类中注册事务管理器 PlatformTransactionManager
> 3. 在需要事务的service上加上Transactional注解
