---
title: "Java Annotation"
date: 2019-05-28T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

## 1、 About Java Annotation

Java Annotation 是JDK5.0引入的一種注釋機制，可以當成是標籤，用以解釋、規範、附帶訊息。

## 2、原生Annotation

是可以註解到註解上的註解，或者說原生Annotation是一種基本註解，但是它能夠應用到其它的註解上面。

- @Target
- @Retention
- @Document
- @Inhrited
- @Repeatable

## 3、預置註解

- @Deprecated
- @Override
- @SuppressWarnings
- @SafeVarargs
- @FunctionalInterface

## 4、映射提取

在適當的時候獲取註解內的數值。

## 5、映射反射

在Hibernate與Spring框架中皆利用註解反射來獲取資訊，並達到對應的操作。

## 6、總結

註解用在聲明優於配置，並指定應用時機、應用位置、攜帶的範圍值，了解Annotation的運作原理更能深入各框架源碼設計理念與思想。

實作功能代碼：[java-annotation](<https://github.com/kyocoolcool/java-tutorial/tree/master/java-annotation>)
