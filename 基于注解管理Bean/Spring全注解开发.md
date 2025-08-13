---
up:
  - "[[Spring6 課程描述]]"
  - "[[基于注解管理Bean]]"
---
全注解开发就是不再使用 spring 配置文件了，写一个配置类来代替配置文件。

```java
@Configuration
@ComponentScan("com.codermast.spring6")
public class Spring6Config {
}
```

测试类

```java
@Test
public void testAllAnnotation(){
    ApplicationContext context = new AnnotationConfigApplicationContext(Spring6Config.class);
    UserController userController = context.getBean("userController", UserController.class);
    userController.out();
    logger.info("执行成功");
}
```

![[1755064663768_uBXTnpLXzB.png]]