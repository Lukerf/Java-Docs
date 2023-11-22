### 简介

解决POJO需要重写一些set和get方法，即减少编码量

### 常用注解

@Getter和@Setter

@NonNull



@builder : 修饰DO,在调用时可以通过建造者模式创建实体对象，比如通过builder创建DO

```java
// 比如Student类被@builder修饰，则可以通过builder构建实体对象
Student.builder()
               .sno( "001" )
               .sname( "admin" )
               .sage( 18 )
               .sphone( "110" )
               .build();
```

@SuperBuilder： @builder的升级版，可以支持对父类属性的构造
