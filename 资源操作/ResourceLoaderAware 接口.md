---
up:
  - "[[Spring6 課程描述]]"
---
ResourceLoaderAware 接口实现类的实例将获得一个 ResourceLoader 的引用，ResourceLoaderAware 接口也提供了一个 setResourceLoader() 方法，该方法将由 Spring 容器负责调用，Spring 容器会将一个 ResourceLoader 对象作为该方法的参数传入。

如果把实现 ResourceLoaderAware 接口的 Bean 类部署在 Spring 容器中，Spring 容器会将自身当成 ResourceLoader 作为 setResourceLoader() 方法的参数传入。由于 ApplicationContext 的实现类都实现了 ResourceLoader 接口，Spring 容器自身完全可作为 ResorceLoader 使用。

