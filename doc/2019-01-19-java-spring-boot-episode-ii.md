---
title: "Spring Boot Episode II"
author: [chrischen]
---
# 四、web開發

## 1、簡介

使用SpringBoot;

1)、創建SpringBoot應用，選中我們需要的模塊；

2)、SpringBoot已經默認將這些場景配置好了，只需要在配置文件中指定少量配置就可以運行起來

3)、自己編寫業務代碼

**自動配置原理？**

這個場景的SpringBoot幫我們配置了什麽？能不能修改？能修改那些配置？能不能擴展？xxx

```java
xxxAutoConfiguration:幫我們給容器中自動配置組件
xxxProperties:配置類來封裝配置文件的內容
```

## 2、靜態資源文件映射規則

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties implements ResourceLoaderAware, InitializingBean {
    //可以設置和靜態資源相關的參數，緩存時間等
```

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
   if (!this.resourceProperties.isAddMappings()) {
      logger.debug("Default resource handling disabled");
      return;
   }
   Integer cachePeriod = this.resourceProperties.getCachePeriod();
   if (!registry.hasMappingForPattern("/webjars/**")) {
      customizeResourceHandlerRegistration(registry
            .addResourceHandler("/webjars/**")
            .addResourceLocations("classpath:/META-INF/resources/webjars/")
            .setCachePeriod(cachePeriod));
   }
   String staticPathPattern = this.mvcProperties.getStaticPathPattern();
   if (!registry.hasMappingForPattern(staticPathPattern)) {
      customizeResourceHandlerRegistration(
            registry.addResourceHandler(staticPathPattern)
                  .addResourceLocations(
                        this.resourceProperties.getStaticLocations())
                  .setCachePeriod(cachePeriod));
   }
}
```

### 1、webjar

1)、所有的/webjars/**，都去classpath:/META-INF/resources/webjars/找資源；

```
webjars：以jar包的方式引入靜態資源
```

http://www.webjars.org/

![post-12](../images/blog/20190119/20190119-post-12.jpg)

localhost:8080/webjars/jquery/3.3.1/jquery.js

### 2、本地資源

```
private String staticPathPattern = "/**";
```

訪問任何資源

2、會在這幾文件夾下去找靜態路徑（靜態資源文件夾）

```
"classpath:/META-INF/resources/", 
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/",
"/";當前項目的根路徑
```

![post-13](../images/blog/20190119/20190119-post-13.jpg)

localhost:8080/abc ==>去靜態資源文件夾中找abc

![post-14](../images/blog/20190119/20190119-post-14.jpg)

3、index頁面歡迎頁，靜態資源文件夾下所有的index.html頁面；被“/**”映射；

localhost:8080/  -->index頁面

```JAVA
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(
      ResourceProperties resourceProperties) {
   return new WelcomePageHandlerMapping(resourceProperties.getWelcomePage(),
         this.mvcProperties.getStaticPathPattern());
}
```

4、喜歡的圖標，即網站title的圖標favicon

```java
@Configuration
@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
public static class FaviconConfiguration {

   private final ResourceProperties resourceProperties;

   public FaviconConfiguration(ResourceProperties resourceProperties) {
      this.resourceProperties = resourceProperties;
   }

   @Bean
   public SimpleUrlHandlerMapping faviconHandlerMapping() {
      SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
      mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
       //把任何favicon的圖標都在靜態文件夾下找
      mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
            faviconRequestHandler()));
      return mapping;
   }

   @Bean
   public ResourceHttpRequestHandler faviconRequestHandler() {
      ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
      requestHandler
            .setLocations(this.resourceProperties.getFaviconLocations());
      return requestHandler;
   }

}
```

可以在配置文件配置靜態資源文件夾

```properties
spring.resources.static-locations=classpath:xxxx
```

## 3、模板引擎

將html和數據 結合到一起 輸出組裝處理好的新文件

SpringBoot推薦Thymeleaf;語法簡單，功能強大

### 1、引入thymeleaf 3

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

默認導入thymeleaf2，版本太低 所以使用thymeleaf3.

[官方導入辦法](https://docs.spring.io/spring-boot/docs/1.5.12.RELEASE/reference/htmlsingle/#howto-use-thymeleaf-3)

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <!--thymeleaf 3的導入-->
    <thymeleaf.version>3.0.9.RELEASE</thymeleaf.version>
    <!--布局功能支持 同時支持thymeleaf3主程序 layout2.0以上版本  -->
    <!--布局功能支持 同時支持thymeleaf2主程序 layout1.0以上版本  -->
    <thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
</properties>
```

### 2、Thymeleaf使用和語法

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

   private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");

   private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");

   public static final String DEFAULT_PREFIX = "classpath:/templates/";

   public static final String DEFAULT_SUFFIX = ".html";
   //只要把HTML文件方法類路徑下的template文件夾下，就會自動導入
```

只要把HTML頁面放到classpath:/templates/,thymeleaf就能自動渲染；

使用：

1、導入thymeleaf的名稱空間

```html
<html xmlns:th="http://www.thymeleaf.org">    
```

2、使用thymeleaf語法；

```html
<!DOCTYPE html>
<html lang="en"  xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8" />
    <title>success</title>
</head>
<body>
<h1>success</h1>
<!--th:text 將div里面的文本內容設置為-->
<div th:text="${Lion}">
前端數據
</div>
</body>
</html>
```

3、語法規則

1）、th:text="${hello}"可以使用任意標簽 替換原生的任何屬性

**在SpringBoot的環境下**

```html
<div id="testid" class="testcalss" th:id="${Lion}" th:class="${Lion}" th:text="${Lion}">
	前端數據
</div>
```

![post-15-01](../images/blog/20190119/20190119-post-15-01.jpg)

**直接訪問HTML頁面**

![post-15-02](../images/blog/20190119/20190119-post-15-02.jpg)

**2)、內聯寫法註意需要在body上加上 th:inline="text"敲黑板**

不然不起作用

```html
<body class="text-center" th:inline="text"></body>
```

th標簽的訪問優先級

Order Feature Attributes

### 3、語法規則

|      | 功能                            | 標簽                                 | 功能和jsp對比                             |
| ---- | ------------------------------- | ------------------------------------ | ----------------------------------------- |
| 1    | Fragment inclusion              | th:insert th:replace                 | include(片段包含)                         |
| 2    | Fragment iteration              | th:each                              | c:forEach(遍歷)                           |
| 3    | Conditional evaluation          | th:if th:unless th:switch th:case    | c:if(條件判斷)                            |
| 4    | Local variable definition       | th:object  th:with                   | c:set(聲明變量)                           |
| 5    | General attribute modification  | th:attr th:attrprepend th:attrappend | 屬性修改支持前面和後面追加內容            |
| 6    | Specific attribute modification | th:value th:href th:src ...          | 修改任意屬性值                            |
| 7    | Text (tag body modification)    | th:text th:utext                     | 修改標簽體內容utext：不轉義字符<h1>大標題 |
| 8    | Fragment specification          | th:fragment                          | 聲明片段                                  |
| 9    | Fragment removal                | th:remove                            |                                           |

 

```properties
Simple expressions:(表達式語法)
    Variable Expressions: ${...}
    	1、獲取對象屬性、調用方法
    	2、使用內置基本對象：
    	    #ctx : the context object.
            #vars: the context variables.
            #locale : the context locale.
            #request : (only in Web Contexts) the HttpServletRequest object.
            #response : (only in Web Contexts) the HttpServletResponse object.
            #session : (only in Web Contexts) the HttpSession object.
            #servletContext : (only in Web Contexts) the ServletContext object.
         3、內置一些工具對象
        	#execInfo : information about the template being processed.
        	#messages : methods for obtaining externalized messages inside variables expressions, in the same way as they
            would be obtained using #{…} syntax.
            #uris : methods for escaping parts of URLs/URIs
            #conversions : methods for executing the configured conversion service (if any).
            #dates : methods for java.util.Date objects: formatting, component extraction, etc.
            #calendars : analogous to #dates , but for java.util.Calendar objects.
            #numbers : methods for formatting numeric objects.
            #strings : methods for String objects: contains, startsWith, prepending/appending, etc.
            #objects : methods for objects in general.
            #bools : methods for boolean evaluation.
            #arrays : methods for arrays.
            #lists : methods for lists.
            #sets : methods for sets.
            #maps : methods for maps.
            #aggregates : methods for creating aggregates on arrays or collections.
            #ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).
    Selection Variable Expressions: *{...} //選擇表達式：和${}功能一樣，補充功能
   # 配合th:object使用，object=${object} 以後獲取就可以使用*{a}  相當於${object.a}
  	    <div th:object="${session.user}">
            <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
            <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
            <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
		</div>
    Message Expressions: #{...} //獲取國際化內容
    Link URL Expressions: @{...} //定義URL鏈接
    	#<a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a>
    Fragment Expressions: ~{...}//片段文檔
    
Literals（字面量）
    Text literals: 'one text' , 'Another one!' ,…
    Number literals: 0 , 34 , 3.0 , 12.3 ,…
    Boolean literals: true , false
    Null literal: null
    Literal tokens: one , sometext , main ,…
Text operations:(文本操作)
    String concatenation: +
    Literal substitutions: |The name is ${name}|
Arithmetic operations:（數學運算）
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
Boolean operations:（布爾運算）
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
Comparisons and equality:（比較運算）
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
Conditional operators:（條件運算）
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
Special tokens:（空操作）
	No-Operation: _
```

inline寫法

```html
[[]] -->th:text
[()] -->th:utext
```



## 4、SpringMVC自動配置

### 1、SpringMVC的自動導入

[Spring框架](https://docs.spring.io/spring-boot/docs/1.5.12.RELEASE/reference/htmlsingle/#boot-features-developing-web-applications)

自動配置好了mvc：

以下是SpringBoot對SpringMVC的默認

Spring Boot provides auto-configuration for Spring MVC that works well with most applications.

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

  - 自動配置了ViewResolver(視圖解析器：根據方法的返回值得到視圖對象（View）,視圖對象決定如何渲染（轉發？重定向？）)
  - `ContentNegotiatingViewResolver`組合所有視圖解析器
  - 如何定制：我們可以自己給容器中添加一個視圖解析器；自動將其整合進來

- Support for serving static resources, including support for WebJars (see below).靜態資源

- Static `index.html` support.

- Custom `Favicon` support (see below).

- 自動註冊 了`Converter`, `GenericConverter`, `Formatter` beans.

  - `Converter`：類型轉換 文本轉為字面量

  - `Formatter` ：格式化器 轉換後格式轉換

    ```java
    @Bean
    @ConditionalOnProperty(prefix = "spring.mvc", name = "date-format")//在文件配置入職格式化的規則
    public Formatter<Date> dateFormatter() {
       return new DateFormatter(this.mvcProperties.getDateFormat());//日期格式化組件
    }
    ```

    自己添加的格式化轉換器，只需要放在容器中即可

- Support for `HttpMessageConverters` (see below).

  - `HttpMessageConverters` ：轉換HTTP轉換和響應：User - json

  - `HttpMessageConverters` ：是從容器中確定；獲取所有的`HttpMessageConverters`  ，將自己的組件註冊在容器中@Bean 

  - If you need to add or customize converters you can use Spring Boot’s `HttpMessageConverters` class:

    ```java
    import org.springframework.boot.autoconfigure.web.HttpMessageConverters;
    import org.springframework.context.annotation.*;
    import org.springframework.http.converter.*;
    
    @Configuration
    public class MyConfiguration {
    
        @Bean
        public HttpMessageConverters customConverters() {
            HttpMessageConverter<?> additional = ...
            HttpMessageConverter<?> another = ...
            return new HttpMessageConverters(additional, another);
        }
    
    }
    ```

- Automatic registration of `MessageCodesResolver` (see below).

  - 定義錯誤代碼生成規則

- Automatic use of a `ConfigurableWebBindingInitializer` bean (see below).

  - ```java
    @Override
    protected ConfigurableWebBindingInitializer getConfigurableWebBindingInitializer() {
       try {
          return this.beanFactory.getBean(ConfigurableWebBindingInitializer.class);
       }
       catch (NoSuchBeanDefinitionException ex) {
          return super.getConfigurableWebBindingInitializer();
       }
    }
    ```

    在beanFactory：中可以自己創建一個，初始化webDataBinder

    請求數據 ==》javaBean

If you want to keep Spring Boot MVC features, and you just want to add additional [MVC configuration](https://docs.spring.io/spring/docs/4.3.16.RELEASE/spring-framework-reference/htmlsingle#mvc) (interceptors, formatters, view controllers etc.) you can add your own `@Configuration` class of type `WebMvcConfigurerAdapter`, but **without** `@EnableWebMvc`. If you wish to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter` or `ExceptionHandlerExceptionResolver` you can declare a `WebMvcRegistrationsAdapter` instance providing such components.

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`.

思想：修改默認配置

### 2、擴展SpringMVC

編寫一個配置類，類型是WebMvcConfigurerAdapter(繼承)，使用WebMvcConfigurerAdapter可以擴展，不能標註@EnableWebMvc;既保留了配置，也能拓展我們自己的應用

```java
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
//        super.addViewControllers(registry);
        //瀏覽器發送wdjr請求，也來到success頁面
        registry.addViewController("/wdjr").setViewName("success");
    }
}
```

原理：

1）、WebMvcAutoConfiguration是SpringMVC的自動配置

2）、在做其他自動配置時會導入；@Import(EnableWebMvcConfiguration.class)

```java
@Configuration
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
    private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

	//從容器中獲取所有webMVCconfigurer
	@Autowired(required = false)
	public void setConfigurers(List<WebMvcConfigurer> configurers) {
		if (!CollectionUtils.isEmpty(configurers)) {
			this.configurers.addWebMvcConfigurers(configurers);
            
            	@Override
                protected void addViewControllers(ViewControllerRegistry registry) {
                    this.configurers.addViewControllers(registry);
                }
            //一個參考實現,將所有的webMVCconfigurer相關配置一起調用（包括自己的配置類）
            	@Override
               // public void addViewControllers(ViewControllerRegistry registry) {
                   // for (WebMvcConfigurer delegate : this.delegates) {
				 //delegate.addViewControllers(registry);
                    //}
                }
		}
	}
    
```



3）、自己的配置被調用

效果：SpringMVC的自動配置和我們的擴展配置都會起作用

### 3、全面接管mvc

不需要SpringBoot對SpringMVC的自動配置。

```java
@EnableWebMvc
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

@Override
public void addViewControllers(ViewControllerRegistry registry) {


//        super.addViewControllers(registry);
        //瀏覽器發送wdjr請求，也來到success頁面
        registry.addViewController("/wdjr").setViewName("success");
    }
}
```

例如靜態資源訪問，不推薦全面接管

原理：

為什麽@EnableWebMvc註解，SpringBoot對SpringMVC的控制就失效了

1）、核心配置

```java
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

2）、DelegatingWebMvcConfiguration

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
```

3）、WebMvcAutoConfiguration

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
      WebMvcConfigurerAdapter.class })
//容器沒有這個組件的時候，這個自動配置類才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
      ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```

4）、@EnableWebMvc將WebMvcConfigurationSupport導入進來了；

5）、導入的WebMvcConfigurationSupport只是SpringMVC最基本的功能



## 5、修改SpringMVC默認配置

模式:

```
1）、SpringBoot在自動配置很多組件的時候，先看容器中有沒有用戶自己配置的（@Bean、@Component）如果有就用用戶配置的，如果沒有，才自動配置；如果有些組件可以有多個（ViewResolver）將用戶配置的和自己默認的組合起來；

2）、在SpringBoot中會有 xxxConfigurer幫助我們擴展配置。
```

## 6、RestfulCRUD

### 1、默認訪問首頁

在config/MyConfig.java中編寫配置類

```java
//所有的webMvcConfigurerAdapter組件會一起起作用
@Bean //註冊到容器去
public WebMvcConfigurerAdapter webMvcConfigurerAdapter(){
    WebMvcConfigurerAdapter adapter = new WebMvcConfigurerAdapter() {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/").setViewName("login");
            registry.addViewController("/login.html").setViewName("login");
        }
    };
    return adapter;
}
```

靜態資源引用

```html
<link href="#" th:href="@{/css/signin.css}" rel="stylesheet" />
```

### 2、國際化

1、編寫國際化配置文件

2、使用ResourceBundleMessageSource管理國際化資源文件

3、在頁面中使用fmt:message，取出國際化內容

#### 1、瀏覽器切換國際化

步驟

1、編寫國際化配置文件，抽取頁面需要的顯示的國際化消息

![post-16](../images/blog/20190119/20190119-post-16.jpg)

2、SpringBoot自動配置好了國際化配置的資源文件

```java
@ConfigurationProperties(prefix = "spring.messages")
public class MessageSourceAutoConfiguration {
    //我們的配置文件可以直接放在類路徑下叫messages.properties
    private String basename = "messages";
    @Bean
	public MessageSource messageSource() {
		ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
		if (StringUtils.hasText(this.basename)) {
            //設置國際化文件的基礎名，去掉語言國家代碼
			messageSource.setBasenames(StringUtils.commaDelimitedListToStringArray(
					StringUtils.trimAllWhitespace(this.basename)));
		}
		if (this.encoding != null) {
			messageSource.setDefaultEncoding(this.encoding.name());
		}
		messageSource.setFallbackToSystemLocale(this.fallbackToSystemLocale);
		messageSource.setCacheSeconds(this.cacheSeconds);
		messageSource.setAlwaysUseMessageFormat(this.alwaysUseMessageFormat);
		return messageSource;
	}
```

3、對IDEA的編碼進行設置

![post-17](../images/blog/20190119/20190119-post-17.jpg)

4、login進行標簽插入

```html
<!DOCTYPE html>
<!-- saved from url=(0051)https://getbootstrap.com/docs/4.1/examples/sign-in/ -->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
    <meta name="description" content="" />
    <meta name="author" content="" />
    <link rel="icon" href="https://getbootstrap.com/favicon.ico" />

    <title>登錄頁面</title>

    <!-- Bootstrap core CSS -->
    <link href="#" th:href="@{/css/bootstrap.min.css}" rel="stylesheet" />

    <!-- Custom styles for this template -->
    <link href="./login_files/signin.css" th:href="@{/css/signin.css}" rel="stylesheet" />
  </head>

  <body class="text-center">
    <form class="form-signin">
      <img class="mb-4" src="./login_files/bootstrap-solid.svg" th:src="@{/img/bootstrap-solid.svg}" alt="" width="72" height="72" />
      <h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
      <label  class="sr-only" th:text="#{login.username}">Username</label>
      <input type="text"  name="username" class="form-control" placeholder="Username" th:placeholder="#{login.username}" required="" autofocus=""/>
      <label for="inputPassword" class="sr-only" th:text="#{login.password}">Password</label>
      <input type="password" name="password" id="inputPassword" class="form-control" placeholder="Password" th:placeholder="#{login.password}" required="" />
      <div class="checkbox mb-3">
        <label>
          <input type="checkbox" value="remember-me" /> [[#{login.remember}]]
        </label>
      </div>
      <button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
      <p class="mt-5 mb-3 text-muted">© 2017-2018</p>
    </form>
  

</body></html>
```

效果根據瀏覽器語言的信息切換國際化

原理：

國際化locale（區域信息對象）；LocaleResolver(獲取區域對象)；

```java
@Bean
@ConditionalOnMissingBean
@ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
public LocaleResolver localeResolver() {
    if (this.mvcProperties
        .getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
        return new FixedLocaleResolver(this.mvcProperties.getLocale());
    }
    AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
    localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
    return localeResolver;
}            

```

默認的就是根據請求頭帶來的區域信息獲取local國際化信息（截圖就是這麽犀利）

![post-18](../images/blog/20190119/20190119-post-18.jpg)

#### 2、點擊鏈接切換國際化

自己編寫localResolver，加到容器中

1、更改HTML代碼

```html
<p class="mt-5 mb-3 text-muted">© 2017-2018</p>
  <a href="#" class="btn btn-sm" th:href="@{/index.html?lg=zh_CN}">中文</a>
  <a href="#" class="btn btn-sm" th:href="@{/index.html?lg=en_US}">English</a>
```

2、新建一個MyLocaleResolver.class

```java
public class MyLocaleResolver implements LocaleResolver {

    //解析區域信息
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String l = request.getParameter("lg");
        Locale locale = Locale.getDefault();
        if(!StringUtils.isEmpty(l)){
            String[] split = l.split("_");
            locale = new Locale(split[0], split[1]);
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

    }
}
```

3、將MyLocaleResolver加入到容器中

```java
@Bean
public LocaleResolver localeResolver(){
    return new MyLocalResolver();
}
```

4、啟動演示

### 3、登錄攔截器

#### 1、登錄

開發技巧

```
1、清除模板緩存

2、Ctrl+F9刷新
```

1、新建一個LoginController

```java
@Controller
public class LoginController {

    @PostMapping(value ="/user/login")
    public String login(@RequestParam("username")String username,
                        @RequestParam("password")String password,
                        Map<String,Object> map){
        if(!StringUtils.isEmpty(username) && "123456".equals(password)){
            //登錄成功
            return "list";
        }else{
            map.put("msg", "用戶名密碼錯誤");
            return "login";
        }

    }
}
```

2、登錄錯誤消息顯示

```html
<!--判斷-->
<p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```

3、表單重複提交

表單重複提交事件 --》重定向來到成功頁面--》模板引擎解析

```java
if(!StringUtils.isEmpty(username) && "123456".equals(password)){
    //登錄成功,防止重複提交
    return "redirect:/main.html";
}else{
    map.put("msg", "用戶名密碼錯誤");
    return "login";
}
```

模板引擎解析

```java
@Override
public void addViewControllers(ViewControllerRegistry registry) {
    registry.addViewController("/").setViewName("login");
    registry.addViewController("/index.html").setViewName("login");
    registry.addViewController("/main.html").setViewName("Dashboard");
}
```

### 4、攔截器

作用：實現權限控制，每個頁面請求前中後，都會進入到攔截器進行處理（登錄權限）

1、在component下新建一個LoginHandlerInterceptor攔截器

```java
public class LoginHandlerInterceptor implements HandlerInterceptor {

    //目標方法執行之前
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object user = request.getSession().getAttribute("loginUser");
        if(user!=null){
            //已經登錄
            return true;
        }
        //未經過驗證
        request.setAttribute("msg", "沒權限請先登錄");
        request.getRequestDispatcher("/index.html").forward(request, response);

        return false;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

2、在MyMvcConfig配置中重寫攔截器方法，加入到容器中

```java
//所有的webMvcConfigurerAdapter組件會一起起作用
@Bean //註冊到容器去
public WebMvcConfigurerAdapter webMvcConfigurerAdapter(){
    WebMvcConfigurerAdapter adapter = new WebMvcConfigurerAdapter() {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/").setViewName("login");
            registry.addViewController("/index.html").setViewName("login");
            registry.addViewController("/main.html").setViewName("Dashboard");
        }
        //註冊攔截器
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            //靜態資源 css js img 已經做好了靜態資源映射
            registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**").
                    excludePathPatterns("/index.html","/","/user/login");
        }
    };
    return adapter;
}
```

3、在LoginHandler中添加登錄成功寫入session

```java
@Controller
public class LoginController {

    @PostMapping(value ="/user/login")
    public String login(@RequestParam("username")String username,
                        @RequestParam("password")String password,
                        Map<String,Object> map,
                        HttpSession session){
        if(!StringUtils.isEmpty(username) && "123456".equals(password)){
            //登錄成功,防止重複提交
            session.setAttribute("loginUser", username);
            return "redirect:/main.html";
        }else{
            map.put("msg", "用戶名密碼錯誤");
            return "login";
        }

    }
}
```

### 5、CRUD-員工列表

實驗要求：

1）、RestfulCRUD：CRUD滿足Rest風格

URI:/資源名稱/資源標識+HTTP操作

|      | 普通CRUD                | RestfulCRUD       |
| ---- | ----------------------- | ----------------- |
| 查詢 | getEmp                  | emp -- GET        |
| 添加 | addEmp?xxx              | emp --POST        |
| 修改 | updateEmp?id=xxx&xxx=xx | emp/{id} -- PUT   |
| 刪除 | deleteEmp?id=1          | emp/{id} --DELETE |

2、實驗的請求架構

|                | 請求URI  | 請求方式 |
| -------------- | -------- | -------- |
| 查詢所有員工   | emps     | GET      |
| 查詢某個員工   | emp/{id} | GET      |
| 添加頁面       | emp      | GET      |
| 添加員工       | emp      | POST     |
| 修改頁面(回顯) | emp/{id} | GET      |
| 修改員工       | emp/{id} | PUT      |
| 刪除員工       | emp/{id} | DELETE   |

3、員工列表

#### 1、公共頁面抽取

使用方法

```html
1、抽取公共片段
<!--footer.html-->
<div id="footid" th:fragment="copy">xxx</div>
2、引入公共片段
<!--test.html-->
<div th:insert=~{footer::copy}></div>
~{templatename::selector} 模板名::選擇器  footer::#footid
~{templatename::fragmentname} 模板名::片段名稱 footer::copy
行內寫法可以加~{xx::xx} 標簽體可以 xx::xx
```



**三種引用方式**

**th:insert** :加個外層標簽 +1

**th:replace** :完全替換 1

**th:include**：就替換里面的內容 -1

公共頁面

```html
<body>
	...
    <div th:insert="footer :: copy"></div>
    <div th:replace="footer :: copy"></div>
    <div th:include="footer :: copy"></div>
</body>
```

結果

```html
<body>
...
    <!-- th:insert -->
    <div>
        <footer>
            &copy; 2011 The Good Thymes Virtual Grocery
        </footer>
    </div>
    <!--th:replace-->
    <footer>
   		&copy; 2011 The Good Thymes Virtual Grocery
    </footer>
    <!--th:include-->
    <div>
        &copy; 2011 The Good Thymes Virtual Grocery
    </div>
</body>
```

用此種方法將公共頁面引入

#### 2、列表高亮

引入片段的時候傳入參數，新建一個commons文件夾存儲公共頁面bar.html

模板引入變量名

dashboard

```html
<a class="nav-link active"
   th:class="${activeUri}=='main.html'?'nav-link active':'nav-link'"
   href="https://getbootstrap.com/docs/4.1/examples/dashboard/#" th:href="@{/main.html}">
    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-home"><path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path><polyline points="9 22 9 12 15 12 15 22"></polyline></svg>
    Dashboard <span class="sr-only">(current)</span>
</a>
```

員工管理

```html
<li class="nav-item">
    <a class="nav-link"
       th:class="${activeUri}=='emps'?'nav-link active':'nav-link'"
       href="https://getbootstrap.com/docs/4.1/examples/dashboard/#" th:href="@{/emps}">
        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-users"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="9" cy="7" r="4"></circle><path d="M23 21v-2a4 4 0 0 0-3-3.87"></path><path d="M16 3.13a4 4 0 0 1 0 7.75"></path></svg>
        員工管理
    </a>
```

引入模板的時候傳入參數

dashboard.html引入

```html
<!--引入側邊欄-->
   <div th:replace="commons/bar :: sidebar(activeUri='main.html')"></div>
```

list.html引入

```html
<!--引入側邊欄-->
<div th:replace="commons/bar::sidebar(activeUri='emps')"></div>
```

### 6、列表數據顯示（查）

#### 1、傳入員工對象

EmployeeController類,傳入員工對象

```java
@Controller
public class EmployeeController {

    @Autowired
    EmployeeDao employeeDao;
    /**
     * 查詢所有員工返回列表頁面
     */
    @GetMapping(value = "/emps")
    public String list(Model model){

        Collection<Employee> employees = employeeDao.getAll();
        model.addAttribute("emps",employees);
        return "emp/list";
    }
}
```

#### 2、 遍歷對象

list.html中 使用模板的 `th:each`方法

```html
table class="table table-striped table-sm">
    <thead>
    <tr>
        <th>#</th>
        <th>lastName</th>
        <th>email</th>
        <th>gender</th>
        <th>department</th>
        <th>birth</th>
        <th>操作</th>
    </tr>
    </thead>
    <tbody>
        <tr th:each="emp:${emps}">
            <td th:text="${emp.id}">1</td>
            <td th:text="${emp.lastName}">1</td>
            <td th:text="${emp.email}">1</td>
            <td th:text="${emp.gender}">1</td>
            <td th:text="${emp.department.departmentName}">1</td>
            <td th:text="${#dates.format(emp.birth,'yyyy-MM-dd HH:mm:ss')}">1</td>
            <td>
                <button class="btn btn-sm btn-primary">編輯</button>
                <button class="btn btn-sm btn-danger">刪除</button>
            </td>
        </tr>
    </tbody>
