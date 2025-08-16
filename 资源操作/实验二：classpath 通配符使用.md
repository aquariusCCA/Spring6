---
up:
  - "[[Spring6 課程描述]]"
---
`classpath*:`  前缀提供了加载多个 XML 配置文件的能力，当使用 `classpath*:` 前缀来指定 XML 配置文件时，系统将搜索类加载路径，找到所有与文件名匹配的文件，分别加载文件中的配置定义，最后合并成一个 ApplicationContext。

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath*:beans.xml");
System.out.println(ctx);
```

当使用 `classpath*:` 前缀时，Spring 将会搜索类加载路径下所有满足该规则的配置文件。

如果不是采用 `classpath*:` 前缀，而是改为使用 `classpath:前缀`，Spring 则只加载第一个符合条件的 XML 文件

> [!NOTE] **注意 ：**
> 
> `classpath*:` 前缀仅对 ApplicationContext 有效。实际情况是，创建 ApplicationContext 时，分别访问多个配置文件(通过 ClassLoader 的 getResource 方法实现)。因此，`classpath*:` 前缀不可用于 Resource。

