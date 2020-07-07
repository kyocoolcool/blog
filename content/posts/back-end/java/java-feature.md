---
title: "Java 8 Feature"
date: 2019-04-23T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

## 1、 About Java 8

Java 8 是JDK很重大的版本，Oracle 公司於 2014 年 3 月 18 日發佈 Java 8 ，它支持函數式編程，新的 JavaScript 引擎，新的日期 API，新的Stream API 等。

## 2、Features

- Lambda expressions

- Method references

- Functional interfaces

- Stream API

- Default methods

- Base64 Encode Decode

- Static methods in interface

- Optional class

- Collectors class

- ForEach() method

- Nashorn JavaScript Engine

- Parallel Array Sorting

- Type and Repating Annotations

- IO Enhancements

- Concurrency Enhancements

- JDBC Enhancements etc

- Date API

  

## 3、Lambda

Lambda表達式需要函數式接口的支持:函數式接口:Interface中只有一個抽象方法，稱為函數式接口，可以使用@FunctionInterface修飾，JDK提供許多實用的Functional Interface，主要是能將方法作為參數傳入。

- void Consumer<T>
- <T> Supplier<T>
- <R> Fuction<T,R>
- boolean Predicate<T>
- <R> BiFunction<T,U,R>
- <T> UnaryOperator<T>
- <T> BinaryOperator<T,T>
- void BiConsumer<T,U>
- int ToIntFunction<T>
- long ToLongFunction<T>
- double ToDoubleFunction<T>
- IntFunction<R>
- LongFunction<R>
- DoubleFunction<R>

## 4、Method references

搭配Lambda 使用可以很簡單的操作物件的方法。

## 5、Stream

像是在SQL中操作集合中的數據，可以任意的條件化對集合做過濾、篩選、排序等操作。

## 6、Optional

容器類，避免NullPoinException的相對操作。

## 7、Date API

- LocalDate
- LocalTime
- LocalDateTIme
- Instant
- Duration
- Period
- TemporalAdjuster
- DateTimeFormatter
- ZoneDate
- ZoneTime
- ZoneDateTime
- ZoneId

## 8、總結

在研究Lambda expressions，因有別於以往Java風格，所以開始的學習曲線較緩慢，但了解設計由來理念之後，感覺到一片海闊天空，在coding部分真的超方便、編碼品質也大大提升，且readable，可見JDK設計者在此方面下了很大的用心。

實作功能代碼：[java8](<https://github.com/kyocoolcool/java-tutorial/tree/master/java8-feature>)
