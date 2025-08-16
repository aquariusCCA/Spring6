---
up:
  - "[[Spring6 課程描述]]"
---
```java
public static void main(String[] args) {
    // 创建Spring应用上下文，从类路径中加载配置文件
    ApplicationContext ctx = new ClassPathXmlApplicationContext();
    //        通过ApplicationContext访问资源
    //        ApplicationContext实例获取Resource实例时，
    //        默认采用与ApplicationContext相同的资源访问策略
    Resource res = ctx.getResource("atguigu.txt");
    System.out.println(res.getFilename());
}
```

> Spring 将采用和 ApplicationContext 相同的策略来访问资源。也就是说，如果ApplicationContext 是 ClassPathXmlApplicationContext，res 就是 ClassPathResource 实例

```java
public static void main(String[] args) {
    // 创建Spring应用上下文，指定文件系统路径加载配置文件
    ApplicationContext ctx = new FileSystemXmlApplicationContext();
    Resource res = ctx.getResource("atguigu.txt");
    System.out.println(res.getFilename());
}
```

>  Spring 将采用和 ApplicationContext 相同的策略来访问资源。也就是说，如果 ApplicationContext 是 FileSystemXmlApplicationContext，res 就是 FileSystemResource 实例

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.core.io.Resource;

public class ResourceLoaderExample {

    public static void main(String[] args) {
        // 创建一个ApplicationContext容器对象，该对象会读取classpath（类路径）下的名为applicationContext.xml的配置文件
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        // 使用 classpath: 前缀指定使用 ClassPathResource 实现类
        Resource resource1 = context.getResource("classpath:config.properties");
        System.out.println("Resource 1: " + resource1.getClass().getSimpleName());

        // 使用 file: 前缀指定使用 FileSystemResource 实现类
        Resource resource2 = context.getResource("file:/path/to/file.txt");
        System.out.println("Resource 2: " + resource2.getClass().getSimpleName());

        // 使用 http: 前缀指定使用 UrlResource 实现类
        Resource resource3 = context.getResource("http://www.example.com");
        System.out.println("Resource 3: " + resource3.getClass().getSimpleName());
    }
}
```

> 使用 ApplicationContext 的 getResource() 方法获取了三个不同类型的资源，使用不同的前缀指定了使用不同的 Resource 实现类