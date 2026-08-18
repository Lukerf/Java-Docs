### 1. Bean的生命周期

##### 实例化

获取配置文件中Bean定义，Spring通过反射创建实例对象，

**属性赋值**

设置属性和依赖，例如处理标记在字段或 Setter 方法上的 `@Autowired`、`@Value`、`@Resource` 等注解。这里解决了bean的循环依赖问题

**初始化**

- Aware接口：BeanNameAware、ApplicationContextAware
- 初始化前：@PostConstruct、BeanPostProcessor.postProcessBeforeInitialization
- 初始化：InitializingBean.afterPropertiesSet()` 或自定义 `init-method
- 初始化后：BeanPostProcessor.postProcessAfterInitialization，Spring AOP就是在这里生成代理对象的，之后才放入单例池。

一句话总结：**Bean 的初始化阶段 = Aware 接口回调 + `BeanPostProcessor` 前置处理 + `@PostConstruct` + `InitializingBean.afterPropertiesSet()` + 自定义 `init-method` + `BeanPostProcessor` 后置处理（生成代理）。它的目的是在依赖注入完成后，让 Bean 完成自身的“准备动作”，进入可用状态。**需要关注的主要是BeanPostProcessor.postProcessAfterInitialization方法，Spring AOP就是在这里生成代理对象的，之后才放入单例池。



**使用**

**销毁**

DisposableBean.destroy 或 @PreDestroy用于自定义的销毁方法

### 2. 如何解决Bean的循环依赖

三级缓存

| 缓存级别     | 缓存名称                | 存储内容                                                     | 作用                                                 |
| :----------- | :---------------------- | :----------------------------------------------------------- | :--------------------------------------------------- |
| **一级缓存** | `singletonObjects`      | 完全初始化好的单例 Bean 对象（成品）                         | 存放最终可以直接使用的 Bean。                        |
| **二级缓存** | `earlySingletonObjects` | 提前暴露的单例 Bean 对象（半成品，已完成实例化但未完成属性注入和初始化） | 存放那些被提前引用、但尚未完成全部初始化的 Bean。    |
| **三级缓存** | `singletonFactories`    | `ObjectFactory` 对象工厂                                     | 用于生成 Bean 的早期引用，是解决循环依赖的关键所在。 |

### 

比如A依赖B,B依赖A

Spring 在创建A的时候，先通过反射创建实例化对象，然后放入到三级缓存singletonFactories中，开始属性赋值，发现依赖@Autowired的B，此时B还没有创建，从三级缓存中都获取不到，开始创建B，B也是创建实例对象，放入到三级缓存，属性赋值时去获取A的Bean,从三级缓存可以拿到A的Bean,因此会调用BeanFactory的getObject方法获取到A的引用，A也会进入二级缓存（getObject方法如果有AOP代理对象，会提前生成代理对象返回给B），这时B拿到A的引用以后就可以正常初始化了，进入到一级缓存，创建完成。A也继续往下执行，可以从一级缓存拿到B。

### 3. AOP是在什么时候创建的代理对象

BeanPostProcessor.postProcessAfterInitialization方法，Spring AOP就是在这里生成代理对象的，之后才放入单例池。

如果存在循环依赖，可能会提前生成代理对象，由B调用getEarlyBeanReference（）方法，并且放入到一个map中，后面初始化阶段从这个map读取，发现已经生成过代理对象了就不会再生成





### 4. BeanFactoryPostProcessor 

BeanFactoryPostProcessor 可以在 Bean 实例化之前获取 Bean 的配置元数据，并允许用户对其修改。而 BeanPostProcessor 是在 Bean 初始化前、后执行，它并不能修改 Bean 的配置信息。

