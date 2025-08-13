---
up:
  - "[[Spring6 課程描述]]"
  - "[[基于XML管理Bean]]"
---
在通过 xml 方式获取 bean 之前，我们需要先导入对应的配置文件，这里我们是 beans.xml。构建 ApplicationContext 容器。

```java
// 导入 bean 的 xml 配置文件
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
```

# 在 xml 方式下获取 bean 对象的方式有三种：

### **根据 id 获取**

```java
// 1.根据 id 获取对象
User user1 = (User) context.getBean("user");
// 调用 user 对象的 run 方法
user1.run();
System.out.println("1 根据 id 获取的 User 对象" + user1);
```

### **根据 Class 获取**

```java
// 2.根据类型获取对象
User user2 = context.getBean(User.class);
user2.run();
System.out.println("2 根据 类型 获取的 User 对象" + user2);
```

### **同时根据 id 和 Class 获取**

```java
// 3.同时根据id和类型获取对象
User user3 = context.getBean("user", User.class);
user3.run();
System.out.println("3 同时根据 id 和 类型 获取的 User 对象" + user3);
```

![[1755055129030_J4pTfGE51K.png]]

---

# 注意

要注意在 bean 的配置文件中，如果定义了两个相同的类，并赋予了不同的 id，那么此时就无法仅依靠类型来创建对象。

```xml
<!-- 创建user -->
<bean id="user" class="com.codermast.spring6.iocxml.User"/>
<bean id="user" class="com.codermast.spring6.iocxml.User"/>
```

报错信息：

```shell
Exception in thread "main" org.springframework.beans.factory.NoUniqueBeanDefinition
Exception: No qualifying bean of type 'com.codermast.spring6.iocxml.User' 
available: expected single matching bean but found 2: user,user1
```

报错的意思就是，期望的应为单个匹配bean，但找到：user，user1 两个。

这个时候可以使用 id 或者 id 和类型同时使用的方式进行获取，只要保证唯一性，理论上就可以创建。

**是否可以根据接口类型来获取bean？**

- 如果接口的实现唯一，此时可根据接口类型来获取该实现类的Bean
- 如果接口的实现不唯一，那么久无法根据接口类型来获取该实现类的Bean

![[1755055129036_i2zuyCa4pl.png]]


---
# **结论**

根据类型来获取bean时，在满足 bean 唯一性的前提下，其实只是看：『对象 **instanceof** 指定的类型』的返回结果，只要返回的是 true 就可以认定为和类型匹配，能够获取到。

Java 中，instanceof 运算符用于判断前面的对象是否是后面的类，或其子类、实现类的实例。如果是返回true，否则返回false。也就是说：用instanceof关键字做判断时， instanceof 操作符的左右操作必须有继承或实现关系