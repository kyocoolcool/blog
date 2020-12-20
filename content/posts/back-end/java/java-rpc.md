---
title: "Java RPC應用"
date: 2020-12-20T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

## 1、 What is RPC
[Demo Code](https://github.com/kyocoolcool/rpc-fundamentals)  
RPC(Remote Procedure Call)，是一種概念，在分佈式計算系統，不同系統間可以透過約定好的協定(實作的方式)調用，就像是直接在本地調用方法。

## 2、Features

從單機到分佈式應用，跨系統之間的溝通得到解決。

  

## 3、RPC實作方式

![OCAJP](https://raw.githubusercontent.com/kyocoolcool/blog/master/static/images/blog/20201220/20201220-post.png)

## 4、RPC序列化框架
- java.io.serializable
- Hessian
- google protobuf
- facebook Thrift
- kyro
- fst
- json序列化(Jackson、google Gson、Ali FastJson)
- xml序列化(xStream)

## 5、RPC通訊協定
- http
- http2.0(gRPC)
- TCP
- WebService


