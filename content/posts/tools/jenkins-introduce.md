---
title: "Jenkins Introduce"
date: 2019-04-16T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- tools
---

## 1、Jenkins介紹

The leading open source automation server, Jenkins provides hundreds of plugins to support building, deploying and automating any project.

Jenkins是由[Java](https://zh.wikipedia.org/wiki/Java)編寫的[開源](https://zh.wikipedia.org/wiki/%E5%BC%80%E6%BA%90)的[持續整合及部署](https://zh.wikipedia.org/wiki/%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90)工具。

Jenkins提供了軟體開發的持續整合服務。它執行在Servlet容器中（例如Apache Tomcat）。它支援[軟體組態管理](https://zh.wikipedia.org/wiki/%E8%BD%AF%E4%BB%B6%E9%85%8D%E7%BD%AE%E7%AE%A1%E7%90%86)（SCM）工具（包括[AccuRev SCM](https://zh.wikipedia.org/w/index.php?title=AccuRev_SCM&action=edit&redlink=1)、[CVS](https://zh.wikipedia.org/wiki/CVS)、[Subversion](https://zh.wikipedia.org/wiki/Subversion)、[Git](https://zh.wikipedia.org/wiki/Git)、[Perforce](https://zh.wikipedia.org/w/index.php?title=Perforce&action=edit&redlink=1)、[Clearcase](https://zh.wikipedia.org/w/index.php?title=Clearcase&action=edit&redlink=1)和[RTC](https://zh.wikipedia.org/w/index.php?title=Rational_Team_Concert&action=edit&redlink=1)），可以執行基於[Apache Ant](https://zh.wikipedia.org/wiki/Apache_Ant)和[Apache Maven](https://zh.wikipedia.org/wiki/Apache_Maven)的專案，以及任意的[Shell指令碼](https://zh.wikipedia.org/wiki/Shell%E8%84%9A%E6%9C%AC)和Windows[批次處理命令](https://zh.wikipedia.org/wiki/%E6%89%B9%E6%AC%A1%E6%AA%94)。

可以通過各種手段觸發構建。例如提交給版本控制系統時被觸發，也可以通過類似[Cron](https://zh.wikipedia.org/wiki/Cron)的機制排程，也可以在其他的構建已經完成時，還可以通過一個特定的[URL](https://zh.wikipedia.org/wiki/URL)進行請求。

## 2、CI/CD

主要是程式開發完成後自動完成繁複的連串動作。

- CI(Continuous Integration):持續整合

  版本控制、自動構建、自動化測試、程式碼分析

- CD(Continuous Delivery):持續部署

  發佈、手動測試

- CD(Continuous Deployment):持續交付

  將最新版本交付

  {{< img src="/images/blog/20190416/20190416-post-1.png">}}


## 3、WebHook

- 搭配IDE VCS 在SVN或git commit 之後，觸發WebHook去執行CI/CD等連串活動


## 4、總結

CI/CD 概念，可以加速程式開發的速度、程式碼的品質(避免衝突或Bug)、簡化繁雜的Flow，在軟體工程中可以縮短開發時程，並確保最新的程式碼是沒問題的。

目前實作:git->Jenkins->Github(最新版本)->Maven->Deploy Tomcat

後續會再紀錄自動測試部分。:smile:

實作功能代碼：[jenkins](<https://github.com/kyocoolcool/jenkins-cicd>)
