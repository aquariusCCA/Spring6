---
up:
  - "[[Spring6 課程描述]]"
  - "[[基于注解管理Bean]]"
---
单独使用@Autowired注解，**默认根据类型装配**。【默认是byType】

查看源码：

```java
package org.springframework.beans.factory.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {
    boolean required() default true;
}
```

源码中有两处需要注意：

- 第一处：该注解可以标注在哪里？
    - 构造方法上
    - 方法上
    - 形参上
    - 属性上
    - 注解上
- 第二处：该注解有一个required属性，默认值是true，表示在注入的时候要求被注入的Bean必须是存在的，如果不存在则报错。如果required属性设置为false，表示注入的Bean存在或者不存在都没关系，存在的话就注入，不存在的话，也不报错。
    

# 1. **属性注入**

UserDao接口

```java
public interface UserDao {
    public void print();
}
```

UserDaoImpl实现类

```java
@Repository
public class UserDaoImpl implements UserDao {

    @Override
    public void print() {
        System.out.println("Dao层执行结束");
    }
}
```

UserService接口

```java
public interface UserService {
    public void out();
}
```

UserServiceImpl实现类

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    @Override
    public void out() {
        userDao.print();
        System.out.println("Service层执行结束");
    }
}
```

UserController

```java
@Controller
public class UserController {

    @Autowired
    private UserService userService;

    public void out() {
        userService.out();
        System.out.println("Controller层执行结束。");
    }
}
```

测试

```java
@Test
public void testAnnotation() {
    ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
    UserController userController = context.getBean("userController", UserController.class);
    userController.out();
    logger.info("执行成功");
}
```

![[1755064663768_CIoyvZWrfD.png]]

> 以上构造方法和setter方法都没有提供，经过测试，仍然可以注入成功。

---

# 2. **Set 注入**

修改 UserServiceImpl 类

```java
@Service
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    @Autowired
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void out() {
        userDao.print();
        System.out.println("Service层执行结束");
    }
}
```

修改 UserController 类

```java
@Controller
public class UserController {

    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    public void out() {
        userService.out();
        System.out.println("Controller层执行结束。");
    }

}
```

---

# 3. **构造方法注入**

修改 UserServiceImpl 类

```java
// 增加构造方法
@Autowired
public UserServiceImpl(UserDao userDao) {
    this.userDao = userDao;
}
```

修改 UserController 类

```java
// 增加构造方法
@Autowired
public UserController(UserService userService) {
    this.userService = userService;
}
```

---

# 4. **形参上注入**

修改 UserServiceImpl 类

```java
public UserServiceImpl(@Autowired UserDao userDao) {
    this.userDao = userDao;
}
```

修改 UserController 类

```java
public UserController(@Autowired UserService userService) {
    this.userService = userService;
}
```

---

# 5. **只有一个构造函数，无注解**

修改 UserServiceImpl 类

```java
public UserServiceImpl(UserDao userDao) {
    this.userDao = userDao;
}
```

当有参数的构造方法只有一个时，@Autowired注解可以省略。

说明：有多个构造方法时呢？大家可以测试（再添加一个无参构造函数），测试报错

---

# 6. **@Autowired注解和@Qualifier注解联合**

添加 dao 层实现

```java
@Repository
public class UserDaoRedisImpl implements UserDao {

    @Override
    public void print() {
        System.out.println("Redis Dao层执行结束");
    }
}
```

测试：测试异常

```shell
expected single matching bean but found 2: userDaoImpl,userDaoRedisImpl
```

错误信息中说：不能装配，UserDao 这个 Bean 的数量等于 2

怎么解决这个问题呢？**当然要byName，根据名称进行装配了。**

修改 UserServiceImpl 类

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    @Qualifier("userDaoImpl") // 指定bean的名字
    private UserDao userDao;

    @Override
    public void out() {
        userDao.print();
        System.out.println("Service层执行结束");
    }
}
```

---

# **总结**

- @Autowired注解可以出现在：属性上、构造方法上、构造方法的参数上、setter方法上。

- 当带参数的构造方法只有一个，@Autowired注解可以省略。

- @Autowired注解默认根据类型注入。如果要根据名称注入的话，需要配合@Qualifier注解一起使用。

---