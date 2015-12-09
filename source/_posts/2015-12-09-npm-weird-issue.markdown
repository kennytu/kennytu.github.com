---
layout: post
title: "NPM的怪問題"
date: 2015-12-09 20:13:55 +0800
comments: true
categories: [node.js, npm, sails]
---

好的, 這篇我要來好好的念一下NPM, 也就是 package manager for node.js

這真的太令我生氣啦!!!

如果你是用**WIN7**... 對! 就是**WIN7**, 你可能會遇到底下我遇到的問題

當你去了官網 <a href="https://nodejs.org/en/">nodejs.org</a> 下載Windows的安裝包, 安裝完成之後, 通常`npm`都會一起被安裝進去了

這個時候你開啟`cmd`, 輸入`node`或者`npm`, 應該都已經可以執行

<!--more-->

好的, 然後你就用了一段時間, 使用`npm install -g`巴拉巴拉 用的很高興

突然間, 你看到網路上, 欸! 有`node.js`以及`npm`的**更新**耶, 超爽Der

你就順手的打開`cmd`, 輸入`npm update -g`

然後`npm`跑了好一段時間來update....

##接著..噩夢從此開始...

你開心地覺得有更新版的可用真好

然後阿

你又在node.js找到一些想要用的module, OK

馬上來玩玩看! 使用 `npm install 巴拉巴拉 -g`

疑!? ERROR!?

蛤!!?!? 簡單的裝個npm install 套件, 居然會ERROR?!!?

大概類似出現底下這樣,

`npm err windows_nt 6.1.7601`

然後還有什麼...

`Error: EXDEV, cross-device link not permitted`

OK, 你開始找問題, 上網搜尋了一堆, 大家好像也都遇到奇怪的問題

然後還會亂給建議...

沒辦法! 真的沒頭緒, 只能一個個Try...

不管如何Try, 你會發現這個ERROR出現的太詭異了...

找啊找的.. 真的完全沒頭緒

你快瘋了...

##就這樣, 突然靈光一閃

好的, 遇到一些奇怪的問題時, 真的沒辦法解, 你會想要怎麼做?

對, 使用**Clean Room**的環境!!!

先解釋什麼叫做`Clean Room`, `Clean Room`就是在乾淨的地方, 重新做一次實驗

我想到, 假如我安裝個模組就出現一堆ERROR, 那node.js release manager不就是要去吃...嗎!!

當然他不會吃...! 他一定會做好完整的測試才會Release對吧!?

OK, 所以呢, 接下來, 我執行了

1. `npm cache clean -f` (npm還會跳出一個警告視窗說: 你確定你知道你要做的是什麼對吧?) 
2. `npm install -g` (重裝一次)

好的, 沒用!

**Fuxxxxk!**

##突然間, 又靈光一閃

`npm cache clean` 真的有完全清除嗎!?

我就到了`C:\Users\KennyTu\AppData\Roaming`目錄下的`npm-cache`去看, 執行完clean之後, 這個目錄的東西完全沒動到阿!

為什麼我會知道要去這個`C:\Users\KennyTu\AppData\Roaming`目錄呢, 你就執行

`npm config get prefix`, npm就會return它把安裝的東西放到哪邊去了

哈哈哈哈~~~

好的, 最後我怎麼做呢?

##最終解法

不囉嗦, 照著底下做

1. 從windows的新增移除程式, 移除node.js以及相關的套件(如果有的話)
2. 到`C:\Users\KennyTu\AppData\Roaming`把`npm-cache`以及`npm`這兩個目錄幹掉!
3. 重開機
3. 重新從<a href="https://nodejs.org/en/">nodejs.org</a> 下載Windows的安裝包, 安裝一次
4. 大功告成!


對, 就是這樣

為什麼會有這樣的問題呢!?

起因就是在於, 新版和舊版的npm對於路徑的設定不同, 然後又不向下相容, 所以就會導致新的npm要去找某個路徑, 發現他找不到!?

舉個例子, 新版npm在安裝沒有安裝過的模組的時候, 需要 `超機車的耶` 這個套件, 然後npm發現cache裡面已經有 `超機車的耶` 這個套件, 但是這個 `超機車的耶` 這個套件是舊的npm裝的, 有些路徑設定在新版的npm沒有, 然後呢, 新版的npm就很高興的去 `超機車的耶` 這個cache裡面找他要的一些資訊.... 欸!!? 沒有找到!? 怎麼可能!!? 幹! 報錯給使用者了啦! 老子不爽裝了啦!!! 

好的, 就是這樣, 這也是為什麼我們要把前朝的npm所遺留下來的cache清掉的原因....

##提醒Windows用戶的注意事項

所以呢, 總結一下經驗法則

1. **他X的千萬不要在console中, 使用npm update -g**, 你會死掉
2. **如果你不小心手癢了! 請照上述的最終解法重來一遍吧**

OK, 附帶一提

###新版sails的問題

當你把npm修好之後, 使用`npm install -g sails` 安裝成功

使用`sails new testProj`

then `cd testProj`

Run `sails lift`

你會發現幹~ 怎麼不能RUN, 報錯呢!!! 靠杯

好的, 自力自強, 看error log, 是說找不到什麼洨的套件

就安裝他吧! 例如他靠杯說少了`基八毛`套件

那就是`npm install -g 基八毛`

裝到Global安裝目錄去! 

接著看`sails lift` 報什麼洨 她找不到的, 就通通滿足她!

裝飽之後~ 

`sails lift` 就可以正常執行啦!



