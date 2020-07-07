---
title: "Web Internationalization"
date: 2018-12-02T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

Web application implement i18n
<!--more-->
## 網站實現多語系提供兩種做法
1:單獨將選擇的語言放入WebApplication的Session（本偏說明此方法）  
2:判斷瀏覽器目前使用的Language,也是Location的方式

## 範例使用SSH架構
分成前端和後端兩種方式,實現多語系功能  
後端：後台讀取多語系的property檔

```java
public String login() {
  String language = this.request.getParameter("language");
  sessionMap.put("language", language);
  request.setAttribute("language", language);
  ReadPropertiesFile file=new ReadPropertiesFile();
    Map<String, String> fileMap = null;
  try {
  //讀取對應的property語言檔
    fileMap = file.select(language);
  } catch (IOException e) {
    e.printStackTrace();
  }
  //將讀取的語言欓放入Session中
  sessionMap.put("alertNote", fileMap);
  Map<String, String> alertNote=(Map<String, String>) sessionMap.get("alertNote");
  return SUCCESS;
}
```

>之後就能在SessionMap中利用key對應要輸出的value
```java
public String execute() {
  Map<String, String> alertNote=(Map<String, String>) sessionMap.get("alertNote");
  String sessionAuthcode = (String) this.request.getSession().getAttribute("authCode");
  if (!sessionAuthcode.equals(authcode)) {
    //對應的key
    this.addActionError(alertNote.get("errphrase28"));
    return INPUT;
  }
  return SUCCESS
```

>前端：讀取json語系檔
```html
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
  <script language="JavaScript" src="${pageContext.request.contextPath}/js/jquery-1.4.4.min.js"></script>
 //在DOM元件加上tkey屬性,並對應到語系檔的key
<title tkey="title">Demo</title>
  <script type="text/javascript">
  //利用jQuery改寫網頁達成多語系效果
   function translate(lang){
	   var translate = function(jsdata){
       //一般欄位
	       jQuery("[tkey]").each(function(index) {
	   	      var strTr = jsdata [jQuery(this).attr('tkey')];
	   	      jQuery(this).html(strTr);
	   	   });
	   	 //按鈕
	       jQuery("[btkey]").each(function(index) {
		   	   var strTr = jsdata [jQuery(this).attr('btkey')];
		   	   jQuery(this).val(strTr);
		   });	    

       //可以針對option進行轉換
       jQuery("option").each(function(index) {
         if (jQuery(this).html() == '-請選擇-') {
           var strTr = jsdata ['select'];
           jQuery(this).html(strTr);   
       }
         if (jQuery(this).html() == '男') {
           var strTr = jsdata ['male'];
           jQuery(this).html(strTr);   
       }
         if (jQuery(this).html() == '女') {
           var strTr = jsdata ['female'];
           jQuery(this).html(strTr);   
       }
      });	    
	   }
	   //此處注意lang source對應路徑
	   if (lang=='tw') {
	       jQuery.getJSON('lang/tw.json',translate);
	   }

	   if (lang=='jp') {
	       jQuery.getJSON('lang/jp.json',translate);
	   }

	   if(lang=='en') {
		   jQuery.getJSON('lang/en.json',translate);
	   }
   }
   $(function() {
		   translate('${language}')
   });
  </script>
</head>
<body leftmargin="0" topmargin="0" marginwidth="0" marginheight="0">
</body>
</html>
```

## 總結
利用jQuery達到多語系效果,就是用selector選擇該物件並進行改寫其值，
可以直接將轉換的JavaScript寫在util的jsp，
再將其include,但要注意判斷source路徑。
