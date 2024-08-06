昨天學了如何上傳圖片給 AI 辨識，今天我們讓 AI 來產生圖片吧
不果要生成圖片前先給大家看一下最近 API 測試的數據

![https://ithelp.ithome.com.tw/upload/images/20240806/20161290W523NsCxie.png](https://ithelp.ithome.com.tw/upload/images/20240806/20161290W523NsCxie.png)

多虧 gpt-4o mini 模型，測試了幾天才花 0.16 美金，可是有注意到 Image models 就佔了 0.08 美金嗎？而且我只產生一張圖，以實用性來說還是有點貴

如果要測試功能的話可以先找些便宜的平台測式，之後再進行切換才不會浪費冤枉錢，今天就教大家使用免費的模型來進行測試，下方是 Spring AI 有支援文生圖的 AI 供應商，扣掉 OpenAI 就只剩三家

![https://ithelp.ithome.com.tw/upload/images/20240806/20161290FAT9u7UJZS.png](https://ithelp.ithome.com.tw/upload/images/20240806/20161290FAT9u7UJZS.png)

Stability: 提供 25 個點數可供測試，用最舊的模型還能跑的百來張圖片，有興趣的也可以去測試看看

ZhiPuAI: 有個開發數限制，只要同時請求的數量沒太多就能一直測下去

QianFan: 非大陸手機無法註冊，放棄

拿最友善的 ZhiPuAI 測試吧，在官網 https://open.bigmodel.cn/ 註冊後點選由上方 API 密鑰就能取得 API Key

![https://ithelp.ithome.com.tw/upload/images/20240806/20161290qkPNJ0Ho4Y.png](https://ithelp.ithome.com.tw/upload/images/20240806/20161290qkPNJ0Ho4Y.png)

接下來要在 Spring 專案上加入 ZhiPuAI 的依賴，可以直接在 pom.xml 加上以下內容

```xml
<dependency>
  <groupId>org.springframework.ai</groupId>
  <artifactId>spring-ai-zhipuai-spring-boot-starter</artifactId>
</dependency>
```

完成後就可以設定 ZhiPuAI 的 Apikey了，把原本設定 openai 的地方改成 zhipuai 即可，key 就填官網上複製的

```yaml
spring:
  ai:
    zhipuai:
      api-key: ${ZHIPUAI_APIKEY}
```

下面就是程式的內容

```java
@RestController
@RequiredArgsConstructor
class AiController {
	private final ImageModel imageModel;
	@GetMapping("/image")
    public String image(String prompt) {
		ImageResponse response = imageModel.call(new ImagePrompt(prompt));
        Image image = response.getResult().getOutput();
        return String.format("<img src='%s' alt='%s'>", image.getUrl(), prompt);
    }
}
```

程式重點說明

- 文生圖的 Interface 模組是 ImageModel，因為上面引入 ZhiPuAI 的依賴，也設定了 apikey，SpringBoot 會自動建立 Bean 並且幫我們注入
- ImageModel 的方法比較單存，只有一個 call 呼叫，且只能傳入 ImagePrompt 變數（下方為原始碼）

```java
@FunctionalInterface
public interface ImageModel extends Model<ImagePrompt, ImageResponse> {
	ImageResponse call(ImagePrompt request);
}
```

- 回傳的 Image 包在 ImageResponse 中，可以透過 Image.getUrl() 取得網址，這類的文生圖的檔案都會在遠端伺服器

看看成果吧，使用 GetMapping 直接在瀏覽器輸入即可

![https://ithelp.ithome.com.tw/upload/images/20240806/20161290y9tHGVoXnR.png](https://ithelp.ithome.com.tw/upload/images/20240806/20161290y9tHGVoXnR.png)

雖然 ZhiPuAI 提供的選項不多，人物風格也不是我喜歡的（特別愛給中國古典美人圖），不過拿來練習還是蠻不錯的

有一點要特別注意，ZhiPuAI 會封鎖一些敏感的字眼有問題會出現下面的錯誤

```json
{
	"contentFilter":[{"level":2,"role":"user"}],
	"error":{
						"code":"1301",
						"message":"系统检测到输入或生成内容可能包含不安全或敏感内容，请您避免输入易产生敏感内容的提示语，感谢您的配合。"
					}
}
```

回顧一下今天學到甚麼:

- 文生圖的 ImageModel 調用
- 引入不同 AI 模組以及非 openai 的設定
- ImageModel 的引用
- 產生的圖檔取得
