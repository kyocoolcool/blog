---
title: "Spring Data Fundamentals"
date: 2019-04-21T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

## 1、 About Spring Data

Spring Data 提供利用簡單的方式存取資料庫，減少使用JDBC對資料庫的開銷，透過更高級的封裝來讓開發者有更靈活的應用，不管是關係型資料庫或是非關係形資料庫皆可使用。

Spring Data’s mission is to provide a familiar and consistent, Spring-based programming model for data access while still retaining the special traits of the underlying data store.

It makes it easy to use data access technologies, relational and non-relational databases, map-reduce frameworks, and cloud-based data services. This is an umbrella project which contains many subprojects that are specific to a given database. The projects are developed by working together with many of the companies and developers that are behind these exciting technologies.

## 2、Features

- Powerful repository and custom object-mapping abstractions
- Dynamic query derivation from repository method names
- Implementation domain base classes providing basic properties
- Support for transparent auditing (created, last changed)
- Possibility to integrate custom repository code
- Easy Spring integration via JavaConfig and custom XML namespaces
- Advanced integration with Spring MVC controllers
- Experimental support for cross-store persistence

## 3、Spring Data JPA

Spring Data JPA是基於關係型資料庫的ORM框架，JPA是應用規範，底層實現使用Hibernate，簡化開發者對資料庫存取的操作，除了基本CRUD，還有排序、分頁操作。

## 4、Spring Data 操作概念

因Spring Data JPA內建提供多種Repository API，所以只要在Interface層 繼承JPA Repository，定義方法時，只要符合Spring Data規定，底層就會自動實現ORM的操作。還有更進階的操作可以透過JpaSpecificationExecutor，帶入查詢條件並進行排序分頁操作。

## 5、方法關鍵字

| Logical keyword     | Keyword expressions                      |
| ------------------- | ---------------------------------------- |
| AND                 | And                                      |
| OR                  | Or                                       |
| AFTER               | After, IsAfter                           |
| BEFORE              | Before, IsBefore                         |
| CONTAINING          | Containing, IsContaining, Contains       |
| BETWEEN             | Between, IsBetween                       |
| ENDING_WITH         | EndingWith, IsEndingWith, EndsWith       |
| EXISTS              | Exists                                   |
| FALSE               | False, IsFalse                           |
| GREATER_THAN        | GreaterThan, IsGreaterThan               |
| GREATER_THAN_EQUALS | GreaterThanEqual, IsGreaterThanEqual     |
| IN                  | In, IsIn                                 |
| IS                  | Is, Equals, (or no keyword)              |
| IS_EMPTY            | IsEmpty, Empty                           |
| IS_NOT_EMPTY        | IsNotEmpty, NotEmpty                     |
| IS_NOT_NULL         | NotNull, IsNotNull                       |
| IS_NULL             | Null, IsNull                             |
| LESS_THAN           | LessThan, IsLessThan                     |
| LESS_THAN_EQUAL     | LessThanEqual, IsLessThanEqual           |
| LIKE                | Like, IsLike                             |
| NEAR                | Near, IsNear                             |
| NOT                 | Not, IsNot                               |
| NOT_IN              | NotIn, IsNotIn                           |
| NOT_LIKE            | NotLike, IsNotLike                       |
| REGEX               | Regex, MatchesRegex, Matches             |
| STARTING_WITH       | StartingWith, IsStartingWith, StartsWith |
| TRUE                | True, IsTrue                             |
| WITHIN              | Within, IsWithin                         |

## 6、總結

Spring Data JPA，在配置上與Spring+Hibernate差不多，一樣是透過DataSource取得連線，Spring Data JPA交給entityManagerFactory管理，單純Hibernate則交由sessionFactory管理。在學習起來建議先從Hibernate了解，再研究Spring Data JPA，就可以體會Spring Data JPA帶來的魔力。

實作功能代碼：[spring-data](<https://github.com/kyocoolcool/spring-integration/tree/master/spring-data-fundamentals>)
