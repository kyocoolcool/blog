---
title: "What Is SSL & TLS"
date: 2019-04-17T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- computer-science
---

# SSL&TLS

## 1、HTTP(HyperText Transfer Protocol)
在說到SSL與TLS之前，要先介紹http屬於TCP/IP的應用層，用來客戶端與伺服器端請求與應答的標準。但在傳輸過程的封包內容是以明碼的方式傳輸，所以會有資料外洩的風險，因此才需要再封包進行加密後傳輸。

## 2、HTTPS(HyperText Transfer Protocol Secure)
比HTTP更安全的傳輸協定，在傳輸的時候進行授權認證及加密。主要是透過SSL及TLS。

## 3、SSL(Secure Socket Layer)
主要用於保持網際網路連線安全，採用非同步加密方式(RSA)，當第一次Client與Server端建立連線前，會先詢問Server確認身份，Server會將授權憑證(CA 廠商、地址、)傳輸給Client端，並交給public key，並自己擁有private key，Client收到後會再透過約定的加密方式回覆Server已收到訊息，前面的流程就叫做hand shake，之後傳輸的內容就會用public key加密再傳至Server，Server再用private key解密，Server端回應Client端後在用public key解密。目前SSL大多採用2048bit加密。

## 4、TLS(Transport Layer Security)
是更新、更安全的 SSL 版本。
