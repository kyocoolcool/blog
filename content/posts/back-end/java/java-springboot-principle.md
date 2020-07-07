---
title: "Spring Boot Principle"
date: 2020-05-08T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

Spring boot

## Spring Boot

## Feature

### 使用起步依賴
- 解決版本依賴問題  
library之間版本兼容問題，直到程式運作起來才能得知。
起步依賴本質就是透過Maven或Gradle在文件定義對其他library傳遞依賴，
spring-boot-starter-web，就是其中一個起步依賴，內含多個獨立的library，
而spring-boot 提供基於多種功能的起步依賴，可以依需要配置。
提供的各個library版本是經過兼容測試的，所以可以解決library衝突的問題。

- 若需要引入更新的library直接在文件聲明版本號，如下
```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.4.3</version>
</dependency>
```

- 若需要移除某個library，聲明exclusion，如下
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.fasterxml.jackson.core</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

### 使用自動配置
spring boot 自動配置是應用程式在啟動時的一個過程，考慮眾多因素才決定spring bean要配置哪個，以及不配置哪個。
舉例:
- spring的JdbcTemplate是否有在Classpath中，並且有DataSource bean則自動配置jdbcTemplate bean。
- Thymeleaf是否有在Classpath中，如果有則配置Thymeleaf的模板解析器、視圖解析器以及模板引擎。
所以每當啟動應用程式，spring boot都要做將近這樣200個決定，涵蓋安全、集成、持久化、Web開發等諸多方面，
所以自動配置就是讓我們減少煩雜的配置及樣板代碼。

根據spring提供的條件化註解，當作自動配置的判斷是否配置為spring bean，常見的註解如下

條件註解 | 配置生效條件
--- | --- | 
@ConditionalOnBean | 配置了某個特定Bean |
@ConditionalOnMissingBean | 沒有配置特定Bean | 
@ConditionalOnClass | Classpath里有指定的類 | 
@ConditionalOnMissingClass | Classpath里缺少指定的類 | 
@ConditionalOnExpression | 給定的Spring Expression Language（SpEL）表達式計算結果為true | 
@ConditionalOnJava | Java的版本匹配特定值或者一個范圍值 | 
@ConditionalOnJndi | 參數中給定的JNDI位置必須存在一個，如果沒有給參數，則要有JNDI InitialContext | 
@ConditionalOnProperty | 指定的配置屬性要有一個明確的值 | 
@ConditionalOnResource | Classpath里有指定的資源 | 
@ConditionalOnWebApplication | 這是一個Web應用程序 | 
@ConditionalOnNotWebApplication | 這不是一個Web應用程序 | 

### 自定義配置

#### 覆蓋spring boot自動配置
覆蓋自動配置是加上@EnableXxx，例如:@EnableWebSecurity配置，會放棄原本Spring Security自動配置，變成完全可控制並自定義。

### 通過屬性外置配置

#### 自動配置微調
若只是微調參數，而放棄Spring boot自動配置，而重新自定義配置是很不划算的，透過application.property或application.yml，
來做屬性微調。
程式碼優化，透過定義properties，再注入而不是直接將屬性注入到業務邏輯中。
> properties bean
```java
package readinglist;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties("amazon")
public class AmazonProperties {
    private String associateId;
    public void setAssociateId(String associateId) {
        this.associateId = associateId;
    }
    public String getAssociateId() {
        return associateId;
    }
}
```

> injection dependence bean
```java
package readinglist;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
@Controller
@RequestMapping("/")
public class ReadingListController {
    private ReadingListRepository readingListRepository;
    private AmazonProperties amazonProperties;
    @Autowired
    public ReadingListController(ReadingListRepository readingListRepository,AmazonProperties amazonProperties) {
        this.readingListRepository = readingListRepository;
        this.amazonProperties = amazonProperties;
    }
    @RequestMapping(method=RequestMethod.GET)
    public String readersBooks(Reader reader, Model model) {
        List<Book> readingList =
        readingListRepository.findByReader(reader);
            if (readingList != null) {
                model.addAttribute("books", readingList);
                model.addAttribute("reader", reader);
                model.addAttribute("amazonID", amazonProperties.getAssociateId());
            }
        return "readingList";
    }
    @RequestMapping(method=RequestMethod.POST)
    public String addToReadingList(Reader reader, Book book) {book.setReader(reader);
        readingListRepository.save(book);
        return "redirect:/";
    }
}
```

#### profile
可根據運行環境不同選擇引入參數，若適用application.property，則可以透過application-{profile}.property，
若是共用屬性則在，application.property設定，最後使用`spring.profiles.active=production`，來描述運行哪個屬性檔(ex:production)。
若是在application.yml使用profile則是透過`---`符號來分割運行環境。

## 測試

### 集成測試自動配置
若應用程序有透過spring配置並組成元件，則集成測試就需要spring配置及組裝元件。spring提供JUnit支持，目的是在測試程序裡加載spring應用程序上下文。
`@SpringApplicationConfiguration`相較`@ContextConfiguration` 更能完整加載spring boot，不僅加載應用程序上下文、開啟日誌、加載外部配置屬性。

### 測試Web應用程式
在spring boot測試Web應用程式提供兩種方式:
- Spring Mock MVC:能在一個近似真實的模擬servlet容器裡測試控制器，而不用啟動應用伺服器。
- Web集成測試:在嵌入式server容器裡啟動應用程式，在真正的應用服務器裡進行測試。
