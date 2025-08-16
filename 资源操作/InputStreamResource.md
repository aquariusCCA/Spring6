---
up:
  - "[[Spring6 課程描述]]"
  - "[[Resource 的实现类]]"
---
> InputStreamResource 是给定的输入流(InputStream)的 Resource 实现。它的使用场景在没有特定的资源实现的时候使用(感觉和@Component 的适用场景很相似)。与其他 Resource 实现相比，这是已打开资源的描述符。 因此，它的 isOpen() 方法返回 true。如果需要将资源描述符保留在某处或者需要多次读取流，请不要使用它。

---

# 1. 核心概念

* `InputStreamResource` 是 Spring 的一個 `Resource` 實現。
* 它不是去找檔案、URL、Classpath 這種「有來源可重複讀取」的資源，而是**直接包裝一個已經開啟的 `InputStream`**。
* 因為它包的是「已打開的流」，所以：

  * `isOpen()` 永遠是 `true`。
  * **不能重複讀取**（除非你自己重新打開新的流）。
  * 適合一次性傳輸（例如回應檔案下載）。

---

# 2. 範例：檔案下載 API

假設你有一個 REST API，要提供使用者下載一個檔案內容，檔案來源是你臨時從資料庫或壓縮檔中解壓出來的 `InputStream`，而且不需要也不能把它存成一個固定檔案路徑。

```java
import org.springframework.core.io.InputStreamResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.ByteArrayInputStream;
import java.io.InputStream;

@RestController
public class FileDownloadController {

    @GetMapping("/download")
    public ResponseEntity<InputStreamResource> downloadFile() {
        // 模擬從某個來源產生的檔案內容
        String fileContent = "Hello, this is a test file.";
        InputStream inputStream = new ByteArrayInputStream(fileContent.getBytes());

        // 用 InputStreamResource 包裝已打開的 InputStream
        InputStreamResource resource = new InputStreamResource(inputStream);

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment;filename=test.txt")
                .contentType(MediaType.TEXT_PLAIN)
                .contentLength(fileContent.length())
                .body(resource);
    }
}
```

---

# 3. 為什麼用 `InputStreamResource`？

* **其他 `Resource`（如 FileSystemResource、ClassPathResource）**：它們內部知道檔案位置，可以多次打開流。
* **`InputStreamResource`**：你已經有了一個「打開的流」，直接包起來給 Spring 用（例如 HTTP 回應）。
* **限制**：這個流**只能用一次**，因為流讀完就關閉了；不適合需要多次存取同一資源的情況。

---

# 4. 適用情境

* 動態生成檔案（不落地到檔案系統），直接輸出給客戶端。
* 從資料庫 `BLOB` 欄位讀取檔案內容，直接串流回傳。
* 從壓縮檔或外部 API 的回應流中讀取資料，直接包裝傳輸。

---

# 5. 不適用情境

* **需要多次讀取資源**（因為它的流無法重複打開）。
* **需要長期保存資源描述**（因為它不是檔案、不是 URL）。

---