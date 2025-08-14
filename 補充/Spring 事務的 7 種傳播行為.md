---
up:
  - "[[事务属性：传播行为]]"
---
Spring 事務的 **7 種傳播行為（Propagation）**，也就是在方法呼叫時，**事務要怎麼跟「目前存在的事務」互動**。

---

# 🌱 1. `Propagation.REQUIRED`（預設）

**邏輯：**

> 有就用目前的事務，沒有就開一個新的。

**用法最多，因為是預設行為。**

## ✅ 範例

```java
@Transactional(propagation = Propagation.REQUIRED)
public void methodA() {
    methodB(); // 如果 B 也是 REQUIRED，會用同一個事務
}
```

- 如果外面已經有一個事務，這個方法就**加入現有事務**。
- 如果沒有，就自己建立一個。

---

# 🆕 2. `Propagation.REQUIRES_NEW`

**邏輯：**

> 無論有沒有事務，都掛起原本的事務，然後「自己開一個全新的事務」。

## ✅ 範例

```java
@Transactional(propagation = Propagation.REQUIRED)
public void outer() {
    saveMainData(); // 這個在事務 A
    subService.inner(); // 這裡會掛起 A，執行 B（新事務）
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void inner() {
    saveSubData(); // 這裡是事務 B
}
```

## 🧠 關鍵差異

即使 `inner()` 出錯拋出例外，只要你 try-catch 處理掉，**outer() 的事務 A 不會受影響**。

---

# 🔁 3. `Propagation.NESTED`

**邏輯：**

> 如果當前有事務，就開一個內部的子事務 Savepoint（可以部分回滾）。如果沒有，就等同於 REQUIRED。

## ✅ 範例

```java
@Transactional
public void outer() {
    saveA();
    try {
        inner(); // 這裡是「子事務」
    } catch (Exception e) {
        // 捕獲錯誤，讓外部繼續跑
    }
    saveB();
}

@Transactional(propagation = Propagation.NESTED)
public void inner() {
    saveSomethingWrong(); // 出錯了，只回滾這段
}
```

- 如果 `inner()` 出錯，會**只回滾 inner 部分**。
- `outer()` 的其他操作仍然保留（若 `PlatformTransactionManager` 支援 nested）。

---

# 🧭 4. `Propagation.SUPPORTS`

**邏輯：**

> 有事務就加入，沒有就「不開事務」，普通執行。

## ✅ 範例

```java
@Transactional
public void outer() {
    subService.readOnly(); // 這裡會有事務包著
}

@Transactional(propagation = Propagation.SUPPORTS)
public void readOnly() {
    queryOnly(); // 可加入事務，也可單獨跑
}
```

適合查詢邏輯，**有事務更好，沒有也能執行。**

---

# 🚫 5. `Propagation.NOT_SUPPORTED`

**邏輯：**

> 明確不要使用事務，如果目前有事務，就「暫停它」。

## ✅ 範例

```java
@Transactional
public void doLogic() {
    subService.noTxMethod(); // 掛起主事務，非事務方式執行
}

@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void noTxMethod() {
    // 執行一些非事務要求的操作
}
```

---

# ❗ 6. `Propagation.MANDATORY`

**邏輯：**

> 必須在一個已存在的事務中執行，否則就報錯。

## ✅ 範例

```java
public void doSomething() {
    subService.mustHaveTx(); // 沒有事務 → 直接報錯
}

@Transactional(propagation = Propagation.MANDATORY)
public void mustHaveTx() {
    // 僅能被事務包著呼叫
}
```

用途少，一般用來強制「**只能被事務控制的地方呼叫**」。

---

# ❌ 7. `Propagation.NEVER`

**邏輯：**

> 必須「沒有事務」，如果當前有事務，就報錯。

## ✅ 範例

```java
@Transactional
public void doSomething() {
    subService.mustBeNonTx(); // 有事務 → 這裡會報錯
}

@Transactional(propagation = Propagation.NEVER)
public void mustBeNonTx() {
    // 僅能在無事務情況執行
}
```

---

# 🧾 總結對照表

|傳播行為|有事務時|無事務時|
|---|---|---|
|`REQUIRED`（預設）|加入當前事務|新建事務|
|`REQUIRES_NEW`|掛起原事務，開新事務|開新事務|
|`NESTED`|子事務（Savepoint）|新建事務（等同 REQUIRED）|
|`SUPPORTS`|加入事務|非事務方式執行|
|`NOT_SUPPORTED`|掛起事務|非事務方式執行|
|`MANDATORY`|加入事務|**拋出異常**|
|`NEVER`|**拋出異常**|非事務方式執行|

---

# 補充：掛起事務（suspend transaction）是什麼意思？

在 Spring 事務管理中，**「掛起事務」**（suspend transaction）意思是：

> 暫時停止當前正在進行的事務，把它「存起來」不讓它繼續作用，等新的事務執行完後再恢復它。

這是為了讓某些方法（像是使用 `REQUIRES_NEW` 或 `NOT_SUPPORTED` 的方法）能夠在「沒有原本事務干擾」的情況下執行。

---

### 🔧 具體來說：

假設我們有一個主事務（A），然後呼叫另一個需要「新事務」的子方法（B）：

```java
@Transactional(propagation = Propagation.REQUIRED)
public void outer() {
    saveA();                  // 屬於事務 A
    subService.inner();       // inner 要求開啟新的事務 B
    saveC();                  // 回到事務 A
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void inner() {
    saveB(); // 屬於新的事務 B
}
```

### 🧠 運作順序：

1. `outer()` 開始，Spring 建立事務 **A**
2. 執行 `saveA()` → 在事務 A 裡
3. 呼叫 `inner()`，因為是 `REQUIRES_NEW`：
    - 👉 **掛起事務 A**
    - 👉 建立一個新事務 **B**
    - 執行 `saveB()` → 在事務 B 裡
    - 事務 B 提交/回滾
4. 回到 `outer()`：
    - 👉 **恢復被掛起的事務 A**
    - 執行 `saveC()` → 繼續在事務 A 裡

---

### 🧱 掛起 = 暫時封存

你可以把掛起想像成：

> 📦「先把原本的事務用箱子裝起來放旁邊，避免干擾其他工作，等處理完其他事務後再打開繼續用。」

---

### 🧪 實際效果（好處）

這麼做的好處是可以**控制每段資料處理的事務獨立性**：

- 若 `inner()` 發生錯誤，只會影響事務 B，不會影響事務 A。
- 能實現「局部失敗但整體繼續」的邏輯。

---

### ⛔ 注意

- 「掛起」只有在事務管理器（如 `DataSourceTransactionManager`）支援的情況下才有效。
- 如果不支援掛起，可能會拋出 `TransactionSuspensionNotSupportedException`。

---

如你有興趣，我可以幫你用一段 Spring Boot 測試程式碼展示「掛起與恢復」的真實效果。要嗎？