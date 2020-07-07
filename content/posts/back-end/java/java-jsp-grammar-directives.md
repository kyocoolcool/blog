---
title: "JSP Grammar Directives"
date: 2018-10-03T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

JSP(Java Server Page)屬於模板引擎一種，是Servelet的特例，最後也會編譯成.class檔，
JSP將Java代碼和特定變動內容嵌入到靜態的頁面中，實現以靜態頁面為模板，動態生成其中的部分內容。
當客戶端傳送的請求，並根據請求內容動態地生成HTML、XML或其他格式文件的Web網頁，然後返回給請求者。
<!--more-->
## JSP四種語法
>JSP註釋
```jsp
<%-- 註釋內容 --%>
```

>JSP宣告
```jsp
<%! 宣告內容 --%>
```

>JSP輸出表達
```jsp
<%= 輸出表達式 --%>
```

>JSP寫程式
```jsp
<%= 程式內容 --%>
```

## JSP三種編譯指令
>page
```jsp
<%@page  屬性名="屬性值" --%>
```
###屬性名
 | `language` | `extends` | `import` | `session` |`buffer` |`autoFlush` |`info` |`errorPage` |`isErrorPage` |`contentType` |`pageEncoding` |
 
 >include
 ```jsp
 <%@include file="relativeURLSpec" --%>
 ```

>taglib
 ```jsp
 <%@taglib 屬性名="屬性值" --%>
 ```

## JSP七種動作指令
###指令
 | `jsp:forward` | `jsp:param` | `jsp:include` | `jsp:plugin` |`jsp:useBean` |`jsp:setProperty` |`jsp:getProperty` |

