---
up:
  - "[[Spring6 課程描述]]"
---
> Spring 的 Resource 接口位于 `org.springframework.core.io` 中。 旨在成为一个更强大的接口，用于抽象对低级资源的访问。以下显示了Resource接口定义的方法

```java
public interface Resource extends InputStreamSource {

    boolean exists();

    boolean isReadable();

    boolean isOpen();

    boolean isFile();

    URL getURL() throws IOException;

    URI getURI() throws IOException;

    File getFile() throws IOException;

    ReadableByteChannel readableChannel() throws IOException;

    long contentLength() throws IOException;

    long lastModified() throws IOException;

    Resource createRelative(String relativePath) throws IOException;

    String getFilename();

    String getDescription();
}
```

Resource 接口继承了 InputStreamSource 接口，提供了很多 InputStreamSource 所没有的方法。InputStreamSource 接口，只有一个方法：

```java
public interface InputStreamSource {
    InputStream getInputStream() throws IOException;
}
```

**其中一些重要的方法：**

getInputStream(): 找到并打开资源，返回一个InputStream以从资源中读取。预计每次调用都会返回一个新的InputStream()，调用者有责任关闭每个流

exists(): 返回一个布尔值，表明某个资源是否以物理形式存在  

isOpen: 返回一个布尔值，指示此资源是否具有开放流的句柄。如果为true，InputStream就不能够多次读取，只能够读取一次并且及时关闭以避免内存泄漏。对于所有常规资源实现，返回false，但是InputStreamResource除外。  

getDescription(): 返回资源的描述，用来输出错误的日志。这通常是完全限定的文件名或资源的实际URL。

**其他方法：**

isReadable(): 表明资源的目录读取是否通过getInputStream()进行读取。  

isFile(): 表明这个资源是否代表了一个文件系统的文件。  

getURL(): 返回一个URL句柄，如果资源不能够被解析为URL，将抛出IOException  

getURI(): 返回一个资源的URI句柄  

getFile(): 返回某个文件，如果资源不能够被解析称为绝对路径，将会抛出FileNotFoundException  

lastModified(): 资源最后一次修改的时间戳  

createRelative(): 创建此资源的相关资源  

getFilename(): 资源的文件名是什么 例如：最后一部分的文件名 myfile.txt

