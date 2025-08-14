---
up:
  - "[[底层原理]]"
url: https://www.cnblogs.com/wlwl/p/10092494.html
---
Transactional 是 spring 中定义的事务注解，在方法或类上加该注解开启事务。主要是通过反射获取bean 的注解信息，利用AOP对编程式事务进行封装实现。

我们先写个 demo，感受它的加载过程。

# spring 事务注解：

![[1363214-20181209182520523-539178551.png]]

## 自定义一个注解

```java
/**
 * @Target  作用域(作用在方法上,类上,或是属性上)
 * @Retention 运行周期
 * @interface 定义注解
 */
@Target(value = ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    //自定义注解的属性
    int id() default 0;
    String name() default "默认名称";
    String[]arrays();
    String title() default "默认标题";
}
```

## 测试

```java
public class User {    
    @MyAnnotation(name="吴磊",arrays = {"2","3"})  
    public  void aMethod () {}  
  
    public  void bMethod () {}  
  
    public static void main(String[] args) throws ClassNotFoundException {  
        //1. 反射获取到类信息  
        Class<?> forName = Class.forName("com.atguigu.spring6.User");  
        //2. 获取到当前类（不包含继承）所有的方法  
        Method[] declaredMethods = forName.getDeclaredMethods();  
        //3. 遍历每个方法的信息  
        for (Method method : declaredMethods) {  
            System.out.println("方法名称:" + method.getName());  
            //4. 获取方法上面的注解  
            MyAnnotation annotation = method.getDeclaredAnnotation(MyAnnotation.class);  
            if(annotation == null) {  
                System.out.println("该方法上没有加注解....");  
            }else {  
                System.out.println("Id:" + annotation.id());  
                System.out.println("Name:" + annotation.name());  
                System.out.println("Arrays:" + annotation.arrays());  
                System.out.println("Title:" + annotation.title());  
            }  
            System.out.println("--------------------------");  
        }  
    }  
}
```

```shell
方法名称:main
该方法上没有加注解....
--------------------------
方法名称:aMethod
Id:0
Name:吴磊
Arrays:[Ljava.lang.String;@52cc8049
Title:默认标题
--------------------------
方法名称:bMethod
该方法上没有加注解....
--------------------------
```

> [!NOTE]  总结：
> 
> 通过上面这么一个小demo我们就能发现，反射获取到每一个方法的注解信息然后进行判断，如果这是@Transactional注解，spring就会开启事务。接下来我们可以按照这种思路基于上一篇博客的编程式事务自己实现一个事务注解。

---

# **手写注解事务：**

## 导包

```xml
<dependencies>  
    <!--spring context依赖-->  
    <!--当你引入Spring Context依赖之后，表示将Spring的基础依赖引入了-->  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-context</artifactId>  
        <version>6.0.2</version>  
    </dependency>  
    <!--spring aop依赖-->  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-aop</artifactId>  
        <version>6.0.2</version>  
    </dependency>    <!--spring aspects依赖-->  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-aspects</artifactId>  
        <version>6.0.11</version>  
    </dependency>  
    <!--spring jdbc  Spring 持久化层支持jar包-->  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-jdbc</artifactId>  
        <version>6.0.2</version>  
    </dependency>    <!-- MySQL驱动 -->  
    <dependency>  
        <groupId>mysql</groupId>  
        <artifactId>mysql-connector-java</artifactId>  
        <version>8.0.33</version>  
    </dependency>    <!-- 数据源 -->  
    <dependency>  
        <groupId>com.alibaba</groupId>  
        <artifactId>druid</artifactId>  
        <version>1.2.15</version>  
    </dependency>    <!--spring对junit的支持相关依赖-->  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-test</artifactId>  
        <version>6.0.2</version>  
    </dependency>    <!--junit5测试-->  
    <dependency>  
        <groupId>org.junit.jupiter</groupId>  
        <artifactId>junit-jupiter-api</artifactId>  
        <version>5.3.1</version>  
    </dependency>  
    <!--log4j2的依赖-->  
    <dependency>  
        <groupId>org.apache.logging.log4j</groupId>  
        <artifactId>log4j-core</artifactId>  
        <version>2.19.0</version>  
    </dependency>    
    <dependency>        
	    <groupId>org.apache.logging.log4j</groupId>  
        <artifactId>log4j-slf4j2-impl</artifactId>  
        <version>2.19.0</version>  
    </dependency>
</dependencies>
```

