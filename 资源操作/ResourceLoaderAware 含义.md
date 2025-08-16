---
up:
  - "[[Spring6 課程描述]]"
---
> ResourceLoaderAware 是一个 Spring Bean 接口，用于将 ResourceLoader 实例注入到实现该接口的 Bean 中。它定义了一个 setResourceLoader(ResourceLoader resourceLoader) 方法，Spring 容器在启动时将 ResourceLoader 实例作为参数传递给该方法。