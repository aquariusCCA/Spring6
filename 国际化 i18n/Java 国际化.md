---
up:
  - "[[Spring6 課程描述]]"
補充內容:
  - "[[常见国家、地区的语言编码]]"
  - "[[Spring 系列之国际化详解]]"
---
# Java 自身是支持国际化的

java.util.Locale用 于指定当前用户所属的语言环境等信息，java.util.ResourceBundle 用于查找绑定对应的资源文件。Locale包含了language信息和country信息，Locale创建默认locale对象时使用的静态方法：

```java
/**
 * This method must be called only for creating the Locale.*
 * constants due to making shortcuts.
 */
private static Locale createConstant(String lang, String country) {
	BaseLocale base = BaseLocale.createInstance(lang, country);
	return getInstance(base, null);
}
```

---

# 配置文件命名规则：  

**basename_language_country.properties** 

必须遵循以上的命名规则，java才会识别。

其中，basename 是必须的，语言和国家是可选的。这里存在一个优先级概念，如果同时提供了messages.properties 和 messages_zh_CN.propertes 两个配置文件，如果提供的 locale 符合en_CN，那么优先查找 messages_en_CN.propertes 配置文件，如果没查找到，再查找messages.properties 配置文件。最后，提示下，所有的配置文件必须放在 classpath 中，一般放在resources 目录下

---

# **实验：演示Java国际化**

**第一步: 建立語言的資源文件**

>  在 resource 目录下创建两个配置文件：messages_zh_CN.propertes、messages_en_GB.propertes、messages.properties

![[1755157154411_1yuinbccem.png]]

![[1755137210515_KiEbbDNokJ.png]]

在 messages_zh_CN.properties 中写：test=ZH test

在 messages_en_GB.properties 中写：test=GB test

在 messages.properties 中寫： test=默認內容


**第二步: 测试**

```java
import java.nio.charset.StandardCharsets;
import java.util.Locale;
import java.util.ResourceBundle;

public class Demo1 {

    public static void main(String[] args) {
        System.out.println(ResourceBundle.getBundle("messages",
                new Locale("en","GB")).getString("test"));

        System.out.println(ResourceBundle.getBundle("messages",
                new Locale("zh","CN")).getString("test"));
    }
}
```