## 配置 spring.xml 文件

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xmlns:aop="http://www.springframework.org/schema/aop"  
       xmlns:tx="http://www.springframework.org/schema/tx"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans  
       http://www.springframework.org/schema/beans/spring-beans.xsd       http://www.springframework.org/schema/context       http://www.springframework.org/schema/context/spring-context.xsd       http://www.springframework.org/schema/aop       http://www.springframework.org/schema/aop/spring-aop.xsd       http://www.springframework.org/schema/tx       http://www.springframework.org/schema/tx/spring-tx.xsd">  
      
    <!--扫描 com.atguigu.spring6 包下的所有注解-->  
    <context:component-scan base-package="com.atguigu.spring6"/>  
  
    <!--  
    基于注解的AOP的实现：  
    1、将目标对象和切面交给IOC容器管理（注解+扫描）  
    2、开启AspectJ的自动代理，为目标对象自动生成代理  
    3、将切面类通过注解@Aspect标识  
    -->  
    <aop:aspectj-autoproxy />  
  
    <!-- 导入外部属性文件 -->  
    <context:property-placeholder location="classpath:jdbc.properties" />  
  
    <!-- 配置数据源 -->  
    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">  
        <property name="url" value="${jdbc.url}"/>  
        <property name="driverClassName" value="${jdbc.driver}"/>  
        <property name="username" value="${jdbc.user}"/>  
        <property name="password" value="${jdbc.password}"/>  
    </bean>  
    <!-- 配置 JdbcTemplate -->    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">  
        <!-- 装配数据源 -->  
        <property name="dataSource" ref="druidDataSource"/>  
    </bean>  
    <!-- 事务管理器 -->  
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">  
        <property name="dataSource" ref="druidDataSource"/>  
    </bean>  
    <!--  
        开启事务的注解驱动  
        通过注解@Transactional所标识的方法或标识的类中所有的方法，都会被事务管理器管理事务  
    -->  
    <!-- transaction-manager属性的默认值是transactionManager，如果事务管理器bean的id正好就是这个默认值，则可以省略这个属性 -->  
    <tx:annotation-driven transaction-manager="transactionManager" />  
</beans>
```

## 自定义事务注解 

> 通过反射解析方法上的注解，如果有这个注解就执行事务逻辑

```java
package com.atguigu.spring6;  
  
import java.lang.annotation.ElementType;  
import java.lang.annotation.Retention;  
import java.lang.annotation.RetentionPolicy;  
import java.lang.annotation.Target;  
  
//@Transaction可以作用在类和方法上, 我们这里只作用在方法上。  
@Target(value = ElementType.METHOD)  
@Retention(RetentionPolicy.RUNTIME)  
/**  
 * 自定义事务注解  
 */  
public @interface MyAnnotation {  
  
}
```

##  封装编程式事务

```java
package com.atguigu.spring6;  
  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.context.annotation.Scope;  
import org.springframework.jdbc.datasource.DataSourceTransactionManager;  
import org.springframework.stereotype.Component;  
import org.springframework.transaction.TransactionStatus;  
import org.springframework.transaction.interceptor.DefaultTransactionAttribute;  
  
@Component  
@Scope("prototype") // 申明为多例,解决线程安全问题。  
/**  
 * 手写编程式事务  
 */  
public class TransactionUtil {  
  
    // 全局接受事务状态  
    private TransactionStatus transactionStatus;  
      
    // 获取事务源  
    @Autowired  
    private DataSourceTransactionManager dataSourceTransactionManager;  
  
    // 开启事务  
    public TransactionStatus begin() {  
        System.out.println("开启事务");  
        transactionStatus = dataSourceTransactionManager.getTransaction(new DefaultTransactionAttribute());  
        return transactionStatus;  
    }  
  
    // 提交事务  
    public void commit(TransactionStatus transaction) {  
        System.out.println("提交事务");  
        if(dataSourceTransactionManager != null) dataSourceTransactionManager.commit(transaction);  
    }  
  
    // 回滚事务  
    public void rollback(TransactionStatus transaction) {  
        System.out.println("回滚事务...");  
        if(dataSourceTransactionManager != null) dataSourceTransactionManager.rollback(transaction);  
    }  
}
```

## 通过AOP封装事务工具类, 基于环绕通知和异常通知来触发事务。

```java
package com.atguigu.spring6;  
  
  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.AfterThrowing;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.reflect.MethodSignature;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Component;  
import org.springframework.transaction.TransactionStatus;  
  
import java.lang.reflect.Method;  
  