</table>
```

#### 3、效果顯示

![post-19](../images/blog/20190119/20190119-post-19.jpg)



### 7、員工添加（增）

功能：點擊添加按鈕，出現新增頁面

#### 1、新增頁面

```html
<form>
    <!-- LastName -->
    <div class="form-group">
        <label for="LastName">LastName</label>
        <input type="text" class="form-control" id="LastName"  placeholder="LastName">
    </div>
    <!-- Email -->
    <div class="form-group">
        <label for="Email">Email</label>
        <input type="email" class="form-control" id="Email"  placeholder="zhangsan@163.com">
    </div>
    <!--gender-->
    <div class="form-group">
        <label >Gender</label><br/>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="gender" value="1">
            <label class="form-check-label" >男</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="gender" value="0">
            <label class="form-check-label" >女</label>
        </div>
    </div>
    <!-- department -->
    <div class="form-group">
        <label for="exampleFormControlSelect1">department</label>
        <select class="form-control" id="exampleFormControlSelect1">
            <option th:each="dept:${depts}" th:value="${dept.id}" th:text="${dept.departmentName}"></option>
        </select>
    </div>
    <!--Birth-->
    <div class="form-group">
        <label for="birthDate">Birth</label>
        <input type="text" class="form-control" id="birthDate" placeholder="2012-12-12">
    </div>
    <button type="submit" class="btn btn-primary">添 加</button>
