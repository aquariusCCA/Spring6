---
up:
  - "[[Spring6 課程描述]]"
---
```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.FileSystemXmlApplicationContext;
import org.springframework.core.io.Resource;

public class Demo1 {

    public static void main(String[] args) {
        /*
         * 通过搜索文件系统路径下的xml文件创建ApplicationContext，
         * 但通过指定classpath:前缀强制搜索类加载路径
         * classpath:bean.xml
         * */
        ApplicationContext ctx =
                new ClassPathXmlApplicationContext("classpath:beans.xml");
        System.out.println(ctx);
        Resource resource = ctx.getResource("codermast.txt");
        System.out.println(resource.getFilename());
        System.out.println(resource.getDescription());
    }
}
```