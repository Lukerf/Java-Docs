Spring IoC Container ：容器,用来管理bean

bean:包装好了的Object

### 控制反转（Ioc)：

控制：bean的创建，管理的权限

反转：从自己创建bean，new一个对象。改为交给Spring容器去控制，这就是反转。

### 依赖注入（DI）

依赖：程序运行需要依赖外部的资源，提供程序内对象的所需要的数据、资源

注入：配置文件把资源从外部注入到内部，

### IOC和DI的区别： 

IOC是设计思想，DI是实践方式

### 实践例子：

![Image_20230410165634](D:\MyNote\图片\Image_20230410165634.png)

### 将一个类声明为Bean的注解有哪些

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 `Service` 层返回数据给前端页面。

### @Component和@Bean的区别是什么



### spring 单例bean和单例模式的区别：

单例创造的对象是整个jvm就一个，而spring单例bean是根据配置，默认为同一个，可以修改配置为每次新建



### 实现原理

工场方法+反射机制