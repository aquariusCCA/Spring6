---
up:
  - "[[Java 国际化]]"
  - "[[Spring6 国际化]]"
url: https://juejin.cn/post/7094796759855005709#heading-19
---
# 先说一下什么是国际化

**简单理解，就是对于不同的语言，做出不同的响应。**

比如页面中有个填写用户信息的表单，有个姓名的输入框

浏览器中可以选择语言

选中文的时候会显示：

```shell
姓名：一个输入框
```

选英文的时候会显示：

```shell
Full name：一个输入框
```

国际化就是做这个事情的，根据不同的语言显示不同的信息。

所以需要支持国际化，得先知道选择的是哪种地区的哪种语言，java中使用`java.util.Locale`来表示地区语言这个对象，内部包含了国家和语言的信息。

Locale中有个比较常用的构造方法

```java
public Locale(String language, String country) {
    this(language, country, "");
}
```

2个参数：language (语言)、country (国家)

语言和国家这两个参数的值不是乱写的，国际上有统一的标准：

比如 language 的值：zh 表示中文，en 表示英语，而中文可能很多地区在用，比如大陆地区可以用：CN，新加坡用：SG；英语也是有很多国家用的，GB表示英国，CA表示加拿大

国家语言简写格式：language-country，如：zh-CN（中文【中国】），zh-SG（中文【新加坡】），en-GB（英语【英国】），en-CA（英语【加拿大】）。

还有很多，这里就不细说了，国家语言编码给大家提供一个表格：www.itsoku.com/article/282

Locale类中已经创建好了很多常用的Locale对象，直接可以拿过来用，随便列几个看一下：

```java
static public final Locale SIMPLIFIED_CHINESE = createConstant("zh", "CN"); //zh_CN
static public final Locale UK = createConstant("en", "GB"); //en_GB
static public final Locale US = createConstant("en", "US"); //en_US
static public final Locale CANADA = createConstant("en", "CA"); //en_CA
```

再回头看前面的问题：页面中显示姓名对应的标签，需要我们根据一个key及Locale信息来获取对应的国际化信息，spring中提供了这部分的实现，下面我们来看详情。

---

# Spring中国际化怎么用？

### MessageSource接口

spring中国际化是通过MessageSource这个接口来支持的

```java
org.springframework.context.MessageSource
```

内部有3个常用的方法用来获取国际化信息，来看一下

```java
public interface MessageSource {

    /**
     * 获取国际化信息
     * @param code 表示国际化资源中的属性名；
     * @param args用于传递格式化串占位符所用的运行参数；
     * @param defaultMessage 当在资源找不到对应属性名时，返回defaultMessage参数所指定的默认信息；
     * @param locale 表示本地化对象
     */
    @Nullable
    String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);

    /**
     * 与上面的方法类似，只不过在找不到资源中对应的属性名时，直接抛出NoSuchMessageException异常
     */
    String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;

    /**
     * @param MessageSourceResolvable 将属性名、参数数组以及默认信息封装起来，它的功能和第一个方法相同
     */
    String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException;
}
```

### 常见3个实现类

#### ResourceBundleMessageSource

这个是基于Java的ResourceBundle基础类实现，允许仅通过资源名加载国际化资源

#### ReloadableResourceBundleMessageSource

这个功能和第一个类的功能类似，多了定时刷新功能，允许在不重启系统的情况下，更新资源的信息

#### StaticMessageSource

它允许通过编程的方式提供国际化信息，一会我们可以通过这个来实现db中存储国际化信息的功能。

---

# Spring中使用国际化的3个步骤

通常我们使用spring的时候，都会使用带有ApplicationContext字样的spring容器，这些容器一般是继承了AbstractApplicationContext接口，而这个接口实现了上面说的国际化接口MessageSource，所以通常我们用到的ApplicationContext类型的容器都自带了国际化的功能。

通常我们在ApplicationContext类型的容器中使用国际化3个步骤
步骤一：创建国际化文件
步骤二：向容器中注册一个MessageSource类型的bean，bean名称必须为：messageSource
步骤三：调用AbstractApplicationContext中的getMessage来获取国际化信息，其内部将交给第二步中注册的messageSource名称的bean进行处理

### 创建国际化文件

> **国际化文件命名格式：名称_语言_地区.properties**
##### message.properties

> 这个文件名称没有指定Local信息，当系统找不到的时候会使用这个默认的

