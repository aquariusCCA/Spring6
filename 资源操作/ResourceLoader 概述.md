---
up:
  - "[[Spring6 課程描述]]"
---
# Spring 提供如下两个标志性接口：

1. ResourceLoader ： 该接口实现类的实例可以获得一个 Resource 实例。

2.  ResourceLoaderAware ： 该接口实现类的实例将获得一个 ResourceLoader 的引用。

---

# 在 ResourceLoader 接口里有如下方法：

- Resource getResource（String location） ： 该接口仅有这个方法，用于返回一个 Resource 实例。ApplicationContext 实现类都实现 ResourceLoader 接口，因此 ApplicationContext 可直接获取 Resource 实例。

- ClassLoader getClassLoader(): 获取用于加载类的 ClassLoader 对象。这对于加载类路径下的资源非常有用。 

---

# 实现类

Spring 框架提供了多个实现 ResourceLoader 接口的类，包括：

DefaultResourceLoader: 默认的资源加载器，可用于加载类路径、文件系统和 URL 资源。

FileSystemResourceLoader: 用于加载文件系统中的资源。

ClassPathResourceLoader: 用于加载类路径下的资源。

ServletContextResourceLoader: 用于加载Web应用程序上下文中的资源。


---

