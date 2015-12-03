---
layout: post
title: "Node.js更新問題"
date: 2015-12-03 19:31:51 +0800
comments: true
categories: [node.js, npm, sails]
---

在更新`node.js`以及`npm`之後 (npm現在被包在node.js裡面一起安裝)

我執行npm -g update, global有更新的node.js的module都update到最新的

接著我切換到我這一陣子再用的sails的專案, 執行sails lift的時候

發生了`Cannot find module 'lru-cache'` 這個事件

<!--more-->

解決的方法是, 執行以下的指令

	rm -rf node_modules
	npm cache clean
	npm install

先將sails專案裡面的node_modules刪除, 然後清空Cache, 接著重新安裝一次, 這樣Sails就可以正常運行了


另外, 在執行sails lift的時候, 若sails的資料庫有綁定mongo db, 但你還沒有啟動mondo db時, 會出現以下訊息


```batch

error: A hook (`orm`) failed to load!
E:\sails_app\activeityOverload\node_modules\mongodb\lib\server.js:235
        process.nextTick(function() { throw err; })
                                      ^

TypeError: Cannot read property 'close' of undefined
    at E:\sails_app\activeityOverload\node_modules\sails-mongo\lib\adapter.js:138:13
    at E:\sails_app\activeityOverload\node_modules\sails-mongo\node_modules\async\lib\async.js:187:20
    at E:\sails_app\activeityOverload\node_modules\sails-mongo\node_modules\async\lib\async.js:239:13
    at _arrayEach (E:\sails_app\activeityOverload\node_modules\sails-mongo\node_modules\async\lib\async.js:91:13)
    at _each (E:\sails_app\activeityOverload\node_modules\sails-mongo\node_modules\async\lib\async.js:82:13)

```

這就表示sails 掛不到 mongo db, 只要先啟動mongo db, 然後再啟動sails lift就沒問題了!