```properties
name=您的姓名 
personal_introduction=默认个人介绍:{0},{1}
```

##### message_zh_CN.properties：中文【中国】

```properties
name=姓名 
personal_introduction=个人介绍:{0},{1},{0}
```

##### message_en_GB.properties：英文【英国】

```properties
name=Full name 
personal_introduction=personal_introduction:{0},{1},{0}
```

### spring中注册国际化的bean

> 注意必须是MessageSource类型的，bean名称必须为messageSource，此处我们就使用ResourceBundleMessageSource这个类

```java
@Configuration  
public class MainConfig1 {  
  
    @Bean  
    public ResourceBundleMessageSource messageSource(){  
        ResourceBundleMessageSource messageSource = 
	        new ResourceBundleMessageSource();  
		// 设置资源文件的基名
        messageSource.setBasename("file:/Users/xiaoshilin/Downloads/Spring6-main/spring6-myCode/src/main/resources/message"); //@1
        // 设置默认编码
        messageSource.setDefaultEncoding("UTF-8");  
        return messageSource;  
    }  
}
```

@1：这个地方的写法需要注意，可以指定国际化配置文件的位置，格式：路径/文件名称,注意不包含 **【语言_国家.properties】** 含这部分

### 来个测试用例

```java
public class MessageSourceTest {

    @Test
    public void test1() {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(MainConfig1.class);
        context.refresh();
        //未指定Locale，此时系统会取默认的locale对象，本地默认的值中文【中国】，即：zh_CN
        System.out.println(context.getMessage("name", null, null));
        //CHINA对应：zh_CN
        System.out.println(context.getMessage("name", null, Locale.CHINA));
        //UK对应en_GB 
        System.out.println(context.getMessage("name", null, Locale.UK)); 
    }
}
```

### 运行输出

```shell
您的姓名 
姓名
Full name
```

### 动态参数使用

> 配置文件中的`personal_introduction`，个人介绍，比较特别，包含了`{0},{1}`这样一部分内容，这个就是动态参数，调用`getMessage`的时候，通过第二个参数传递过去，来看一下用法：
  
```java
@Test  
public void test2(){  
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();  
    context.register(MainConfig1.class);  
    context.refresh();  
    System.out.println(context.getMessage("personal_introduction", new String[]{"spring 高手", "java 高手"}, Locale.CHINA));  
    System.out.println(context.getMessage("personal_introduction", new String[]{"spring", "java"}, Locale.UK));  
}
```

---

# 监控国际化文件的变化

用`ReloadableResourceBundleMessageSource`这个类，功能和上面案例中的`ResourceBundleMessageSource`类似，不过多了个可以监控国际化资源文件变化的功能，有个方法用来设置缓存时间：

```java
public void setCacheMillis(long cacheMillis)
```

> -1：表示永远缓存
> 
> 0：每次获取国际化信息的时候，都会重新读取国际化文件
> 
> 大于0：上次读取配置文件的时间距离当前时间超过了这个时间，重新读取国际化文件

还有个按秒设置缓存时间的方法`setCacheSeconds`，和`setCacheMillis`类似

下面我们来案例

```java
@Configuration  
public class MainConfig1 {  
    @Bean  
    public MessageSource messageSource(){  
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();  
        // 设置资源文件的基名  
        messageSource.setBasename("file:/Users/xiaoshilin/Downloads/Spring6-main/spring6-myCode/src/main/resources/message");  
        // 设置默认编码  
        messageSource.setDefaultEncoding("UTF-8");  
        // 设置是否缓存  
        messageSource.setCacheMillis(1000); // 缓存1秒钟  
        return messageSource;  
    }  
}
```

message_zh_CN.properties中新增一行内容

```properties
address=上海
```

对应的测试用例

```java
@Test  
public void test3() throws InterruptedException {  
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();  
    context.register(MainConfig1.class);  
    context.refresh();  
  
    // 輸出兩次  
    for (int i = 0; i < 2; i++) {  
        System.out.println(context.getMessage("address", null, Locale.CHINA));  
        TimeUnit.SECONDS.sleep(10);  
    }  
}
```

上面有个循环，当第一次输出之后，修改一下`message_zh_CN.properties`中的address为`上海松江`，最后运行结果如下

```shell
上海
上海松江
```

> **使用注意：线上环境，缓存时间最好设置大一点，性能会好一些。**

---

# 国际化信息存在db中 從這開始


---