</form>
```

#### 2、頁面跳轉

在EmployeeController中添加addEmpPage方法

```java
/**
 * 添加員工
 */
@GetMapping(value = "/emp")
public String toAddPage(Model model){
    //來到添加頁面,查出所有部門顯示
    Collection<Department> depts = departmentDao.getDepartments();
    model.addAttribute("depts",depts);
    return "emp/add";
}
```

關鍵點：在添加部門頁面要遍歷部門信息，所以在方法中出入部門信息

#### 3、添加功能完成

新建一個PostMapping

> ThymeleafViewResolver 查看redirect和forward,原生的sendredirect方法；

1、新建一個postMapping的方法用來接受頁面的添加POST請求

```java
/**
 * 員工添加
 */
@PostMapping(value = "/emp")
public String addEmp(Employee employee){

    employeeDao.save(employee);
    //來到員工列表頁面、redirect:重定向到一個地址，forward轉發到一個地址
    return "redirect:/emps";
}
```

2、修改添加頁面，添加name屬性

```html
<form th:action="@{/emp}" method="post">
    <!-- LastName -->
    <div class="form-group">
        <label for="LastName">LastName</label>
        <input type="text" class="form-control" id="LastName" name="lastName" placeholder="LastName">
    </div>
    <!-- Email -->
    <div class="form-group">
        <label for="Email">Email</label>
        <input type="email" class="form-control" id="Email"  name="email" placeholder="zhangsan@163.com">
    </div>
    <!--gender-->
    <div class="form-group">
        <label >Gender</label><br/>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="gender" value="1">
            <label class="form-check-label" >男</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="gender" value="0">
            <label class="form-check-label" >女</label>
        </div>
    </div>
    <!-- department -->
    <div class="form-group">
        <label >department</label>
        <select class="form-control"  name="department.id">
            <option th:each="dept:${depts}" th:value="${dept.id}" th:text="${dept.departmentName}"></option>
        </select>
    </div>
    <div class="form-group">
        <label for="birthDate">Birth</label>
        <input type="text" class="form-control" id="birthDate" placeholder="2012-12-12" name="birth">
    </div>
    <button type="submit" class="btn btn-primary">添 加</button>