@Aspect// 申明为切面  
@Component  
/**  
 * 切面类封装事务逻辑  
 */  
public class AopTransaction {  
  
    @Autowired  
    private TransactionUtil transactionUtil;  
  
    private TransactionStatus transactionStatus;  
    /**  
     * 环绕通知 在方法之前和之后处理事情  
     * @param pjp 切入点  
     */  
    @Around("execution(* com.atguigu.spring6.*.*(..))")  
    public Object around(ProceedingJoinPoint pjp) throws Throwable {  
        // 1.获取方法的注解  
        MyAnnotation annotation = this.getMethodMyAnnotation(pjp);  
        // 2.判断是否需要开启事务  
        transactionStatus = begin(annotation);  
        // 3.调用目标代理对象方法  
        Object result = pjp.proceed();  
        // 4.判斷關閉事務  
        commit(transactionStatus);  
        return result;  
    }  
    /**  
     * 获取代理方法上的事务注解  
     * @param pjp 切入点  
     */  
    private MyAnnotation getMethodMyAnnotation(ProceedingJoinPoint pjp) throws Exception {  
        //1. 获取代理对对象的方法  
        String methodName = pjp.getSignature().getName();  
        //2. 获取目标对象  
        Class<?> classTarget = pjp.getTarget().getClass();  
        //3. 获取目标对象类型  
        Class<?>[] par = ((MethodSignature) pjp.getSignature()).getParameterTypes();  
        //4. 获取目标对象方法  
        Method objMethod = classTarget.getMethod(methodName, par);  
        //5. 获取该方法上的事务注解  
        MyAnnotation annotation = objMethod.getDeclaredAnnotation(MyAnnotation.class);  
        return annotation;  
    }  
    /** 开启事务  */  
    private TransactionStatus begin(MyAnnotation annotation) {  
        if(annotation == null) return null;  
        return transactionUtil.begin();  
    }  
    /** 关闭事务 */  
    private void commit(TransactionStatus transactionStatus) {  
        if(transactionStatus != null) transactionUtil.commit(transactionStatus);  
    }  
    /**  
     * 异常通知进行 回滚事务  
     */  
    @AfterThrowing("execution(* com.atguigu.spring6.*.*(..))")  
    public void afterThrowing() {  
        // 获取当前事务 直接回滚  
        //TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();  
        if(transactionStatus != null) transactionUtil.rollback(transactionStatus);  
    }  
}
```

## 编写dao层

```java
package com.atguigu.spring6;  
  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.jdbc.core.JdbcTemplate;  
import org.springframework.stereotype.Repository;  
  
/*  
 CREATE TABLE `t_users` (   `name` varchar(20) NOT NULL,   `age` int(5) DEFAULT NULL,   PRIMARY KEY (`name`) ) ENGINE=InnoDB DEFAULT CHARSET=utf8; */@Repository  
public class UserDao {  
    @Autowired  
    private JdbcTemplate jdbcTemplate;  
  
    public int add(String name, Integer age) {  
        String sql = "INSERT INTO t_users(NAME, age) VALUES(?,?);";  
        int result = jdbcTemplate.update(sql, name, age);  
        System.out.println("插入成功");  
        return result;  
    }  
}
```

## 编写service

```java
package com.atguigu.spring6;  
  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Service;  
  
@Service  
public class UserService {  
  
    @Autowired  
    private UserDao userDao;  
  
    // 加上事务注解  
    @MyAnnotation  
    public void add() {  
        userDao.add("test001", 20);  
        int i = 1 / 0;  
        userDao.add("test002", 21);  
          
//        // 如果非要try,那么出现异常不会被aop的异常通知监测到,必须手动在catch里面回滚事务。  
//        try {  
//            userDao.add("test001", 20);  
//            int i = 1 / 0;  
//            userDao.add("test002", 21);  
//        } catch (Exception e) {  
//            // 回滚当前事务  
//            System.out.println("回滚");  
//            TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();  
//        }  
    }  
  
    public void del() {  
        System.out.println("del...");  
    }  
}
```

## 测试

```java
@SpringJUnitConfig(locations = "classpath:spring.xml")  
public class TestMyTransaction {  
  
    @Autowired  
    UserService userService;  
  
    @Test  
    public void test01(){  
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");  
        UserService userService = (UserService) applicationContext.getBean("userService");  
        // aop()对userService类进行了拦截,添加自定义事务注解的方法会触发事务逻辑  
        userService.add();  
        // del()方法没有加注解，则什么也不会触发。  
        //userService.del();  
    }  
}
```