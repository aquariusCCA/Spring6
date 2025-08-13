---
up:
  - "[[Spring6 課程描述]]"
  - "[[基于XML管理Bean]]"
---
# 1. 概念

在Spring中可以通过配置bean标签的scope属性来指定bean的作用域范围，各取值含义参加下表：

|取值|含义|创建对象的时机|
|---|---|---|
|singleton（默认）|在IOC容器中，这个bean的对象始终为单实例|IOC容器初始化时|
|prototype|这个bean在IOC容器中有多个实例|获取bean时|

如果是在WebApplicationContext环境下还会有另外几个作用域（但不常用）：

| 取值      | 含义         |
| ------- | ---------- |
| request | 在一个请求范围内有效 |
| session | 在一个会话范围内有效 |

---

# 2. 创建 User 类

```java
package com.atguigu.spring6.bean;
public class User {

    private Integer id;

    private String username;

    private String password;

    private Integer age;

    public User() {
    }

    public User(Integer id, String username, String password, Integer age) {
        this.id = id;
        this.username = username;
        this.password = password;
        this.age = age;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", age=" + age +
                '}';
    }
}
```

这里也可以使用我们之前的 User 类，不影响后续的操作。

---

# 3. 配置bean

```xml
<!-- 配置 User 类 -->
<bean id="userSingleton" class="com.codermast.spring6.iocxml.bean.User" scope="singleton"/>
<bean id="userPrototype" class="com.codermast.spring6.iocxml.bean.User" scope="prototype"/>
```

---

# 4. 测试

```java
@Test
public void testBeanScope(){
    ApplicationContext ac = new ClassPathXmlApplicationContext("spring-scope.xml");
    User user1 = ac.getBean(User.class);
    User user2 = ac.getBean(User.class);
    System.out.println(user1==user2);
}
```

![[1755061827206_wUUm7ak85R.png]]