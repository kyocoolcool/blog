---
title: "Spring Boot Principle"
date: 2020-07-07T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

Spring boot 自動裝配和起步依賴到底如何辦到的
<!--more-->
### 起步依賴帶來什麼好處以及如何辦到的?
- 解決版本依賴問題  
因為library之間版本兼容問題，直到程式運作起來才能得知。
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
### SPI(Service Provider Interface)
在自動裝配之前，先了解SPI，SPI是Java提供的一套用來被第三方實現或者擴展的API，它可以用來啟用框架擴展和替換組件。
更具體來說SPI是基于`接口的編程＋策略模式＋配置文件`組合實現的動態加載機制。
系統設計的各個抽象，往往有很多不同的實現方案，在面向的對象的設計中，一般推薦模塊之間基于接口編程，模塊之間不對實現類進行硬編碼。
一但代碼里涉及具體的實現類，就違反了可拔插的原則，如果需要替換一種實現，就需要修改代碼。為了實現在模塊裝配的時候能在程序中動態指定，這就需要一種服務發現機制。
Java SPI就是提供這樣的一個機制：為某個接口尋找服務實現的機制。有點類似IOC的思想，就是將裝配的控制權移到程序之外，在模塊化設計中這個機制尤其重要。所以SPI的核心思想就是解耦。

SPI代碼操作範例
>代碼結構

{{< img src="/images/blog/20200708/20200708-post-1.png">}}

>IParseDoc.java
```java
package com.kyocoolcool;

public interface IParseDoc {
    void parse();
}
```

>ExcelParse.java
```java
package com.kyocoolcool;

/**
 * @author Chris Chen https://kyocoolcool.com
 * @version 1.0
 * @className ExcelParse
 * @description
 * @date 2020/7/8 9:55 AM
 **/

public class ExcelParse implements IParseDoc{
    @Override
    public void parse() {
        System.out.println("excel parse");
    }
}
```

>WordParse.java
```java
package com.kyocoolcool;

/**
 * @author Chris Chen https://kyocoolcool.com
 * @version 1.0
 * @className WordParse
 * @description
 * @date 2020/7/8 9:55 AM
 **/

public class WordParse implements IParseDoc{
    @Override
    public void parse() {
        System.out.println("word parse");
    }
}
```

>com.kyocoolcool.IParseDoc.property (必須是interface全類名)
```java
#實作的全類名
com.kyocoolcool.ExcelParse
```

>Main.java
```java
package com.kyocoolcool;

import java.util.ServiceLoader;

/**
 * @author Chris Chen https://kyocoolcool.com
 * @version 1.0
 * @className MainC
 * @description
 * @date 2020/7/8 9:56 AM
 **/

public class Main {
    public static void main(String[] args) {
        ServiceLoader<IParseDoc> iParseDocs = ServiceLoader.load(IParseDoc.class);
//        SPI機制取代
//        IParseDoc iParseDocs = new ExcelParse();
        for (IParseDoc iParseDoc : iParseDocs) {
            iParseDoc.parse();
        }
    }
}
```

>output
```
excel parse
```

### 為什麼可以不需要web.xml?
原因是因為servlet3.0提供一個接口，要求javax.servlet.ServletContainerInitializer定義在resources/META-INF/services下，實作的class必須實作onStartup()，
並能直接設定servlet,filter,listener。spring mvc 也是用相同方式實現的，所以在`spring-web`下也有一個相同配置。
操作範例
>MyServletContainerInitializer.java
```java
package com.kyocoolcool.servlet;

import com.kyocoolcool.MyFilter;
import com.kyocoolcool.MyListener;
import com.kyocoolcool.MyServlet;
import com.kyocoolcool.service.Animal;

import javax.servlet.*;
import javax.servlet.annotation.HandlesTypes;
import java.util.Set;

/**
 * @author Chris Chen https://kyocoolcool.com
 * @version 1.0
 * @className MyServletContainerInitializer
 * @description
 * @date 2020/7/8 2:20 PM
 **/
@HandlesTypes(Animal.class)//定義接口tomcat會自動將實作這接口的class實例化放入set
public class MyServletContainerInitializer implements ServletContainerInitializer {
    @Override
    public void onStartup(Set<Class<?>> set, ServletContext servletContext) throws ServletException {
        for (Class<?> aClass : set) {
            try {
                final Object o = aClass.newInstance();
                ((Animal) o).play();
            } catch (InstantiationException | IllegalAccessException e) {
                e.printStackTrace();
            }
        }
        //配置java ee 三元素
            servletContext.addListener(new MyListener());
            final ServletRegistration.Dynamic myServlet = servletContext.addServlet("myServlet", new MyServlet());
            myServlet.addMapping("/hello");
        final FilterRegistration.Dynamic myFilter = servletContext.addFilter("myFilter", new MyFilter());
        myFilter.addMappingForUrlPatterns(null, false, "/*");

    }
}
```