</form>
```

1、部門對象問題？

```html
<select class="form-control"  name="department.id">
```

2、日期格式化？

屬性中添加 date-formate 默認是 / 

```java
@Bean
@ConditionalOnProperty(prefix = "spring.mvc", name = "date-format")
public Formatter<Date> dateFormatter() {
   return new DateFormatter(this.mvcProperties.getDateFormat());
}

@Override
public MessageCodesResolver getMessageCodesResolver() {
   if (this.mvcProperties.getMessageCodesResolverFormat() != null) {
      DefaultMessageCodesResolver resolver = new DefaultMessageCodesResolver();
      resolver.setMessageCodeFormatter(
            this.mvcProperties.getMessageCodesResolverFormat());
      return resolver;
   }
   return null;
}
```

```properties
spring.mvc.date-format=yyyy-MM-dd
```

### 8、員工編輯（改）

思路使用add頁面，並且數據回顯，然後區分添加，PUT請求

#### 1、修改按鈕

在list.html的`編輯`按鈕加上鏈接

```html
<td>
    <a  href="#" th:href="@{/emp/}+${emp.id}" class="btn btn-sm btn-primary">編輯</a>
    <button class="btn btn-sm btn-danger">刪除</button>
</td>
```

#### 2、編寫跳轉頁面

跳轉到員工編輯頁面的Controller

```java
/**
 * 員工編輯頁面
 */
