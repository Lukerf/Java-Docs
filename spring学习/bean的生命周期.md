### 1. Bean的生命周期

实例化 

属性赋值 -- >BeanPostProcessor.postProcessBeforeInitialization

初始化 --> （InitializingBean.afterPropertiesSet 或 @PostConstruct）

使用

销毁



### 1. BeanPostProcessor 

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



### 2. FactoryBean 

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



### 3. BeanFactoryPostProcessor 

BeanFactoryPostProcessor 可以在 Bean 实例化之前获取 Bean 的配置元数据，并允许用户对其修改。而 BeanPostProcessor 是在 Bean 初始化前后执行，它并不能修改 Bean 的配置信息。

```
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