### 如何描述spring ioc 容器?
是一組集合:包含BeanDefinitionMap(BeanDefinition:包含屬性`class`,scope,`autoWired`,lazyInit,abstractFlag,dependOn,`constructionArgumentValue`...),
BeanFactoryPostProcessor(提供擴展BeanDefinition機會),BeanPostProcessor,存放bean對象單例緩存池，這些集合提供ioc和di功能。

###  spring boot 啟動類為什麼有@import 可以做什麼用?
1. 導入普通類:`@Import({InsertA.class})` 在InsertA.class上沒有加任何的annotation，但透過這種方式仍可以將它列入ioc容器控管。
2. 導入`@Import(MyImportBeanDefinitionRegister.class)` 這class實現ImportBeanDefinitionRegistrar。
3. 導入`@Import(MyImportSelector.class)`，使種方式可以一次導入多個bean。

### 自動配置好處以及原理?
透過spi機制去讀取META-INF/spring.factories中符合條件的配置類(bean definition)，解析bean definition,批量加載bean definition。
Spring boot 提供一個將繁雜且重複的樣板代碼簡化的開發框架，最主要原理是自動裝配概念，讓開發者專注在邏輯代碼上，同時縮短開發時程，也不會陷入配置的坑。
spring boot 自動配置是應用程式在啟動時的一個過程，考慮眾多因素才決定spring bean要配置哪個，以及不配置哪個。
 
舉例:
- Spring boot 會在`spring-boot-autoconfigure-2.1.8.RELEASE.jar`這jar包下的,META-INF/spring.factories，
`org.springframework.boot.autoconfigure.EnableAutoConfiguration` 這個key下的配置類名都會讀取，經過filter後放在cache中。
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

### 完全自定義裝配

- 覆蓋spring boot自動配置
覆蓋自動配置是加上@EnableXxx，例如:@EnableWebSecurity配置，會放棄原本Spring Security自動配置，變成完全可控制並自定義。

### 自動配置微調
若只是微調參數，而放棄Spring boot自動配置，而重新自定義配置是很不划算的，透過application.property或application.yml，
通過屬性外置配置來做屬性微調。
屬性外配置的幾種方式:
- 命令行参数
- JVM系统属性
- 操作系统环境变量
- 应用程序外的application.properties或是application.yml
- 打包在应用程序以内的application.properties或是application.yml

### 程式碼優化更彈性
透過定義properties，再注入而不是直接將屬性注入到業務邏輯中。
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

### profile使用
可根據運行環境不同選擇引入參數，若適用application.property，則可以透過application-{profile}.property，
若是共用屬性則在，application.property設定，最後使用`spring.profiles.active=production`，來描述運行哪個屬性檔(ex:production)。
若是在application.yml使用profile則是透過`---`符號來分割運行環境。

### 集成測試自動配置
若應用程序有透過spring配置並組成元件，則集成測試就需要spring配置及組裝元件。spring提供JUnit支持，目的是在測試程序裡加載spring應用程序上下文。
`@SpringApplicationConfiguration`相較`@ContextConfiguration` 更能完整加載spring boot，不僅加載應用程序上下文、開啟日誌、加載外部配置屬性。

### 測試Web應用程式
在spring boot測試Web應用程式提供兩種方式:
- Spring Mock MVC:能在一個近似真實的模擬servlet容器裡測試控制器，而不用啟動應用伺服器。
- Web集成測試:在嵌入式server容器裡啟動應用程式，在真正的應用服務器裡進行測試。

### 總結
理解spring boot魔法，需要讀到spring源碼，若能明白其中原理，魔法也就只是手法。同時也能欣賞設計模式並加以擴展運用。
