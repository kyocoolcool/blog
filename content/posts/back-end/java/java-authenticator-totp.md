---
title: "Java Authenticator TOTP"
date: 2019-08-13T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- back-end
- java
---

## 1、 About TOTP

TOTP(基於時間的一次性密碼演算法)，是一種根據預共用的金鑰與目前時間計算一次性密碼的演算法。它已被接納為RFC 6238標準，成為oauth的基石，並被用於眾多驗證系統當中。

TOTP是HMAC(金鑰雜湊訊息鑑別碼)當中的一個例子。它結合一個私鑰與目前時間，使用一個密碼雜湊涵式來生成一次性密碼。由於網路延遲與時鐘不同步可能導緻密碼接收者不得不嘗試多次遇到正確的時間來進行身分驗證，時間戳通常以30秒為間隔，從而避免反覆嘗試。

## 2、Java實現

透過TOTP演算法產生一組key是Client與Server共同約定的，再利用google.zxing API產生QRCode圖片，並將訊息放入，因此訊息內容必須包含金鑰及使用者名稱產出的標準須以otpauth://totp為開頭，才能利用Authenticator辨識，演算法會依金鑰隨著每30秒，產出6位密碼，在用此密碼核對儲存在server的金鑰所產出的密碼是否相同，因考量計算時間的時間差，30秒可能會不足，所以通常預設都是會依當下時間產出前3組密碼去進行比對，此部分能在系統內調整數量。

```
/*user:自定義user
	host:自定義domain
	secret:約定金鑰
*/
otpauth://totp/user@host?secret=secret
```

## 3、產生QRCode在網頁

QRCode可以產生實體圖片檔在硬碟中，但大多數會放在HTML頁面上讓User掃描，這邊補充Struts2產生QRCode轉成OutputStream方法，若是Spring MVC則在範例代碼中實現。

```java
public void getQRCode() {
        List<ApsAccount> accoutntList = wapDAO.findAccountByLoginId(loginId);
        ApsAccount apsAccount = accoutntList.get(0);
        String securityKey=apsAccount.getSecuritykey();
        int width = 300;
        int height = 300;
        String content = "otpauth://totp/" + loginId + "@WebAPIS?secret=" + securityKey;
        ServletOutputStream outputStream = null;
        try {
            HttpServletResponse response = ServletActionContext.getResponse();
            response.setContentType("image/png");
            outputStream = response.getOutputStream();
            QRCodeUtil.writeToStream(content, outputStream, width, height);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (outputStream != null) {
                try {
                    outputStream.flush();
                    outputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```


## 4、總結

增加TOTP authenticator ，對於帳密保護以及系統安全多了一道防護，但仍必須考量登入失敗的次數鎖定，避免無上限try 出user 金鑰，以及忘記金鑰後或手機丟失等後續確認使用者真實身份流程作業。

實作功能代碼：[java-authenticator-totp](https://github.com/kyocoolcool/authenticator-totp-fundamentals)
Note:實際透過此程式產出金鑰及QRCode

> 實際產生一個專屬QRCode 🤗
{{< img src="/images/blog/20190813/20190813-post-1.png">}}