```java
@Component
@Slf4j
public class RpcConsumerPostProcessor implements ApplicationContextAware, BeanClassLoaderAware, BeanFactoryPostProcessor {

    private ApplicationContext context;

    private ClassLoader classLoader;

    private final Map<String, BeanDefinition> rpcRefBeanDefinitions = new LinkedHashMap<>();

    @Override

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {

        this.context = applicationContext;

    }

    @Override

    public void setBeanClassLoader(ClassLoader classLoader) {

        this.classLoader = classLoader;

    }

    @Override

    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

        for (String beanDefinitionName : beanFactory.getBeanDefinitionNames()) {

            BeanDefinition beanDefinition = beanFactory.getBeanDefinition(beanDefinitionName);

            String beanClassName = beanDefinition.getBeanClassName();

            if (beanClassName != null) {

                Class<?> clazz = ClassUtils.resolveClassName(beanClassName, this.classLoader);

                ReflectionUtils.doWithFields(clazz, this::parseRpcReference);

            }

        }

        BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;

        this.rpcRefBeanDefinitions.forEach((beanName, beanDefinition) -> {

            if (context.containsBean(beanName)) {

                throw new IllegalArgumentException("spring context already has a bean named " + beanName);

            }

            registry.registerBeanDefinition(beanName, rpcRefBeanDefinitions.get(beanName));

            log.info("registered RpcReferenceBean {} success.", beanName);

        });

    }

    private void parseRpcReference(Field field) {

        RpcReference annotation = AnnotationUtils.getAnnotation(field, RpcReference.class);

        if (annotation != null) {

            BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(RpcReferenceBean.class);

            builder.setInitMethodName(RpcConstants.INIT_METHOD_NAME);

            builder.addPropertyValue("interfaceClass", field.getType());

            builder.addPropertyValue("serviceVersion", annotation.serviceVersion());

            builder.addPropertyValue("registryType", annotation.registryType());

            builder.addPropertyValue("registryAddr", annotation.registryAddress());

            builder.addPropertyValue("timeout", annotation.timeout());

            BeanDefinition beanDefinition = builder.getBeanDefinition();

            rpcRefBeanDefinitions.put(field.getName(), beanDefinition);

        }

    }

}
```





### 5. BeanPostProcessor 和InitializingBean	

Spring 的 BeanPostProcessor 接口给提供了对 Bean 进行再加工的扩展点，BeanPostProcessor 常用于处理自定义注解。在RpcProvider实例化的前后加入自定义的逻辑处理

```java
public class RpcProvider implements InitializingBean, BeanPostProcessor {
    // 省略其他代码
    private final Map<String, Object> rpcServiceMap = new HashMap<>();
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        RpcService rpcService = bean.getClass().getAnnotation(RpcService.class);
        if (rpcService != null) {
            String serviceName = rpcService.serviceInterface().getName();
            String serviceVersion = rpcService.serviceVersion();
            try {
                ServiceMeta serviceMeta = new ServiceMeta();
                serviceMeta.setServiceAddr(serverAddress);
                serviceMeta.setServicePort(serverPort);
                serviceMeta.setServiceName(serviceName);
                serviceMeta.setServiceVersion(serviceVersion);
                // TODO 发布服务元数据至注册中心
                rpcServiceMap.put(RpcServiceHelper.buildServiceKey(serviceMeta.getServiceName(), serviceMeta.getServiceVersion()), bean);
            } catch (Exception e) {
                log.error("failed to register service {}#{}", serviceName, serviceVersion, e);
            }
        }
        return bean;
    }
}
 @Override
 public void afterPropertiesSet() throws Exception {
        new Thread(() -> {
            try {
                startRpcServer();
            } catch (Exception e) {
                log.error("start rpc server error.", e);
            }
        }).start();
  }
```

### 6. BeanFactory

BeanFactory是容器，负责创建、配置和管理Bean实例，默认情况下BeanFactory是延迟加载Bean的，只有在真正需要时才会创建Bean实例

### 7. FactoryBean 

Spring 的 FactoryBean 接口可以帮助我们实现自定义的 Bean，FactoryBean 是一种特种的工厂 Bean，通过 getObject() 方法返回对象，而并不是 FactoryBean 本身。

```java
public class RpcReferenceBean implements FactoryBean<Object> {

    private Class<?> interfaceClass;

    private String serviceVersion;

    private String registryType;

    private String registryAddr;

    private long timeout;

    private Object object;

    @Override

    public Object getObject() throws Exception {

        return object;

    }

    @Override

    public Class<?> getObjectType() {

        return interfaceClass;

    }

    public void init() throws Exception {

        // TODO 生成动态代理对象并赋值给 object

    }

    public void setInterfaceClass(Class<?> interfaceClass) {

        this.interfaceClass = interfaceClass;

    }

    public void setServiceVersion(String serviceVersion) {

        this.serviceVersion = serviceVersion;

    }

    public void setRegistryType(String registryType) {

        this.registryType = registryType;

    }

    public void setRegistryAddr(String registryAddr) {

        this.registryAddr = registryAddr;

    }

    public void setTimeout(long timeout) {

        this.timeout = timeout;

    }

}
```



