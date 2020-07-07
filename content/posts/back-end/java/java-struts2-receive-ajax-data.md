---
title: "Struts2 Receive AJAX Data"
date: 2019-11-01T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

Struts2解析請求的集合數據映射到物件的方式較SpringWeb繁瑣，本篇主要來說明如何傳送及接收數據並映射到物件集合。

### 案例1、Array數據傳送接收

> JavaScript (前端發送請求代碼)
```javascript
function issue(){
  let mainDocNoList=["123","456","789"];//陣列
  const params=$.params({"mainDocNoList":mainDocNoList},true);
  //將陣列序列化，可再增加數據e.g.
  //params=$.params({"mainDocNoList":mainDocNoList,"name":"Chris"},true)
  $.ajax({
    type:'POST',
    url:'issue.do',
    data:params,
    cache:false,
    async:false,
    dataType:'json',
    success:function(data){
      console.log(data);
    }
  });
}
```

> Java (後端接收請求代碼)
```java
public class IssueAction extends ActionSupport{
  List<String> mainDocNoList;//也可以改成String[] mainDocNoList;
  private String result;
//省略getter&setter 必須要加，是透過setter賦值
  public String issue(){
    System.out.pring(mainDocNoList);//已映射到Action中的屬性
    result="success";//回傳結果
    return SUCCESS;
  }
}
```

```xml
<struts> <!--struts.xml 需有json plugin才能接收回覆json request&response-->
  <package name="issue" namespace="/api" extends="struts-default,json-default">
    <action name="issueAction" method="issue" class="action.issueAction">
    <result type="json">
      <parm name="root">result</parm> <!--回傳結果-->
    </result>
  </package>
</struts>
```

### 案例2、json數據(非集合)傳送接收

> JavaScript (前端發送請求代碼)
```javascript
function issue(){
  var name="Chris";
  $.ajax({
    type:'POST',
    url:'issue.do',
    data:{"name":name},//封裝成json格式
    cache:false,
    async:false,
    dataType:'json',
    success:function(data){
      console.log(data);
    }
  });
}
```

> Java (後端接收請求代碼)
```java
public class IssueAction extends ActionSupport{
  private String name;
  private String result;
//省略getter&setter 必須要加，是透過setter賦值
  public String issue(){
    System.out.pring(name);//已映射到Action中的屬性
    result="success";//回傳結果
    return SUCCESS;
  }
}
```

### 案例3、json數據(集合)傳送接收

> JavaScript (前端發送請求代碼)
```javascript
function issue(){
  json={
    "animalVoList[0].name":"Dog",
    "animalVoList[0].age":"3",
    "animalVoList[1].name":"Cat",
    "animalVoList[1].age":"5",
  }
  $.ajax({
    type:'POST',
    url:'issue.do',
    data:json,//傳送json格式資料
    cache:false,
    async:false,
    dataType:'json',
    success:function(data){
      console.log(data);
    }
  });
}
```

> Java (後端接請求代碼)
```java
public class IssueAction extends ActionSupport{
  private List<Animal> animalVoList;
  private String result;
//Animal類有name及age fields
//省略getter&setter 必須要加，是透過setter賦值
  public String issue(){
    System.out.pring(animalVoList);//已映射到Action中的屬性
    result="success";//回傳結果
    return SUCCESS;
  }
}
```

### 總結

利用Ajax傳送json數據映射到Struts2的屬性，這種操作在實務上很實用，也與SpringWeb操作方式差異較大，所以整理記錄下來。
