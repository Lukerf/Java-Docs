### 1. BeanFactory好FactoryBean

BeanFactory是容器，负责创建、配置和管理Bean实例，默认情况下BeanFactory是延迟加载Bean的，只有在真正需要时才会创建Bean实例

FactoryBean是一个接口，通过实现FactoryBean可以自定义Bean

### 2. BeanFactoryPostProcessor和BeanPostProcessor

BeanFactoryPostProcessor 是 Spring 容器加载 Bean 的定义之后以及 Bean 实例化之前执行，所以 BeanFactoryPostProcessor 可以在 Bean 实例化之前获取 Bean 的配置元数据，并允许用户对其修改。而 BeanPostProcessor 是在 Bean 初始化前后执行，它并不能修改 Bean 的配置信息。