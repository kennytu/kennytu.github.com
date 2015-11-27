---
layout: post
title: "Mongo Database 簡介以及GUI介面使用"
date: 2015-11-27 19:25:17 +0800
comments: true
categories: mongodb
---

在進行sails練習的下一章之前, 有需要對Mongo Database做一些簡單的練習. 

由於我看到Irl nathan是使用<a href="http://genghisapp.com/" target="_blank">Genghis</a>來作為Mongo DB的Web GUI介面, 但是是在MAC OS上安裝的, 我想在Win7上面也來裝一下, 以方便連接Mongo DB, 但天殺的在win7上安裝遇到太多問題, 甚至還要修改C語言的程式碼....

我review了一下genghis, 發現這個作者在2014年年底就沒有再維護了...

奮戰了三到四個小時之後, 我更改戰略, 到mongodb的官網看看還有沒有其他更適合的WEB ADMIN GUI可以用

所以我改用了<a href="https://github.com/andzdroid/mongo-express" target="_blank">mongo-express</a>

這篇文章就是在介紹mongo-express怎麼使用(最基本的)

<!--more-->

若要看看其他Mongo database的其他延伸應用, 請參考<a href="https://docs.mongodb.org/ecosystem/tools/administration-interfaces/" target="_blank">MongoDB Integration and Tools > Admin UIs</a>

說實在的我之前都是用SQL的, 對於NOSQL還真的沒啥概念. 

底下開始介紹Mongo Database

##我的版本

npm版本 `2.14.4`, 使用`npm -v` 取得

node版本 `v4.1.1`, 使用`node -v` 取得

mongod版本:

	db version v3.0.7
	git version: 6ce7cbe8c6b899552dadd907604559806aa2e9bd

使用 `mongod.exe --version` 取得

mongo-express版本 `0.27.3`, 使用`npm list mongo-express -g` 取得

**其實MongoDB的版本2.4, 版本2.6以及3.x版本的改動很大... 也沒有向下相容, 所以在學習上頗有曲線**


##安裝Mongo Database

基礎的安裝我描述在**Sails練習之八-連結資料庫**, 可以參考. 基本上我是安裝到`global`


##Mongo Database基本指令

###Server端常用指令

`mongod.exe -d user_path_for_db` 

###Client端常用指令
在MongoDB的安裝路徑下的bin目錄, 輸入`mongo.exe`, 進入互動CMD介面

`db` : 列出現在使用的database

`show dbs` : 列出所有的databases

`show users` :  列出所有的用戶

`use [database]` : 使用use切換到指定的database, ex: `use test`


