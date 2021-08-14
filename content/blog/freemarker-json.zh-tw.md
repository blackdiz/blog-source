---
title: "Use FreeMarker Template to Output JSON"
date: 2021-07-06T01:28:39+08:00
Tags: ["backend", "server side", "java"]
Categories: ["note"]
---

首先我們先準備一個 FreeMarker 的 template: `test.fltj`, `fltj` 是我們自創給 JSON 格式 tempalte 的副檔名:
```
{
    "name": "test",
    ${content}
}
```
`${content}` 就是我們等會兒要動態設置的部分。

接著我們需要建立 FreeMarker 的 `Configruation`，這個 `Configuration` 理論上只需要建立一個：
```java
    Configuration configuration = new Configuration(Configuration.VERSION_2_3_25);
    // 這裡設定 JSON 範本的路徑
    configuration.setClassForTemplateLoading(this.getClass(), "/templates/json-templates");
    configuration.setDefaultEncoding("UTF-8");
    configuration.setTemplateExceptionHandler(TemplateExceptionHandler.RETHROW_HANDLER);
    configuration.setLogTemplateExceptions(false);
```

這裡我們使用 jackson 這個 library 來幫我們把 Java 物件轉成 JSON 格式的字串：
```java
Map<String, Object> data = new HashMap<>();
data.put("text", "test message");
data.put("age", 23);
data.put("boolean", true);
ObjectMapper mapper = new ObjectMapper();
// writerWithDefaultPrettyPrinter() 會排版輸出的結果
String jsonString = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(data);
```

最後我們把得到的 JSON 字串替換掉 `${content}` 的部分：
```java
Map<String, String> content = new HashMap<>();
content.put("content", jsonString);
Writer writer = new StringWriter();
Template template = configuration.getTemplate("test.fltj");
template.process(content, writer);
```
這樣就會把替換後的 JSON 輸出到 `StringWriter` 中。

但因為 FreeMarker 預設會 escape 特殊字元，例如 `&` 會改成 `&amp;`，`"` 會改成 `&quot;`，所以當我們要用動態產生的 JSON 字串替換掉 `${content}` 時，輸出的內容中的 `"` 都會被改成 `&quot;`，FreeMarker 其實針對不同類型的輸出有內建的一些格式類別預設是否要 escape 的行為設定，而 JSON 格式的預設格式類別是不會 escape 的，因此我們要做的是告訴 FreeMarker 只要 template 副檔名是 `json` 就套用 JSON 格式的行為，這裡要用到 `TemplateConfiguration`：
```java
TemplateConfiguration jsonConfiguration = new TemplateConfiguration();
// JSONOutputFormat.INSTANCE 即 JSON 格式的處理, 不會去 escape 掉特殊字元
jsonConfiguration.setOutputFormat(JSONOutputFormat.INSTANCE);

configuration.setTemplateConfigurations(
    new FirstMatchTemplateConfigurationFactory(
        new ConditionalTemplateConfigurationFactory(
            // 這裡設定副檔名是 fltj 就套用 JSON 格式設定
            new FileExtentionMatcher("fltj"), jsonConfiguration)
        )
    )
)
```
這樣輸出的最終結果 JSON 就不會 escape 特殊字元了。

---
參考資料：
1. [Apache FreeMarker document: Output formats](https://freemarker.apache.org/docs/dgui_misc_autoescaping.html#dgui_misc_autoescaping_outputformat)
2. [Apache FreeMarker document: Associating output formats with templates](https://freemarker.apache.org/docs/pgui_config_outputformatsautoesc.html)

如果有什麼想法或需要指正的地方，歡迎您留言或來信 😄