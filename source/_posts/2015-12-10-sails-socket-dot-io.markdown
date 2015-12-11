---
layout: post
title: "Sails Socket.io基本概念 - watch and subscribe"
date: 2015-12-10 11:09:01 +0800
comments: true
categories: [sails, socket.io]
---

OK, 由於Irl先生接下來會用到socket.io, 整合到activityOverlord專案中

因為Irl先生是用舊版的Sails, 也就是Sails 0.9的, 而在Sails 0.11之後, sails對於socket.io的支援改動滿大的

所以我就上網找了一些相容於sails 0.11的專案來練習

先講幾個用法上的觀念

1. Sails已經將socket.io整合進自己的架構, 你可以在`assets\js\dependencies\`找到`sails.io.js`這個檔案, 這個檔案就是socket.io的精隨了
2. Sails將socket.io的使用整合進它自己的MVC架構, 也就是說, 我們在操控socket.io這些方法的時候, 大部分是在Controller這邊處理, 我們會在後面的例子中談到
3. Sails針對socket.io的用法有提供socket.io模式的Method,也有提供Sails本身Data Module的Method, 看你要用哪一種

<!--more-->

以上幾個觀念, 詳細請參考<a href="http://sailsjs.org/documentation/reference/web-sockets/socket-client" target="_blank">Socket Client (sails.io.js)</a>, 老實說文件寫得有夠精簡的.... 看完也不知道怎麼用

好的, 接下來我們用例子來讓大家有感覺

本篇的實作是參考**stenio123**大哥的專案, 他的專案程式碼都放在他的Github, 請參考 <a href="https://github.com/stenio123/sails-socket-example" target="_blank">sails-socket-example</a>

他也有寫了一個網誌來教學, 請看 <a href="http://impacta.us/sails-with-socket-io-part-14/" target="_blank">Sails with socket.io</a>, 不過他好像只寫了前兩篇, 後兩篇就沒著落了.

讓我們開始, 先建立一個Sails專案

	sails new socket_test

	cd socket_test

先試試看可不可以Run
	
	sails lift

開啟瀏覽器, 輸入`localhost:1337` 看看結果

注意! 你在執行sails lift的時候有可能會發生缺少什麼module, 請看我上一篇網誌**NPM的怪問題**獲得可能的解答

第一次執行的時候, Sails會問你關於資料庫的存取問題, 到`config`目錄下的`models.js`, 把`migrate`改成 `'safe'`即可

OK, 沒問題的話, 接下來就開始來寫Code了

	socket_test> sails generate api User

Sails會幫我們在`api`目錄下的`controllers`以及`models`這兩個目錄下, 產生相對應的檔案

我們先修改`api\controllers\UserController.js`, 把裡面的東西全刪掉, 將下面的程式複製貼上

```js UserController.js

module.exports = {
    watchUser: function (req, res) {
        // watch for new User instances
        User.watch(req.socket);
        console.log('User watching ' + req.socket.id);
    },
    subscribeUser: function (req, res) {
        // subscribe client to model changes
        User.subscribe(req.socket);
        console.log('User subscribed to ' + req.socket.id);
    },
    createpub: function (req, res) {
        var data_from_client = req.params.all();

        if (req.isSocket && req.method === 'POST') {
            User.create(data_from_client)
                .then(function (data_from_client) {
                    console.log(data_from_client);
                    User.publishCreate({id: data_from_client.id, name: data_from_client.name});
                })
                .catch(function (err) {
                    console.log('Error creating new User. Error: ' + err);
                    return res.serverError('Server encountered a problem. Please contact administrator.');
                });
        }
        else {
            return res.badRequest('Sorry, this endpoint only accessible via sockets.');
        }
    },
    index: function (req, res) {
        res.view({
            welcomeMessage: 'Welcome to Sails + socket.io simple test. Please refrain from adding client logic on the server!'
        });
    }

};

```

接著, 我們到`views`底下新增一個`user`的目錄, 在`user`目錄新增一個`index.ejs`的檔案

```js views\user\index.ejs

<%= welcomeMessage %>
<br/>
<p>
	<input type="text" id="name">
	<button id="go-data">Submit!</button>
</p>
<br/>
<ul id="user-list">
</ul>

<% block('localScripts', '<script src="/view_specific/user_index_misc.js"></script>') %>

```

這個程式碼解釋一下, Sails有提供layout, partial and block template functions for the EJS template engine. 

其實就是express 3.x然後使用ejs-local的範本(因為sails就是架構在express上面的呀)

其中`<% block ... %> `, 裡面的意思是說, 這個local的javasript, 就只有給這個index.ejs用, 不會被其他的網頁包含

這個技巧在動態網頁互動很常使用到, 所以可以學習一下

要很注意的是, 我在`<% block` 這個標籤, 並沒有加上 **-** 這個, 

也就是說, 我沒有用`<%- block ... %> `這個加上**-**的方式, 網路上有不少人都會加上這個dash sign, 但這會造成一些錯誤, 我們會在附錄中解說

但在sails的使用上, 我們還得注意到另外一個地方, 若要使用這個`<% block ... %> `標籤, 我們需要去修改`layout.ejs`

在`<!--SCRIPTS END-->`下面, 加上一段話, 程式碼如下

```js views\layout.ejs

    <!--SCRIPTS-->
    <script src="/js/dependencies/sails.io.js"></script>
    <script src="/js/dependencies/jquery-1.11.1.min.js"></script>
    <!--SCRIPTS END-->
    <%- blocks.localScripts %>

  </body>