@GetMapping(value = "/emp/{id}")
public String toEditPage(@PathVariable("id") Integer id ,Model model){
    Employee emp = employeeDao.getEmpById(id);
    Collection<Department> departments = departmentDao.getDepartments();
    model.addAttribute("emp",emp);
    model.addAttribute("depts",departments);
    return "emp/add";
}
   
```

#### 3、對頁面修改

對add頁面進行修改

1）、添加回顯

2）、添加判斷是否emp!=null（區分add or edit）

3）、添加put請求 --兩個input的hidden標簽

```html
 <form th:action="@{/emp}" method="post">
        <!--發送put請求-->
        <!--1.SpringMVC配置HiddenHttpMethodFilter
            2.頁面創建一個post表單
            3.創建一個 input name_method 值就是我們請求的方式-->
        <input type="hidden" name="_method" value="put" th:if="${emp!=null}">

        <input type="hidden" name="id" th:value="${emp.id}" th:if="${emp!=null}">
        <!-- LastName -->
        <div class="form-group">
            <label for="LastName">LastName</label>
            <input type="text" class="form-control" id="LastName" name="lastName" placeholder="LastName" th:value="${emp!=null}?${emp.lastName}">
        </div>
        <!-- Email -->
        <div class="form-group">
            <label for="Email">Email</label>
            <input type="email" class="form-control" id="Email"  name="email" placeholder="zhangsan@163.com" th:value="${emp!=null}?${emp.email}">
        </div>
        <!--gender-->
        <div class="form-group">
            <label >Gender</label><br/>
            <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" name="gender" value="1" th:checked="${emp!=null}?${emp.gender}==1">
                <label class="form-check-label" >男</label>
            </div>
            <div class="form-check form-check-inline">
                <input class="form-check-input" type="radio" name="gender" value="0" th:checked="${emp!=null}?${emp.gender}==0">
                <label class="form-check-label" >女</label>
            </div>
        </div>
        <!-- department -->
        <div class="form-group">
            <label >department</label>
            <select class="form-control"  name="department.id" >
                <option th:selected="${emp!=null}?${dept.id == emp.department.id}" th:each="dept:${depts}" th:value="${dept.id}" th:text="${dept.departmentName}"></option>
            </select>
        </div>
        <div class="form-group">
            <label for="birthDate">Birth</label>
            <input type="text" class="form-control" id="birthDate" placeholder="2012-12-12" name="birth" th:value="${emp!=null}?${#dates.format(emp.birth,'yyyy-MM-dd HH:mm:ss')}">
        </div>
        <button type="submit" class="btn btn-primary" th:text="${emp!=null}?'修改':'添加'">添 加</button>
    </form>
</main>
```

### 9、員工刪除（刪）

#### 1、新建Contoller

```java
/**
 * 員工刪除
 */
@DeleteMapping(value = "/emp/{id}")
public String deleteEmp(@PathVariable("id") Integer id){
    employeeDao.deleteEmpById(id);
    return "redirect:/emps";
}
```

#### 2、修改刪除標簽

```html
<button th:attr="del_uri=@{/emp/}+${emp.id}"  class="btn btn-sm btn-danger deleteBtn">
    刪除
</button>
```

#### 3、寫Form表單

form表單卸載外面，input 中 name="_method" value="delete" 模擬delete請求

```html
                </tbody>
            </table>
        </div>
    </main>
    <form id="deleteEmpForm" method="post">
        <input type="hidden" name="_method" value="delete">
    </form>
</div>
```

#### 4、寫JS提交

```javascript
<script>
    $(".deleteBtn").click(function () {
        $("#deleteEmpForm").attr("action",$(this).attr("del_uri")).submit();
        return false;
    })
</script>
```

> return false;禁用btn提交效果

## 7、錯誤機制的處理

### 1、默認的錯誤處理機制

默認錯誤頁面

![post-20](../images/blog/20190119/20190119-post-20.jpg)

原理參照

ErrorMvcAutoConfiguration:錯誤處理的自動配置

```
org\springframework\boot\spring-boot-autoconfigure\1.5.12.RELEASE\spring-boot-autoconfigure-1.5.12.RELEASE.jar!\org\springframework\boot\autoconfigure\web\ErrorMvcAutoConfiguration.class

