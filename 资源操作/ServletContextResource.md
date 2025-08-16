---
up:
  - "[[Spring6 課程描述]]"
  - "[[Resource 的实现类]]"
---
> 这是 ServletContext 资源的 Resource 实现，它解释相关 Web 应用程序根目录中的相对路径。它始终支持流(stream)访问和 URL 访问，但只有在扩展 Web 应用程序存档且资源实际位于文件系统上时才允许 java.io.File 访问。无论它是在文件系统上扩展还是直接从JAR或其他地方（如数据库）访问，实际上都依赖于Servlet容器。

---

# 1. 背景

`ServletContextResource` 是 Spring Framework 提供的一個 `Resource` 實現類，
它的用途就是**用 `ServletContext` 去定位資源**，而不是用檔案系統的絕對路徑。

它解析的路徑是 **相對於 Web 應用程式根目錄（webapp/）** 的，例如 `/WEB-INF/config.properties`。

---

# 2. 例子

假設你的專案結構如下：

```
src/
 └─ main/
     ├─ webapp/
     │   ├─ index.jsp
     │   └─ WEB-INF/
     │       ├─ config.properties
     │       └─ views/
     │           └─ home.jsp
     └─ java/
         └─ com.example.demo
             └─ MyServlet.java
```

`config.properties` 放在 `/WEB-INF/` 裡。

---

# 3. 程式範例

```java
import jakarta.servlet.ServletContext;
import org.springframework.core.io.Resource;
import org.springframework.web.context.support.ServletContextResource;

import java.io.BufferedReader;
import java.io.InputStreamReader;

public class MyService {

    private final ServletContext servletContext;

    public MyService(ServletContext servletContext) {
        this.servletContext = servletContext;
    }

    public void readConfigFile() throws Exception {
        // 建立一個 ServletContextResource，路徑是相對於 webapp 根目錄
        Resource resource = new ServletContextResource(servletContext, "/WEB-INF/config.properties");

        // 讀取檔案內容
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(resource.getInputStream()))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println("Config: " + line);
            }
        }

        // 檢查是否可轉為 java.io.File（視情況而定）
        if (resource.isFile()) {
            System.out.println("檔案路徑：" + resource.getFile().getAbsolutePath());
        } else {
            System.out.println("資源不在檔案系統上（可能打包在 WAR 裡或 JAR 裡）");
        }
    }
}
```

---

# 4. 關鍵行為解釋

1. **支援 stream / URL 存取**
   即使這個資源是在 WAR 壓縮包裡，仍可以用 `getInputStream()` 或 `getURL()` 取得內容。

2. **不一定支援 File 物件**

   * 如果你的 WAR 是展開（exploded）部署在檔案系統上，例如 Tomcat 把它解壓到 `/opt/tomcat/webapps/myapp/`，那 `resource.getFile()` 就可用。
   * 如果是直接以 WAR 的壓縮形式部署，或是容器將資源存在資料庫、JAR 內，那就沒法取得 `File`。

3. **路徑解析依賴 Servlet 容器**

   * `/WEB-INF/config.properties` 是由容器（Tomcat、Jetty…）決定怎麼映射到實際位置。
   * 你不用關心物理路徑，`ServletContextResource` 會透過 `ServletContext.getResource()` 處理。

---

# 5. 實際應用場景

* **讀取 WebApp 的設定檔**（`/WEB-INF` 下的 properties、xml）
* **載入靜態檔案**（但不暴露在外部 URL，例如只給程式內部用）
* **在 Spring XML 配置中注入 WebApp 資源**：

  ```xml
  <bean id="configResource" class="org.springframework.web.context.support.ServletContextResource">
      <constructor-arg ref="servletContext"/>
      <constructor-arg value="/WEB-INF/config.properties"/>
  </bean>
  ```

---