</html>

```

注意到我們在`<!--SCRIPTS END-->`下加入了`<%- blocks.localScripts %>` 

簡單介紹一下layout.ejs, 被`<!--SCRIPTS-->`和`<!--SCRIPTS END-->`包住的這兩行程式碼, 是Sails自動產生的,

Sails會根據我們放在`assets\js\dependencies`裡面的檔案, 把這些檔案的檔名掃描進來, 放到`<!--SCRIPTS-->`和`<!--SCRIPTS END-->`中間

那你會想說, 如果我的javascript有先後順序的話, 我要怎麼調整順序呢?

請到`tasks\pipeline.js`, 找到`jsFilesToInject`這個區段, 參考底下

```js pipeline.js

// Client-side javascript files to inject in order
// (uses Grunt-style wildcard/glob/splat expressions)
var jsFilesToInject = [

  // Load sails.io before everything else
  'js/dependencies/sails.io.js',

  // Dependencies like jQuery, or Angular are brought in here
  'js/dependencies/**/*.js',

  // All of the rest of your client-side js files
  // will be injected here in no particular order.
  'js/**/*.js',

  // Use the "exclude" operator to ignore files
  // '!js/ignore/these/files/*.js'
];

```
若要更改順序, 就把你想要的javascript按照寫在這邊, 就可以了, 上面的說明應該滿清楚的

好的, views/user/index.ejs完成之後, 我們要來處理Client端的介面

到`assets`這個路徑下, 新增一個`view_specific`的目錄, 然後在此目錄下新增`user_index_misc.js`這個檔案

檔案內容如下

```js user_index_misc.js

//sends this socket id so it can be subscribed at the controller
io.socket.get('/user/watchUser', function(resData, jwres) {
    console.log(resData);
});

//adds listener and what to do once receives event notification
io.socket.on('user',function(obj){
    //if(obj.verb ==='created'){
    console.log('chat event is '+ JSON.stringify(obj));
    $("#user-list").append('<li>' + obj.data.name + '</li>');
});

$("#go-data").click( function() {
    io.socket.post('/user/createpub/',{name:$("#name").val()});
    $("#name").val("");
});

```

新增完成之後, 最後一件事情就是, 將Jquery放到`assets\js\dependencies\`

也就是上去Jquery的網站, 去下載Jquery的檔案, 1.x版本或者2.x版都可以

例如我這邊就是`assets\js\dependencies\jquery-1.11.1.min.js`

下載完放好之後, 你就會在layout.ejs裡面看到sails已經將jquery-1.11.1.min.js放進去了

OK, 事前準備終於完成了

啊! 還有一個要改, 就是首頁的Route

到`config\routes.js`, 把`view: 'homepage'`拿掉, 改成以下

```js config\routes.js

    '/':{
        controller: 'User',
        action: 'index'
    }

```

這樣就確保我們的首頁就是User底下的index了

現在, 執行以下指令

	sails lift

開啟瀏覽器, 輸入localhost:1337

會出現一個讓我們輸入client的畫面, 我們使用firefox的firebug開啟除錯視窗, 然後觀察輸出的訊息 (也要觀察後台的訊息唷)

若沒有問題的話, 這就是我們stenio123大哥的第一部分的示範


##附錄

上面提到Sails有提供layout, partial and block template functions for the EJS template engine.

然後提到`<%- block ... %> `這個加上**-**的方式

這邊有個觀念是, 在ejs中, 若使用 `<%-`這個有dash符號的, 那麼ejs就會去執行被`<%-`包含起來的javascript程式碼. 專業一點的說法就是, 我們可以藉由這個`<%-`符號, 來注入(inject)我們的javascript的程式碼.

另外一方面, 為什麼提到上述我們在使用`<% block`這個tag的時候, 不能加dash呢? 是因為若我們使用了`<%- block`, ejs會做兩次動作, 第一次是先執行要注入的程式碼, 也就是在`<%- block`包含的javascript, 然後, 將此段javascript轉為此頁面的一部分, 結果, 當瀏覽器執行到此javascript的時候, 又會在執行一次, 結果造成了執行兩次的誤動作.

這樣會有什麼問題呢? 

一個問題就是, 當ejs在執行第一次的注入javascript的時候, 若這個javascript有相依其他的javascript, 例如JQuery的時候, 它會發生找不到相依的錯誤, 直到第二次, 當瀏覽器依照正確的順序將相依的javascript匯入的時候, 第二次執行就沒有問題...

以上的討論可以參考 <a href="http://stackoverflow.com/questions/22112763/sails-js-how-to-inject-a-js-file-to-a-specific-route" target="_blank">Sails.js - How to inject a js file to a specific route?</a> 這個討論串

另外, 關於EJS的一些概念, 請參考底下網站

* <a href="http://www.embeddedjs.com/getting_started.html" target="_blank">Getting Started with EJS</a>

* <a href="http://blogger.gtwang.org/2014/02/ejs-embedded-javascript.html" target="_blank">EJS：Client 端嵌入式（Embedded）JavaScript</a>

* <a href="http://ccc.nqu.edu.tw/wd.html#wp:ejs" target="_blank">ejs 樣板引擎</a>

* <a href="http://www.embeddedjs.com/screencast/index.html" target="_blank">Clean AJAX with EJS</a>



##Sails不錯的簡報

* <a href="http://www.slideshare.net/stenio123/sails-js-intro-impactaus" target="_blank">ails-js-intro-impactaus</a>, 第21頁, 很棒的介紹!