```

- DefaultErrorAttributes

  幫我們在頁面共享信息

  ```java
  @Override
  public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes,
        boolean includeStackTrace) {
     Map<String, Object> errorAttributes = new LinkedHashMap<String, Object>();
     errorAttributes.put("timestamp", new Date());
     addStatus(errorAttributes, requestAttributes);
     addErrorDetails(errorAttributes, requestAttributes, includeStackTrace);
     addPath(errorAttributes, requestAttributes);
     return errorAttributes;
  }
  ```

- BasicErrorController

  ```java
  @Controller
  @RequestMapping("${server.error.path:${error.path:/error}}")
  public class BasicErrorController extends AbstractErrorController {
      //產生HTML數據
      @RequestMapping(produces = "text/html")
  	public ModelAndView errorHtml(HttpServletRequest request,
  			HttpServletResponse response) {
  		HttpStatus status = getStatus(request);
  		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
  				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
  		response.setStatus(status.value());
  		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
  		return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
  	}
  	//產生Json數據
  	@RequestMapping
  	@ResponseBody
  	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
  		Map<String, Object> body = getErrorAttributes(request,
  				isIncludeStackTrace(request, MediaType.ALL));
  		HttpStatus status = getStatus(request);
  		return new ResponseEntity<Map<String, Object>>(body, status);
  	}
  ```

- ErrorPageCustomizer

  ```java
  @Value("${error.path:/error}")
  private String path = "/error";//系統出現錯誤以後來到error請求進行處理，(web.xml)
  ```

- DefaultErrorViewResolver

  ```java
  @Override
  public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status,
        Map<String, Object> model) {
     ModelAndView modelAndView = resolve(String.valueOf(status), model);
     if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
        modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
     }
     return modelAndView;
  }
  
  private ModelAndView resolve(String viewName, Map<String, Object> model) {
      //默認SpringBoot可以找到一個頁面？error/狀態碼
     String errorViewName = "error/" + viewName;
      //如果模板引擎可以解析地址，就返回模板引擎解析
     TemplateAvailabilityProvider provider = this.templateAvailabilityProviders
           .getProvider(errorViewName, this.applicationContext);
     if (provider != null) {
         //有模板引擎就返回到errorViewName指定的視圖地址
        return new ModelAndView(errorViewName, model);
     }
      //自己的文件 就在靜態文件夾下找靜態文件 /靜態資源文件夾/404.html
     return resolveResource(errorViewName, model);
  }
  ```

一旦系統出現4xx或者5xx錯誤 ErrorPageCustomizer就回來定制錯誤的響應規則,就會來到 /error請求,BasicErrorController處理，就是一個Controller

1.響應頁面,去哪個頁面是由 DefaultErrorViewResolver 拿到所有的錯誤視圖

```java
protected ModelAndView resolveErrorView(HttpServletRequest request,
      HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
   for (ErrorViewResolver resolver : this.errorViewResolvers) {
      ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
      if (modelAndView != null) {
         return modelAndView;
      }
   }
   return null;
}
```

l瀏覽器發送請求 accpt:text/html

客戶端請求：accept:/*

### 2、如何定制錯誤響應

```
1）、如何定制錯誤的頁面

	1.有模板引擎：靜態資源/404.html,什麽錯誤什麽頁面；所有以4開頭的 4xx.html 5開頭的5xx.html

	有精確的404和4xx優先選擇404

	頁面獲得的數據

		timestamp：時間戳

		status：狀態碼

		error：錯誤提示

		exception：異常對象

		message：異常信息

		errors:JSR303有關

	2.沒有放在模板引擎，放在靜態文件夾，也可以顯示，就是沒法使用模板取值

	3.沒有放模板引擎，沒放靜態，會顯示默認的錯誤

2）、如何定義錯誤的數據
```



舉例子：新建4xx和5xx文件

![post-21](../images/blog/20190119/20190119-post-21.jpg)



```html
<body >
    <p>status: [[${status}]]</p>
    <p>timestamp: [[${timestamp}]]</p>
    <p>error: [[${error}]]</p>
    <p>message: [[${message}]]</p>
    <p>exception: [[${exception}]]</p>
</body>
```

![post-22](../images/blog/20190119/20190119-post-22.jpg)

### 3、如何定制Json數據

#### 1、僅發送json數據

```java
public class UserNotExitsException extends  RuntimeException {
    public UserNotExitsException(){
        super("用戶不存在");
    }
}
```

```java
/**
 * 異常處理器
 */
@ControllerAdvice
public class MyExceptionHandler {

    @ResponseBody
    @ExceptionHandler(UserNotExitsException.class)
    public Map<String ,Object> handlerException(Exception e){
        Map<String ,Object> map =new HashMap<>();
        map.put("code", "user not exist");
        map.put("message", e.getMessage());
        return map;
    }
}
```

無法自適應 都是返回的json數據

#### 2、轉發到error自適應處理

```java
@ExceptionHandler(UserNotExitsException.class)
public String handlerException(Exception e, HttpServletRequest request){
    Map<String ,Object> map =new HashMap<>();
    //傳入自己的狀態碼
    request.setAttribute("javax.servlet.error.status_code", 432);
    map.put("code", "user not exist");
    map.put("message", e.getMessage());
    //轉發到error
    return "forward:/error";
}
```

程序默認獲取狀態碼

```java
protected HttpStatus getStatus(HttpServletRequest request) {
   Integer statusCode = (Integer) request
         .getAttribute("javax.servlet.error.status_code");
   if (statusCode == null) {
      return HttpStatus.INTERNAL_SERVER_ERROR;
   }
   try {
      return HttpStatus.valueOf(statusCode);
   }
   catch (Exception ex) {
      return HttpStatus.INTERNAL_SERVER_ERROR;
   }
```

沒有自己寫的自定義異常數據

#### 3、自適應和定制數據傳入

Spring 默認的原理，出現錯誤後回來到error請求，會被BasicErrorController處理,響應出去的數據是由BasicErrorController的父類AbstractErrorController(ErrorController)規定的方法getAttributes得到的；

1、編寫一個ErrorController的實現類【或者AbstractErrorController的子類】，放在容器中；

2、頁面上能用的數據，或者是json數據返回能用的數據都是通過errorAttributes.getErrorAttributes得到；

容器中的DefaultErrorAtrributes.getErrorAtrributees();默認進行數據處理

```java
public class MyErrorAttributes extends DefaultErrorAttributes {
    @Override
    public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
        Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
        map.put("company", "wdjr");
        return map;
    }
}
```

異常處理：把map方法請求域中

```java
    @ExceptionHandler(UserNotExitsException.class)
    public String handlerException(Exception e, HttpServletRequest request){
        Map<String ,Object> map =new HashMap<>();
        //傳入自己的狀態碼
        request.setAttribute("javax.servlet.error.status_code", 432);
        map.put("code", "user not exist");
        map.put("message", e.getMessage());
        request.setAttribute("ext", map);
        //轉發到error
        return "forward:/error";
    }
}
```

在上面的MyErrorAttributes類中加上

```java
//我們的異常處理器
Map<String,Object> ext = (Map<String, Object>) requestAttributes.getAttribute("ext", 0);
map.put("ext", ext);
```

## 8、配置嵌入式servlet容器

### 1、定制和修改Servlet容器

SpringBoot默認使用Tomcat作為嵌入式的Servlet容器；

![post-23-01](../images/blog/20190119/20190119-post-23-01.jpg)

問題？

1）、如何定制和修改Servlet容器；

1、 修改Server相關的配置文件 application.properties

```properties
#通用的servlet容器配置
server.xxx
#tomcat的配置
server.tomcat.xxxx
```

2、編寫一個EmbeddedServletContainerCustomizer;嵌入式的Servlet容器的定制器；來修改Servlet的容器配置

```java
@Bean
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return new EmbeddedServletContainerCustomizer() {
        //定制嵌入式Servlet的容器相關規則
        @Override
        public void customize(ConfigurableEmbeddedServletContainer container) {
            container.setPort(8999);
        }
    };
}
```

其實同理，都是實現EmbeddedServletContainerCustomizer

### 2、註冊Servlet三大組件

三大組件 Servlet Filter Listener

由於SprringBoot默認是以jar包啟動嵌入式的Servlet容器來啟動SpringBoot的web應用，沒有web.xml

註冊三大組件

#### ServletRegistrationBean

```java
@Bean
public ServletRegistrationBean myServlet(){
    ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new MyServlet(),"/servlet");
    return servletRegistrationBean;
}
```

MyServlet

```java
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("Hello Servlet");
    }
}
```

#### FilterRegistrationBean

```java
@Bean
public FilterRegistrationBean myFilter(){
    FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
    filterRegistrationBean.setFilter(new MyFilter());
    filterRegistrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
    return filterRegistrationBean;
}
```

MyFilter

```java
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("MyFilter process");
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {

    }
}
```

#### ServletListenerRegistrationBean

```java
@Bean
public ServletListenerRegistrationBean myListener(){
    ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(new MyListener());
    return registrationBean;
}
```

MyListener

```java
public class MyListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        System.out.println(".........web應用啟動..........");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        System.out.println(".........web應用銷毀..........");
    }
}
```



SpringBoot幫助我們自動配置SpringMVC的時候，自動註冊SpringMVC的前端控制器；DispatcherServlet;

```java
@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
   public ServletRegistrationBean dispatcherServletRegistration(
         DispatcherServlet dispatcherServlet) {
      ServletRegistrationBean registration = new ServletRegistrationBean(
            dispatcherServlet, this.serverProperties.getServletMapping());
       //默認攔截 /所有請求 包括靜態資源 不包括jsp
       //可以通過server.servletPath來修改SpringMVC前端控制器默認攔截的請求路徑
      registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
      registration.setLoadOnStartup(
            this.webMvcProperties.getServlet().getLoadOnStartup());
      if (this.multipartConfig != null) {
         registration.setMultipartConfig(this.multipartConfig);
      }
      return registration;
   }

}
```

### 3、切換其他的Servlet容器

在ServerProperties中

```java
private final Tomcat tomcat = new Tomcat();

