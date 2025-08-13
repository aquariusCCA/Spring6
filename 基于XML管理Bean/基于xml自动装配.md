---
up:
  - "[[Spring6 課程描述]]"
  - "[[基于XML管理Bean]]"
---

> [!NOTE] 自动装配：
> 
> 根据指定的策略，在IOC容器中匹配某一个bean，自动为指定的bean中所依赖的类类型或接口类型属性赋值

# 1. 场景模拟

创建类UserController

```java
public class UserController {

    private UserService userService;

    public void setUserService(UserService userService) {
        this.userService = userService;
    }

    public void saveUser(){
        userService.saveUser();
    }
}
```

创建接口UserService

```java
public interface UserService {

    void saveUser();

}
```

创建类UserServiceImpl实现接口UserService

```java
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void saveUser() {
        userDao.saveUser();
    }
}
```

创建接口UserDao

```java
public interface UserDao {

    void saveUser();

}
```

创建类UserDaoImpl实现接口UserDao

```java
public class UserDaoImpl implements UserDao{
    @Override
    public void saveUser() {
        System.out.println("保存成功");
    }
}
```

# 2.配置bean

> 使用bean标签的autowire属性设置自动装配效果

## 自动装配方式：byType

byType：根据类型匹配IOC容器中的某个兼容类型的bean，为属性自动赋值

> 若在IOC中，没有任何一个兼容类型的bean能够为属性赋值，则该属性不装配，即值为默认值null

> 若在IOC中，有多个兼容类型的bean能够为属性赋值，则抛出异常NoUniqueBeanDefinitionException

`autowire-byType.xml`

```xml
<!--自动装配 ：byType 方式-->
<bean id="userController" class="com.codermast.spring6.iocxml.bean.autowire.controller.UserController"
    autowire="byType"/>

<bean id="userService" class="com.codermast.spring6.iocxml.bean.autowire.service.UserServiceImpl"
    autowire="byType"/>

<bean id="userDao" class="com.codermast.spring6.iocxml.bean.autowire.dao.UserDaoImpl"/>
```

## 自动装配方式：byName

byName：将自动装配的属性的属性名，作为bean的id在IOC容器中匹配相对应的bean进行赋值

`autowire-byName.xml`

```xml
<!--自动装配：byName 方式-->
<bean id="userController" class="com.codermast.spring6.iocxml.bean.autowire.controller.UserController"
    autowire="byName"/>

<bean id="userService" class="com.codermast.spring6.iocxml.bean.autowire.service.UserServiceImpl"
    autowire="byName"/>
<bean id="userServiceImpl" class="com.codermast.spring6.iocxml.bean.autowire.service.UserServiceImpl"
    autowire="byName"/>

<bean id="userDao" class="com.codermast.spring6.iocxml.bean.autowire.dao.UserDaoImpl"/>
<bean id="userDaoImpl" class="com.codermast.spring6.iocxml.bean.autowire.dao.UserDaoImpl"/>
```

---

# 3. 测试

```java
@Test
public void testAutoWireByXML() {
    ApplicationContext acByName = new ClassPathXmlApplicationContext("autowire-byName.xml");
    UserController userControllerByName = acByName.getBean(UserController.class);
    userControllerByName.saveUser();
    ApplicationContext acByType = new ClassPathXmlApplicationContext("autowire-byType.xml");
    UserController userControllerByType = acByType.getBean(UserController.class);
    userControllerByType.saveUser();
}
```

![[1755064663761_2EslOMpGSW.png]]