---
up:
  - "[[Spring6 課程描述]]"
  - "[[Resource 的实现类]]"
---
> 字节数组的 Resource 实现类。通过给定的数组创建了一个 ByteArrayInputStream。它对于从任何给定的字节数组加载内容非常有用，而无需求助于单次使用的 InputStreamResource。

---

 **`ByteArrayResource`** 把一段 `byte[]` 包裝成 Spring 的 `Resource`，每次呼叫 `getInputStream()` 都會回傳一個新的 `ByteArrayInputStream`，因此可被**重複讀取**，也能準確回報 `contentLength()`（等於陣列長度），`equals/hashCode` 也依據底層位元組內容來比對。這正是它相對 `InputStreamResource` 的關鍵優勢（後者通常**只能讀一次**，`isOpen()` 為 `true`，嘗試第二次讀會丟 `IllegalStateException`）。([Home](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/ByteArrayResource.html "ByteArrayResource (Spring Framework 6.2.10 API)"), [Home](https://docs.spring.vmware.com/spring-framework/docs/5.3.42/javadoc-api/org/springframework/core/io/InputStreamResource.html "InputStreamResource (Spring Framework 5.3.42 API)"))

# 什麼時候選 `ByteArrayResource`（而不是 `InputStreamResource`）

- 內容已在記憶體中（如你用 `ByteArrayOutputStream` 生成 Excel/PDF/CSV），而且**可能被多次讀取**（例如記錄長度、日誌、重試、或底層轉換器二次讀取）。
    
- 你需要把「記憶體中的內容」當成 Spring 通用 `Resource` 傳遞（HTTP 回應、郵件附件、屬性讀取等）。(
    
- 反之，如果資料很大且你想**串流**、避免整段進記憶體，或來源本來就是一次性串流，才考慮 `InputStreamResource`。官方文件也明說：若可用其他 `Resource`（像 `ByteArrayResource` 或檔案型）就**優先別用** `InputStreamResource`。
    

# 實戰場景與範例

## 1) Spring MVC 檔案下載（內容現做現傳）

把記憶體中的位元組當檔案下載回傳，支援多次讀取、長度正確。

```java
@GetMapping("/export")
public ResponseEntity<Resource> export() {
    byte[] bytes = generateExcel(); // 例如用 Apache POI 寫到 ByteArrayOutputStream
    ByteArrayResource resource = new ByteArrayResource(bytes);

    return ResponseEntity.ok()
        .contentType(MediaType.APPLICATION_OCTET_STREAM)
        .contentLength(bytes.length)
        .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"report.xlsx\"")
        .body(resource);
}
```

`contentLength()` 會回傳陣列長度，`getInputStream()` 每次都是新的 `ByteArrayInputStream`。

## 2) 寄送郵件附件（`JavaMailSender`）

`MimeMessageHelper.addAttachment()` 可吃 `InputStreamSource/Resource`；`ByteArrayResource` 非單次串流，符合 JavaMail 可能重複讀取的需求。

```java
MimeMessage mm = mailSender.createMimeMessage();
MimeMessageHelper helper = new MimeMessageHelper(mm, true, StandardCharsets.UTF_8.name());

helper.setTo("user@example.com");
helper.setSubject("月報表");
helper.setText("附件請查收");
helper.addAttachment("report.xlsx", new ByteArrayResource(bytes));

mailSender.send(mm);
```

官方 Javadoc 指出可傳 `InputStreamSource`（所有 Spring `Resource` 也適用），而 JavaMail 可能多次呼叫 `getInputStream()`——這正是 `ByteArrayResource` 的用武之地。

## 3) 以 Multipart 方式上傳（`RestTemplate` / `WebClient`）

有些伺服器端需要 `filename`。**重點：`ByteArrayResource` 預設 `getFilename()` 會是 `null`**，你需覆寫它，否則某些框架會出錯或無法判斷 Content-Type。([Stack Overflow](https://stackoverflow.com/questions/56287405/can-we-get-filename-from-bytearrayresource?utm_source=chatgpt.com "can we get filename from byteArrayResource?"), [GitHub](https://github.com/spring-projects/spring-framework/issues/15475?utm_source=chatgpt.com "NPE in RestTemplate with Resource implementation ..."))

```java
MultiValueMap<String, Object> body = new LinkedMultiValueMap<>();
ByteArrayResource filePart = new ByteArrayResource(bytes) {
    @Override public String getFilename() { return "report.xlsx"; }
};

body.add("file", filePart);

HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.MULTIPART_FORM_DATA);

restTemplate.postForEntity(uploadUrl, new HttpEntity<>(body, headers), String.class);
```

（社群與 issue 討論皆指出需覆寫 `getFilename()` 才能讓 multipart 正常帶檔名。）
## 4) 從記憶體載入屬性/設定

當你在程式中動態生成 `.properties` 內容，可直接用 `ByteArrayResource` 餵給 `PropertiesLoaderUtils.loadProperties(...)`。

```java
Properties props = PropertiesLoaderUtils.loadProperties(
    new ByteArrayResource(propsBytes) // 例如 "a=1\nb=2".getBytes(UTF_8)
);
```

`loadProperties(Resource)` 接受任意 `Resource`，因此不必落地成檔案。

## 5) 單元測試假資料

測試需要 `Resource` 但不想建臨時檔：

```java
Resource stub = new ByteArrayResource("hello".getBytes(StandardCharsets.UTF_8));
// 傳給被測服務，避免 IO 依賴
```

可重複讀取、測試更穩定（不受檔案系統影響）。

# 常見陷阱與判斷清單（重點）

- **沒有檔名**：`ByteArrayResource` 預設 `getFilename()` 為 `null`。做 **multipart 上傳** 或某些 **郵件 inline/附件** 場景，若框架靠檔名判斷 MIME，**務必覆寫 `getFilename()`**，否則可能 NPE 或內容型別錯誤。
    
- **記憶體佔用**：整段內容都在記憶體，不適合超大檔。若檔案很大且只需串流一次，考慮 `InputStreamResource` 或直接用檔案型 `Resource`。
    
- **可多次讀取**：`ByteArrayResource` 每次提供新的 `InputStream`，安全地被 converter、記錄器或重試機制「讀很多次」。
    
- **相等性與長度**：`equals/hashCode` 依據底層位元組內容；`contentLength()` 為陣列長度。這對快取鍵或日誌紀錄很可靠。
    

# 一圖流式的選擇建議

- **我手上已經是 `byte[]`，而且可能被多次讀？** → 用 `ByteArrayResource`。
    
- **我只有一次性來源（串流/網路/DB Blob），不會重讀？** → 勉強可用 `InputStreamResource`，但官方建議若有更適合的 `Resource` 就別用它。
    
- **需要檔名（multipart/郵件）？** → `ByteArrayResource` 要**覆寫 `getFilename()`**。
    

如果你接下來要做「報表匯出下載」「E-mail 附件」「REST 上傳檔案」這三種典型任務，`ByteArrayResource` 幾乎是記憶體工作流的首選；唯一要保持質疑的是**檔案大小與檔名需求**——大檔就串流，multipart 就覆寫 `getFilename()`。