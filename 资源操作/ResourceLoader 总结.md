---
up:
  - "[[Spring6 課程描述]]"
---
Spring 将采用和 ApplicationContext 相同的策略来访问资源。也就是说，如果 ApplicationContext是 FileSystemXmlApplicationContext，res 就是 FileSystemResource 实例；如果ApplicationContext 是 ClassPathXmlApplicationContext，res 就是 ClassPathResource 实例

当 Spring 应用需要进行资源访问时，实际上并不需要直接使用 Resource 实现类，而是调用ResourceLoader 实例的 getResource() 方法来获得资源，ReosurceLoader 将会负责选择 Reosurce 实现类，也就是确定具体的资源访问策略，从而将应用程序和具体的资源访问策略分离开来

另外，使用 ApplicationContext 访问资源时，可通过不同前缀指定强制使用指定的ClassPathResource、FileSystemResource 等实现类

```java
Resource res = ctx.getResource("calsspath:bean.xml");
Resrouce res = ctx.getResource("file:bean.xml");
Resource res = ctx.getResource("http://localhost:8080/beans.xml");
```