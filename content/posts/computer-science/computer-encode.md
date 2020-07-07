---
title: "Computer Encode"
date: 2019-12-16T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- computer-science
---
Ever imagined how a computer is able to understand and display 
what you have written? Ever wondered what a UTF-8 or UTF-16 meant
when you were going through some configurations? Just think about
how “HeLLo WorlD” should be interpreted by a computer.

##為什麼需要編碼
We all know that a computer stores data in bits and bytes. So,
to display a character on screen or map the character as a byte in memory of the computer needs to have a standard. Read the following :

`\x48\x65\x4C\x4C\x6F\x20\x57\x6F\x72\x6C\x44` 這是ASCII 16進制表示法，
表示 HeLLo WorlD，所以上面這串我們並不容易閱讀。 編碼就像名字"小明”，
因電腦只懂得0與1的machine code，所以我們制定一個標準(e.g. Big5)以後小明代號就是9527，
然後告知電腦我是用Big5編碼，所以電腦就知道小明是9527並把9527變成0與1儲存起來。
這樣人類就能跟電腦溝通，但語言不僅僅是中文，也有其他語言，所以各國有不同的標準，這樣語言世界就很混亂了，
e.g. 9527這代號在中文國家編碼標準是"小明”，但在英文國家卻是"Adam”，
為了解決這問題就有一個全世界通用的編碼標準讓電腦識別。

##ASCII
ASCII(American Standard Code for Information Interchange，美國資訊交換標準代碼)
最早用來統一英文文字符號和數字間的對應關係，除了 52 個大小寫英文字母外，其中也包含了常用的符號（如 !, <, = ）
和特殊字元，總共定義了 127 個關係，使用到7個bit。
可以透過ASCII Table，Char的A對應到DEC(十進制)的65。

##EASCII
是將ASCII碼由7bit擴充為8bit(1 byte)而成。EASCII的內碼是由0到255共有256個字元組成。
EASCII碼比ASCII碼擴充出來的符號包括表格符號、計算符號、希臘字母和特殊的拉丁符號。

##Unicode
因 ASCII 中定義了 127 個文字符號和數字之間的關係，但是那是因為英文只需要有 26 個英文字母就可以組成各式單字，
但是中文或者說許多國家的語言並不是如此，所以ASCII並不適用所有國家。而Unicode就誕生了，又稱為萬國碼，
顧名思義就是定義多個國家語言的標準，最喜標準化，透過標準可以讓運用擴充發展，而至2019年5月止已經收藏13萬個字元，
仍在擴充中，當然也支援emoji e.g. 😄😆😎🥶🍌🍎🍒 。  
一般在 Unicode 中我們會用十六進制來表示某一個文字符號的編號，並且使用 U+<十六進制數值> 的方式來表示，
例如 小明的「小」用 Unicode 來表示是 U+5c0f，「明」則是用U+660e 表示。

##進位制是什麼
二進制（Binary；bin）：只用 0 和 1 這兩個數字表示一個數值的方式就稱為二進制，因此最後要給電腦看的數值最終都將轉為二進制（Binary）。
在 JavaScript 中，如果你要告訴程式這是一個二進制的數字，需在數字的最前面加上 0b 。 
八進位（Octal；oct）：只用 0, 1, … 7 這八個數字來表示一個數值的方式稱為八進制。在 JavaScript 中，
如果你要告訴程式這是一個八進制的數字，需在數字的最前面加上 0 或 0o 。 
十進位（Decimal；dec）：這是平常使用的數字表示法，只用 0, 1, … 9 這十個數字來表示一個數值的方式稱為十進制。
在 JavaScript 中，如果你要告訴程式這是一個十進制的數字，你什麼都不必做，直接輸入的數字就是表示十進制。
十六進位（Hexadecimal；hex）： 只用 0, 1, …, 9, A, B, C, D, E, F 這十六個是數字來表示一個數值的方式稱為十六進制，
其中 A 表示十進制中的 10， B 表示十進制中的 11，F 表示十進制中的 15，以此類推。在色碼的表示上，
也常使用十六進制的數字（如，#FF00AA ）來表示一個顏色。在 JavaScript 中，如果你要告訴程式這是一個十六進制的數字，
需在數字的最前面加上 0x。  
使用十六進制的好處是，可以使用較少的位數就能表示更多的數值，例如十六進制的兩位數的 FF 就表示在十進制的需要用三位數表示的 255。

