---
up:
  - "[[Spring6 課程描述]]"
補充內容:
  - "[[常见国家、地区的语言编码]]"
  - "[[Spring 系列之国际化详解]]"
---
# MessageSource接口

> [!NOTE]
> spring中国际化是通过MessageSource这个接口来支持的

## **常见实现类**

### **ResourceBundleMessageSource**

这个是基于Java的ResourceBundle基础类实现，允许仅通过资源名加载国际化资源

### **ReloadableResourceBundleMessageSource**

这个功能和第一个类的功能类似，多了定时刷新功能，允许在不重启系统的情况下，更新资源的信息

### **StaticMessageSource**

它允许通过编程的方式提供国际化信息，一会我们可以通过这个来实现db中存储国际化信息的功能。

---

# 使用Spring6国际化

## **第一步 创建资源文件**

**国际化文件命名格式：基本名称 _ 语言 _ 国家.properties**

**{0},{1}这样内容，就是动态参数**

**（1）创建 codermast_en_US.properties**

```
www.codermast.com=welcome {0},time:{1}
```

**（2）创建 codermast_zh_CN.properties**

```
www.codermast.com=欢迎 {0},时间:{1}
```


## **第二步 创建spring配置文件，配置MessageSource**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="messageSource"
          class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>codermast</value>
            </list>
        </property>
        <property name="defaultEncoding" value="utf-8"/>
    </bean>
</beans>
```

> ​ 在 Spring6 中，将 ResourceBundleMessageSource 的 id 命名 为messageSource，可以减少不必要的配置和显式引用，让Spring6实现自动注入

## **第三步 创建测试类**

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import java.util.Date;
import java.util.Locale;

public class Demo2 {

    public static void main(String[] args) {

        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");

        //传递动态参数，使用数组形式对应{0} {1}顺序
        Object[] objs = new Object[]{"codermast",new Date().toString()};

        //www.codermast.com为资源文件的key值,
        //objs为资源文件value值所需要的参数,Local.CHINA为国际化为语言
        String str = context.getMessage("www.codermast.com", objs, Locale.CHINA);
        System.out.println(str);
    }
}
```

---