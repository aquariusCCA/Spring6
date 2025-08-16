---
up:
  - "[[Spring6 課程描述]]"
---
创建 ApplicationContext 对象时，通常可以使用如下实现类：

（1） ClassPathXMLApplicationContext : 对应使用 ClassPathResource 进行资源访问。

（2）FileSystemXmlApplicationContext ： 对应使用 FileSystemResource 进行资源访问。

（3）XmlWebApplicationContext ： 对应使用 ServletContextResource 进行资源访问。

当使用 ApplicationContext 的不同实现类时，就意味着 Spring 使用响应的资源访问策略。

效果前面已经演示