private final Jetty jetty = new Jetty();

private final Undertow undertow = new Undertow();

```

tomcat(默認支持)

jetty（長連接）

undertow（多並發）

切換容器 僅僅需要修改pom文件的依賴就可以

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
<!--        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
        </dependency>-->

```

### 4、嵌入式Servlet容器自動配置原理

```java
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration
@ConditionalOnWebApplication
@Import(BeanPostProcessorsRegistrar.class)
//給容器導入組件 後置處理器 在Bean初始化前後執行前置後置的邏輯 創建完對象還沒屬性賦值進行初始化工作
public class EmbeddedServletContainerAutoConfiguration {
    @Configuration
	@ConditionalOnClass({ Servlet.class, Tomcat.class })//當前是否引入tomcat依賴
    //判斷當前容器沒有用戶自定義EmbeddedServletContainerFactory，就會創建默認的嵌入式容器
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedTomcat {

		@Bean
		public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
			return new TomcatEmbeddedServletContainerFactory();
		}

```

1）、EmbeddedServletContainerFactory（嵌入式Servlet容器工廠）

```java
public interface EmbeddedServletContainerFactory {
	//獲取嵌入式的Servlet容器
   EmbeddedServletContainer getEmbeddedServletContainer(
         ServletContextInitializer... initializers);

}

```

繼承關系

![post-24](../images/blog/20190119/20190119-post-24.jpg)

2）、EmbeddedServletContainer:(嵌入式的Servlet容器)

![post-25](../images/blog/20190119/20190119-post-25.jpg)

3）、TomcatEmbeddedServletContainerFactory為例 

```java
@Override
public EmbeddedServletContainer getEmbeddedServletContainer(
      ServletContextInitializer... initializers) {
   Tomcat tomcat = new Tomcat();
    //配置tomcat的基本環節
   File baseDir = (this.baseDirectory != null ? this.baseDirectory
         : createTempDir("tomcat"));
   tomcat.setBaseDir(baseDir.getAbsolutePath());
   Connector connector = new Connector(this.protocol);
   tomcat.getService().addConnector(connector);
   customizeConnector(connector);
   tomcat.setConnector(connector);
   tomcat.getHost().setAutoDeploy(false);
   configureEngine(tomcat.getEngine());
   for (Connector additionalConnector : this.additionalTomcatConnectors) {
      tomcat.getService().addConnector(additionalConnector);
   }
   prepareContext(tomcat.getHost(), initializers);
    //將配置好的tomcat傳入進去；並且啟動tomcat容器
   return getTomcatEmbeddedServletContainer(tomcat);
}

```

4）、嵌入式配置修改

```
ServerProperties、EmbeddedServletContainerCustomizer

```

EmbeddedServletContainerCustomizer:定制器幫我們修改了Servlet容器配置？

怎麽修改？



5）、容器中導入了**EmbeddedServletContainerCustomizerBeanPostProcessor**

```java
@Override
public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata,
      BeanDefinitionRegistry registry) {
   if (this.beanFactory == null) {
      return;
   }
   registerSyntheticBeanIfMissing(registry,
         "embeddedServletContainerCustomizerBeanPostProcessor",
         EmbeddedServletContainerCustomizerBeanPostProcessor.class);
   registerSyntheticBeanIfMissing(registry,
         "errorPageRegistrarBeanPostProcessor",
         ErrorPageRegistrarBeanPostProcessor.class);
}

```

```java
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName)
      throws BeansException {
    //如果當前初始化的是一個ConfigurableEmbeddedServletContainer
   if (bean instanceof ConfigurableEmbeddedServletContainer) {
      postProcessBeforeInitialization((ConfigurableEmbeddedServletContainer) bean);
   }
   return bean;
}

private void postProcessBeforeInitialization(
    ConfigurableEmbeddedServletContainer bean) {
    //獲取所有的定制器，調用每個定制器的customer方法給Servlet容器進行賦值
    for (EmbeddedServletContainerCustomizer customizer : getCustomizers()) {
        customizer.customize(bean);
    }
}

private Collection<EmbeddedServletContainerCustomizer> getCustomizers() {
    if (this.customizers == null) {
        // Look up does not include the parent context
        this.customizers = new ArrayList<EmbeddedServletContainerCustomizer>(
            this.beanFactory
            //從容器中獲取所有的這個類型的組件：EmbeddedServletContainerCustomizer
            //定制Servlet,給容器中可以添加一個EmbeddedServletContainerCustomizer類型的組件
            .getBeansOfType(EmbeddedServletContainerCustomizer.class,
                            false, false)
            .values());
        Collections.sort(this.customizers, AnnotationAwareOrderComparator.INSTANCE);
        this.customizers = Collections.unmodifiableList(this.customizers);
    }
    return this.customizers;
}

```

ServerProperties也是EmbeddedServletContainerCustomizer定制器

步驟：

1）、SpringBoot根據導入的依賴情況，給容器中添加響應的容器工廠 例：tomcat

EmbeddedServletContainerFactory【TomcatEmbeddedServletContainerFactory】

2）、容器中某個組件要創建對象就要通過後置處理器；

```java
EmbeddedServletContainerCustomizerBeanPostProcessor

```

只要是嵌入式的Servlet容器工廠，後置處理器就工作；

3）、後置處理器，從容器中獲取的所有的EmbeddedServletContainerCustomizer，調用定制器的定制方法

### 5、嵌入式Servlet容器啟動原理

什麽時候創建嵌入式的Servlet的容器工廠？什麽時候獲取嵌入式的Servlet容器並啟動Tomcat;

獲取嵌入式的容器工廠

1）、SpringBoot應用啟動Run方法

2）、刷新IOC容器對象【創建IOC容器對象，並初始化容器，創建容器的每一個組件】；如果是web環境AnnotationConfigEmbeddedWebApplicationContext,如果不是AnnotationConfigApplicationContext

```JAVA
if (contextClass == null) {
   try {
      contextClass = Class.forName(this.webEnvironment
            ? DEFAULT_WEB_CONTEXT_CLASS : DEFAULT_CONTEXT_CLASS);
   }

```

3）、refresh(context);刷新創建好的IOC容器

```java
try {
   // Allows post-processing of the bean factory in context subclasses.
   postProcessBeanFactory(beanFactory);

   // Invoke factory processors registered as beans in the context.
   invokeBeanFactoryPostProcessors(beanFactory);

   // Register bean processors that intercept bean creation.
   registerBeanPostProcessors(beanFactory);

   // Initialize message source for this context.
   initMessageSource();

   // Initialize event multicaster for this context.
   initApplicationEventMulticaster();

   // Initialize other special beans in specific context subclasses.
   onRefresh();

   // Check for listener beans and register them.
   registerListeners();

   // Instantiate all remaining (non-lazy-init) singletons.
   finishBeanFactoryInitialization(beanFactory);

   // Last step: publish corresponding event.
   finishRefresh();
}

```