## Server Side
啟動伺服器端, 在 `MongoDB\bin`目錄下執行`mongod.exe` 就可以了, 記得要定義預設的資料庫存放路徑, 例如`/data/db` (若Mongo DB的執行檔在`C槽`, 那就是`C:\data\db`, 若沒有定義預設存放路徑, 則需要指名存放資料庫的路徑, 例如`D:\MongoDB\bin>mongod.exe --dbpath=D:\MongoDB\data`

另外, 如果要讓Client連進來的時候需要Auth, 則在啟動mongo的時候, 在後面加入`--auth` 就可以了, 例如 `mongod.exe --auth`

基本上為了方便起見, 我都**先不加**`auth`

請參考<a href="https://docs.mongodb.org/manual/tutorial/enable-authentication/" target="_blank">Enable Client Access Control</a> 

##新增使用者在MongoDB

進入Mongo Client Tool, 然後輸入底下指令

	use admin
	db.createUser({user:"root",pwd:"kenny888",roles:[]})

這樣就可以了, 詳細的官方網站步驟請到底下兩個網頁

* <a href="https://docs.mongodb.org/manual/tutorial/enable-authentication/" target="_blank">Enable Client Access Control</a>
* <a href="https://docs.mongodb.org/manual/tutorial/manage-users-and-roles/" target="_blank">Manage User and Roles</a>

參考以上兩個網頁的步驟照著做也可以.


好的, 這樣我們就已經有一個在Admin資料庫的使用者root了

接下來要來安裝`mongo-express`啦

###安裝mongo-express

	npm install -g mongo-express

安裝的時候須注意, mongo-express在win7上面可能需要visual studio 2013的編譯器來編譯一些C程式碼, 若沒有安裝visual studio 2013, 請先安裝完成之後再回頭來install mongo-express

接著, 因為我們是把mongo-express安裝到global去, 我們要找到安裝路徑, 你可以使用`npm list -g`來找出安裝路徑(會show在第一行), 在我的電腦就是`C:\Users\KennyTu\AppData\Roaming\npm`

好的, 接下來切換路徑到`C:\Users\KennyTu\AppData\Roaming\npm\node_modules\mongo-express`

`copy`一份`config.default.js`到`config.js`

然後修改`config.js`

```js config.js

	...

 	//leave username and password empty if no admin account exists
    adminUsername: 'root',
	adminPassword: 'kenny888',
	...
	basicAuth: {
    	username: process.env.ME_CONFIG_BASICAUTH_USERNAME || 'root',
    	password: process.env.ME_CONFIG_BASICAUTH_PASSWORD || 'kenny888'
  	},

	...

```

其實這一份config.js我還有很多東西沒有搞懂, 但先這樣設定, 我們的mongo-express就可以run了

有時候設定錯誤, 執行mongo-express會直接跳出來給你看, 沒有任何錯誤訊息...滿無言的

好,設定完之後, 存檔, 我們直接打開CMD

輸入`mongo-express`, 若沒有問題, 它會shows出底下訊息, 

	Mongo Express server listening on port 8081 at 0.0.0.0
	Database connected!
	Admin Database connected

> 記得輸入mongo-express之前, 要先執行mongod (server), 這樣mongo-express才不會出錯

大功告成啦

這時候打開瀏覽器, 輸入`localhost:8081` 應該就看得到Web版本的管理頁面了!!!



最後, 附上網路上找到的一些Mongo DB的觀念


##mongod 概念
 
一個 mongod 服務可以有建立多個數據庫，每個數據庫可以有多張表，這裡的表名叫 collection，每個 collection 可以存放多個文檔 (document)，每個文檔都以 BSON(binary json) 的形式存放於硬盤中，因此可以存儲比較復雜的數據類型。它是以單文檔為單位存儲的，你可以任意給一個或一批文檔新增或刪除字段，而不會對其它文檔造成影響，這就是所謂的 schema-free，這也是文檔型數據庫最主要的優點。跟一般的 key-value 數據庫不一樣的是，它的 value 中存儲了結構信息，所以你又可以像關系型數據庫那樣對某些域進行讀寫、統計等操作。Mongo 最大的特點是他支持的查詢語言非常強大，其語法有點類似於面向對象的查詢語言，幾乎可以實現類似關系數據庫單表查詢的絕大部分功能，而且還支持對數據建立索引。Mongo 還可以解決海量數據的查詢效率，根據官方文檔，當數據量達到 50GB 以上數據時，Mongo 數據庫訪問速度是 MySQL10 倍以上。
 
##BSON
 
BSON 是 Binary JSON 的簡稱，是一個 JSON 文檔對象的二進制編碼格式。BSON 同 JSON 一樣支持往其它文檔對象和數組中再插入文檔對象和數組，同時擴展了 JSON 的數據類型。如：BSON 有 Date 類型和 BinDate 類型。
 
BSON 被比作二進制的交換格式，如同 Protocol Buffers，但 BSON 比它更 「schema-less」，非常好的靈活性但空間佔用稍微大一點。
 
BSON 有以下三個特點：輕量級、跨平台、效率高
 
##命名空間
 
MongoDB 存儲 BSON 對象到 collections, 這一系列的數據庫名和 collection 名被稱為一個命名空間。如同：java.util.List; 用來管理數據庫中的數據。
 
##索引
 
mongodb 可以對某個字段建立索引，可以建立組合索引、唯一索引，也可以刪除索引，建立索引就意味著增加空間開銷。默認情況下每個表都會有一個唯一索引：_id，如果插入數據時沒有指定_id，服務會自動生成一個_id，為了充分利用已有索引，減少空間開銷，最好是自己指定一個 unique 的 key 為_id，通常用對象的 ID 比較合適，比如商品的 ID。


##參考資料

<a href="http://www.ithome.com.tw/news/92506" target="_blank">了解 NoSQL 不可不知的 5 項觀念</a>

<a href="http://www.neohope.com/2015/09/12/acid%E3%80%81base%E4%B8%8Ecap/" target="_blank">ACID、BASE 與 CAP</a>

<a href="https://en.wikipedia.org/wiki/List_of_column-oriented_DBMSes" target="_blank">List of column-oriented DBMSes</a>

<a href="https://zh.wikipedia.org/wiki/%E5%88%97%E5%BC%8F%E6%95%B0%E6%8D%AE%E5%BA%93" target="_blank">Row-oriented DBMS</a>

<a href="https://en.wikipedia.org/wiki/Column-oriented_DBMS" target="_blank">Column-oriented DBMS</a>