##進位制轉換工具
計算機是好物，e.g. A對應到的進位制數值，如下:
>二進位制和八進位制

{{< img src="/images/blog/20191216/20191216-post-1.png">}}


>十進位制

{{< img src="/images/blog/20191216/20191216-post-2.png">}}

>十六進位制

{{< img src="/images/blog/20191216/20191216-post-3.png">}}

##碼點（Code Point）
概念上，我們會把這些文字符號所對應到的編號，稱作是碼點，又稱作「編碼位置」。
在Unicode 中指的是 U+ 後面的十六進制數值，每個碼點都是唯一的。

根據碼點的編號範圍，又可以分成「基本平面」和「輔助平面」。
Unicode 編碼中碼點的可能範圍從 U+0000 一直到 U+10FFFF 超過 110 萬個文字符號，
因此又可分為基本平面（BMP, Basic Multilingual Plane）和輔助平面（SMP, Supplementary planes or Astral planes）。
基本平面（BMP）：碼點位置範圍從 U+0000 到 U+FFFF，這個平面放了最常見的文字符號。  
輔助平面（SMP）：碼點位置範圍從 U+010000 一直到 U+10FFFF，又稱為補充平面。  
此外，原本 ASCII 中數字和文字符號的對應關係，可以直接沿用到 Unicode 中，也就是 ASCII 中 a 的編號會和 Unicode 中 a 的編號相同。

在JavaScript其中根據不同的適用時機或場合有幾種不同的表示方式，包括： 只使用到 Unicode 基本平面時（\u<碼點>） 有使用到 Unicode 輔助平面時（\u{ <碼點> }） 只使用到 ASCII 字符時（\x<碼點>）

##UTF-8, UTF-16, UTF-32 是什麼
透過 Unicode（萬國碼）讓所有國家的文字符號都有個唯一的碼點可以在電腦內被辨認，
但是這碼點（從 U+0000 到 U+10FFFF）要如何儲存在電腦中則不是 Unicode 要解決的，
也就是說要用什麼樣的方式才能有效的把所需的碼點存在電腦中，但又不佔據太多的容量則沒有在Unicode 中被規範。
UTF-8, UTF-16 或 UTF-32 則都可以視為是 Unicode 實做的一種方式，
它們用不同的方式來規範要如何將 Unicode 中的碼點儲存在電腦中以被使用，如果直接把整個 Unicode 代碼搬進電腦裡儲存，
可能會佔用太多不必要的空間（例如，UTF-32）。 在網際網路上最常被使用的則是 UTF-8 的編碼方式，
而在 JavaScript 引擎中主要支援的則是 UTF-16。

###UTF-16
代表一個字元是用2 bytes表示。

###UTF-8
並不是用1 byte表示一個字元，而是可變動長度的字元。 用UTF-8就有複雜點.因爲程序是把一個字節一個字節的來讀取,  
然後再根據字節中開頭的bit標誌來識別是該把1個還是兩個或三個字節做爲一個單元來處理。  
0xxxxxxx,如果是這樣的01串,也就是以0開頭後面就不用管了.就表示把一個字節做爲一個單元.就跟ASCII完全一樣。    
110xxxxx 10xxxxxx.如果是這樣的格式,則把兩個字節當一個單元。  
1110xxxx 10xxxxxx 10xxxxxx 如果是這種格式則是三個字節當一個單元。

{{< img src="/images/blog/20191216/20191216-post-4.png">}}

若在純英語系國家會推薦使用UTF-8，節省空間，而且編碼代號也與ASCII相同，若在中文語系則較適用UTF-16，可以減少轉換判斷的時間。