4）、 onRefresh();web的ioc容器重寫了onRefresh方法

5）、webioc會創建嵌入式的Servlet容器；createEmbeddedServletContainer

6）、獲取嵌入式的Servlet容器工廠；

```java
EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();

```

從ioc容器中獲取EmbeddedServletContainerFactory組件；

```java
@Bean
public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
return new TomcatEmbeddedServletContainerFactory();
}

```

TomcatEmbeddedServletContainerFactory創建對象，後置處理器看這個對象，就來獲取所有的定制器來定制Servlet容器的相關配置；

7）、使用容器工廠獲取嵌入式的Servlet容器

8）、嵌入式的Servlet容器創建對象並啟動Servlet容器；

先啟動嵌入式的Servlet容器，在將ioc容器中剩下的沒有創建出的對象獲取出來

ioc啟動創建Servlet容器

## 9、使用外置的Servlet容器

嵌入式的Servlet容器：應用達成jar包

```
優點：簡單、便攜

缺點：默認不支持JSP、優化定制比較複雜（使用定制器【ServerProperties、自定義定制器】，自己來編寫嵌入式的容器工廠）

```

外置的Servlet容器：外面安裝Tomcat是以war包的方式打包。

### 1、IDEA操作外部Servlet

1、創建程序為war程序

![post-26](../images/blog/20190119/20190119-post-26.jpg)

2、選擇版本

![post-27](../images/blog/20190119/20190119-post-27.jpg)

3、添加tomcat

![post-28](../images/blog/20190119/20190119-post-28.jpg)

4、選擇tomcat

![post-30](../images/blog/20190119/20190119-post-30.jpg)

5、選擇本地的Tomcat

![post-31](../images/blog/20190119/20190119-post-31.jpg)

6、配置tomcat路徑

![post-32](../images/blog/20190119/20190119-post-32.jpg)

7、添加服務器

![post-33](../images/blog/20190119/20190119-post-33.jpg)

8、添加exploded的war配置，應用OK tomcat配置完成

![post-34](../images/blog/20190119/20190119-post-34.jpg)

二、配置webapp文件夾

1、點擊配置

![post-35](../images/blog/20190119/20190119-post-35.jpg)

2、添加webapp目錄

![post-36](../images/blog/20190119/20190119-post-36.jpg)

3、默認配置就可以

![post-37](../images/blog/20190119/20190119-post-37.jpg)

4、配置web.xml文件

![post-38](../images/blog/20190119/20190119-post-38.jpg)

5、文檔目錄結構

![post-39](../images/blog/20190119/20190119-post-39.jpg)

### 2、運行一個示例

1、項目目錄

![post-40](../images/blog/20190119/20190119-post-40.jpg)

2、配置文件寫視圖解析前後綴

```properties
spring.mvc.view.prefix=/WEB-INF/jsp/

spring.mvc.view.suffix=.jsp

```

3、HelloController

```java
@Controller
public class HelloController {
    @GetMapping("/hello")
    public String hello(Model model){
        model.addAttribute("message","這是Controller傳過來的message");
        return "success";
    }
}

```

4、success.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Success</title>
</head>
<body>
<h1>Success</h1>
message:${message}
</body>
</html>

```

5、運行結果

![post-41](../images/blog/20190119/20190119-post-41.jpg)

步驟

1、必須創建一個war項目；

2、將嵌入式的Tomcat指定為provided

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>

```

3、必須編寫一個SpringBootServletInitializer的子類，並調用configure方法里面的固定寫法

```java
public class ServletInitializer extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        //傳入SpringBoot的主程序，
        return application.sources(SpringBoot04WebJspApplication.class);
    }

}

```

4、啟動服務器就可以；

### 3、原理

jar包：執行SpringBoot主類的main方法，啟動ioc容器，創建嵌入式的Servlet的容器；

war包：啟動服務器，服務器啟動SpringBoot應用，【SpringBootServletInitializer】啟動ioc容器

servlet3.0規範

 8.2.4 共享庫和運行時插件

規則：

1、服務器啟動（web應用啟動），會創建當前的web應用里面每一個jar包里面ServletContrainerInitializer的實現類的實例

2、SpringBootServletInitializer這個類的實現需要放在jar包下的META-INF/services文件夾下，有一個命名為javax.servlet.ServletContainerInitalizer的文件，內容就是ServletContainerInitializer的實現類全類名

3、還可以使用@HandlerTypes註解，在應用啟動的時候可以啟動我們感興趣的類



流程：

1、啟動Tomcat服務器

2、spring web模塊里有這個文件

![post-42](../images/blog/20190119/20190119-post-42.jpg)

```java
org.springframework.web.SpringServletContainerInitializer

```

3、SpringServletContainerInitializer將handlerTypes標註的所有類型的類傳入到onStartip方法的Set<Class<?>>;為這些感興趣類創建實例

4、每個創建好的WebApplicationInitializer調用自己的onStratup

5、相當於我們的SpringBootServletInitializer的類會被創建對象，並執行onStartup方法

6、SpringBootServletInitializer執行onStartup方法會創建createRootApplicationContext

```java
protected WebApplicationContext createRootApplicationContext(ServletContext servletContext) {
    SpringApplicationBuilder builder = this.createSpringApplicationBuilder();
    //環境構建器
    StandardServletEnvironment environment = new StandardServletEnvironment();
    environment.initPropertySources(servletContext, (ServletConfig)null);
    builder.environment(environment);
    builder.main(this.getClass());
    ApplicationContext parent = this.getExistingRootWebApplicationContext(servletContext);
    if (parent != null) {
        this.logger.info("Root context already created (using as parent).");
        servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, (Object)null);
        builder.initializers(new ApplicationContextInitializer[]{new ParentContextApplicationContextInitializer(parent)});
    }
	
    builder.initializers(new ApplicationContextInitializer[]{new ServletContextApplicationContextInitializer(servletContext)});
    builder.contextClass(AnnotationConfigEmbeddedWebApplicationContext.class);
    //調用Configure,子類重寫了這個方法，將SpringBoot的主程序類傳入進來
    builder = this.configure(builder);
    //創建一個spring應用
    SpringApplication application = builder.build();
    if (application.getSources().isEmpty() && AnnotationUtils.findAnnotation(this.getClass(), Configuration.class) != null) {
        application.getSources().add(this.getClass());
    }

    Assert.state(!application.getSources().isEmpty(), "No SpringApplication sources have been defined. Either override the configure method or add an @Configuration annotation");
    if (this.registerErrorPageFilter) {
        application.getSources().add(ErrorPageFilterConfiguration.class);
    }
	//最後啟動Spring容器
    return this.run(application);
}

```

7、Spring的應用就啟動完了並且創建IOC容器；

```java
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch();
   stopWatch.start();
   ConfigurableApplicationContext context = null;
   FailureAnalyzers analyzers = null;
   configureHeadlessProperty();
   SpringApplicationRunListeners listeners = getRunListeners(args);
   listeners.starting();
   try {
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);
      Banner printedBanner = printBanner(environment);
      context = createApplicationContext();
      analyzers = new FailureAnalyzers(context);
      prepareContext(context, environment, listeners, applicationArguments,
            printedBanner);
      refreshContext(context);
      afterRefresh(context, applicationArguments);
      listeners.finished(context, null);
      stopWatch.stop();
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass)
               .logStarted(getApplicationLog(), stopWatch);
      }
      return context;
   }
   catch (Throwable ex) {
      handleRunFailure(context, listeners, analyzers, ex);
      throw new IllegalStateException(ex);
   }
}

```
