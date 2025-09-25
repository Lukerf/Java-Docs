### 1. 概念定义：

	1. JoinPoint: 可以被注入的业务代码/时机
	1. Pointcut：规则，通过pointcut可以确定哪些JoinPoint被织入代码
	1. Advice：代码织入，对JoinPoint的增强代码，有5个织入时机：Before,After,After returning(正常退出后),After throwing（抛出异常后）,Around（前后都可织入，也可以选择是否执行原有的正常逻辑，如果不执行原有逻辑，甚至可以用自己的返回值替代原有的返回值，甚至抛出异常）

### 2. 实践案例

```java
/**	
 * @Description AOP实践代码
 * @Date 2023/4/23 9:50
 * @Author YXG
 */
@Slf4j
@Aspect //声明为一个切面
@Component
public class TestAdvice {
    // 声明 pointcut,即匹配规则，满足规则的才能够执行advice。比较灵活，可以是注解去匹配，或者具体的某个方法，或者使用正则表达式
    @Pointcut("@annotation(geektime.spring.springbucks.GlobalErrorCatch)")
    private void globalCatch(){}

    // advice 对业务逻辑的增强
    @Before("globalCatch()")
    public void enhanceAddLog() throws  Throwable{
        log.info("excute before=====================12");

    }
}
```

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface GlobalErrorCatch {
}
```

```java
public class CoffeeService {  
    @Autowired
    private CoffeeRepository coffeeRepository;

    // joinpoint ,加了注解标识该方法应该被增强,在调用到该方法时会执行切面逻辑
    @GlobalErrorCatch
    public List<Coffee> findAllCoffee() {
        return coffeeRepository.findAll();
    }
}
```

在调用CoffeService时打印CoffeService对象，发现是以下的一个值。并不是CoffeeService对象，而是用的CGLIB代理的对象，由此引出代理实现原理。

![微信图片_20230423105647](D:\MyNote\图片\微信图片_20230423105647.png)

### 动态代理和静态代理

**静态代理**：在程序运行前代理类的.class文件就已经存在。而且每个委托类都要有一个对应的代理，如果类多的时候就很会冗余了

![640](D:\MyNote\图片\640.jpg)

**动态代理**：程序运行后通过反射创建生成字节码再由JVM加载而成

1. JDK代理：使用反射为目标对象创建代理类对象。但是有个问题是委托类必须是要实现一个接口，因为JDK动态代理是通过与委托类实现同样的接口，然后在实现的接口方法里进行增强的

   ```java
   public class ProxyFactory {
   
      private Object target;// 维护一个目标对象
   
      public ProxyFactory(Object target) {
          this.target = target;
      }
   
      // 为目标对象生成代理对象
      public Object getProxyInstance() {
          return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),
                  new InvocationHandler() {
   
                      @Override
                      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                          System.out.println("计算开始时间");
                          // 执行目标对象方法
                          method.invoke(target, args);
                          System.out.println("计算结束时间");
                          return null;
                      }
                  });
      }
   ```

   1. **loader**: 代理类的ClassLoader，最终读取动态生成的字节码，并转成 java.lang.Class 类的一个实例（即类），通过此实例的 newInstance() 方法就可以创建出代理的对象
   2. **interfaces**: 委托类实现的接口，JDK 动态代理要实现所有的委托类的接口
   3. **InvocationHandler**: 委托对象所有接口方法调用都会转发到 InvocationHandler.invoke()，在 invoke() 方法里我们可以加入任何需要增强的逻辑 主要是根据委托类的接口等通过反射生成的

2. Spring AOP提供的CGLib代理： 通过直接继承自委托类，但是不能代理final方法，final方法不能重写

   示例：

   ```java
    public class Target{
         public void f(){
             System.out.println("Target f()");
         }
         public void g(){
             System.out.println("Target g()");
         }
     }
     
    public class Interceptor implements MethodInterceptor {
        @Override
        public Object intercept(Object obj, Method method, Object[] args,    MethodProxy proxy) throws Throwable {
            System.out.println("I am intercept begin");
    //Note: 此处一定要使用proxy的invokeSuper方法来调用目标类的方法
            proxy.invokeSuper(obj, args);
            System.out.println("I am intercept end");
            return null;
        }
    }
    
    public class Test {
        public static void main(String[] args) {
        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "F:\\code");
             //实例化一个增强器，也就是cglib中的一个class generator
            Enhancer eh = new Enhancer();
             //设置目标类
            eh.setSuperclass(Target.class);
            // 设置拦截对象
            eh.setCallback(new Interceptor());
            // 生成代理类并返回一个实例
            Target t = (Target) eh.create();
            t.f();
            t.g();
        }
    }
   ```
   
   Fastclass机制分析：JDK动态代理通过反射来调用被拦截的方法，反射效率较低，所以cglib通过fastclass来实现对被拦截方法的调用，FastClass就是对一个类的方法建立索引，通过索引直接调用相应的方法。