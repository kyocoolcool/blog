---
title: "Java Concurrent"
date: 2019-08-12T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

## 1、 About Concurrent

在軟體效能提高四種方式:分佈、異步、緩存、併發，此篇主要來探討併發編程，來大幅度提高代碼效能，利用多線程處理業務邏輯或運算，但必須確保線程安全。

## 2、何謂線程安全

要避免資源內容經修改後，其他線程還讀到舊的資源內容(Dirty read)或者多線程同時對資源進行修改產生不是業務邏輯要的結果，所以必須資源是可見的及資源是同步的。

## 3、知識點

> 利用Mind map整理Concurrent用到的知識點

{{< img src="/images/blog/20190812/20190812-post-1.png">}}

## 4、總結

經過實際敲代碼編寫多線程併發時，需注意資源的同步及可見，理解公平鎖及互斥鎖擁有資源的機制，在高併發的容器的運作，各種連線池的應運場景，在進行耗時複雜計算時，將任務分成多個子任務再合併結果，實際掌握後確實大幅度提升系統效能。

實作功能代碼：[java-concurrent](https://github.com/kyocoolcool/java-tutorial/tree/master/java-concurrent)
Note:程式碼已註釋較完整，直接看代碼應用會比較容易理解 😄😄😄
