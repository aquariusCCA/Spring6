---
up:
  - "[[Spring6 課程描述]]"
  - "[[Resource 的实现类]]"
---
Resource 的一个实现类，用来访问网络资源，它支持URL的绝对路径。

**http:** 该前缀用于访问基于HTTP协议的网络资源。

**ftp:** 该前缀用于访问基于FTP协议的网络资源

**file:** 该前缀用于从文件系统中读取资源

# **实验：访问基于HTTP协议的网络资源**

**创建一个maven子模块spring6-resources，配置Spring依赖（参考前面）**

```java
import org.springframework.core.io.UrlResource;

public class UrlResourcesDemo {
    public static void loadAndReadUrlResource(String path){
        // 创建一个 Resource 对象
        UrlResource url = null;
        try {
            url = new UrlResource(path);
            // 获取资源名
            System.out.println(url.getFilename());
            System.out.println(url.getURI());
            // 获取资源描述
            System.out.println(url.getDescription());
            //获取资源内容
            System.out.println(url.getInputStream().read());
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        //访问网络资源
        loadAndReadUrlResource("https://www.codermast.com");
    }
}
```

---

# **实验二：在项目根路径下创建文件，从文件系统中读取资源**

方法不变，修改调用传递路径

```java
public static void main(String[] args) {
    //1 访问网络资源
	//loadAndReadUrlResource("http://www..com");
    
    //2 访问文件系统资源
    loadAndReadUrlResource("file:codermast.txt");
}
```

当访问文件系统资源时，文件需要基于当前项目的根路径下来进行读取

![[1755157154417_Dii5JjgoBY.png]]