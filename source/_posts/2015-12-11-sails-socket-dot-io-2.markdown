---
layout: post
title: "Sails Socket.io基本概念 - rooms and streams"
date: 2015-12-11 15:30:30 +0800
comments: true
categories: [sails, socket.io]
---

好的, 這一篇是接著**stenio123**老大的第二章節, 也就是 <a href="http://impacta.us/sails-with-socket-io-part-24/" target="_blank">part 2 – Sails js Socket.io intermediate – rooms and streams</a>

這一篇的練習做完, 大致上就可以掌握sails socket.io的百分之七十的用法了

我們先把程式碼寫一寫

這個程式碼是接續自上一篇, 我們繼續在socket_test的專案下執行

<!--more-->

	sails generate controller TeamNews

然後修改一下檔案

```js api\controllers\TeamNewsController.js

/**
 * TeamNewsController
 *
 * @description :: Server-side logic for managing Teamnews
 * @help        :: See http://sailsjs.org/#!/documentation/concepts/Controllers
 */

module.exports = {
    index:function (req, res) {
        var teamNames = [{name:'blackhawks'}, {name:'lightning'}];
        var hawks_fans = sails.sockets.subscribers('blackhawks');
        var lightning_fans = sails.sockets.subscribers('lightning');


        res.view({
            teams: teamNames,
            hawksFans: hawks_fans,
            lightningFans: lightning_fans
        });

    },
    broadcastnews: function (req, res) {
        var data = {};
        data.name = req.param("team");
        data.news = req.param("news");

        var rooms = sails.sockets.rooms();
        var hawks = sails.sockets.subscribers("blackhawks");
        var lightning = sails.sockets.subscribers("lightning");

        console.log("Available rooms: " + JSON.stringify(rooms));
        console.log("subscribers hawks: " + JSON.stringify(hawks));
        console.log("subscribers lightning: " + JSON.stringify(lightning));

        sails.sockets.broadcast(data.name, "news", data);
    },
    blastnews: function (req, res) {
        var news = req.param("news");
        sails.sockets.blast('news', news);
    },
    //this is how we keep track of how many sockets each team has in their room,
    joinroomfans: function (req,res) {
        console.log("joining room fans socket id" + req.socket.id);
        sails.sockets.join(req.socket, 'blackhawksFans');
        sails.sockets.join(req.socket, 'lightningFans');
        sails.sockets.broadcast('blackhawksFans', 'updatedblackhawks', sails.sockets.subscribers('blackhawks').length);
        sails.sockets.broadcast('lightningFans', 'updatedlightning', sails.sockets.subscribers('lightning').length);
    },
    //this is how a socket joins room to receive news of their team
    joinroom:function (req,res) {
        var teamName = req.param("teamName");
        sails.sockets.join(req.socket,teamName);
        console.log( 'Socket ' + req.socket.id + 'joined room: ' + teamName);
        //teamName+'Fans' is name of the room, 'updated'+teamName is name of event
        sails.sockets.broadcast(teamName+'Fans', 'updated'+teamName, sails.sockets.subscribers(teamName).length);
    },
    //leave room from team news
    leaveroom:function (req,res) {
        var teamName = req.param("teamName");
        sails.sockets.leave(req.socket,teamName);
        console.log( 'Socket' + req.socket.id + 'left room: ' + teamName);
        //teamName+'Fans' is name of the room, 'updated'+teamName is name of event
        sails.sockets.broadcast(teamName+'Fans', 'updated'+teamName, sails.sockets.subscribers(teamName).length);
    }
};


```

接著新增`views\teamnews\index.ejs`

```js views\teamnews\index.ejs

<h1>Sails js with socket io tutorial - part 2</h1>
You can check the tutorial <a href="http://www.impacta.us/blog">here</a>.
<a href="/user/">Part1 </a> Part2 Part3 Part4
<p>

<% if (teams) { %>
  <select id="teamNameSelect">
    <option value="0" selected="selected" style="display:none;">Select team</option>
     <% _.each(teams, function(team) {  %>
    <option value="<%=team.name%>"><%=team.name%></option>
    <% }) %>
  </select>
<% } %>
<input type="text" id="news" name="news" placeholder="type team news here"/>
<button id="submit">Submit team news!</button> or <button id="blast">Blast to everyone</button>
</p>
<h3>Subscribe to Chicago Blackhawks news</h3>
# of subscribed fans: <div id="blackhawksFans"></div>
<br/>
<button id="gohawks">Go Hawks!</button>
<ul id="blackhawks">
</ul>
<button id="nohawks">Unsubscribe to Hawks!</button>
  
<h3>Subscribe to Tampa Lightning news</h3>
# of subscribed fans: <div id="lightningFans"></div>
<br/>
<button id="gotampa">Go Tampa!</button>
<ul id="lightning">
</ul>
<button id="notampa">Unsubscribe to Tampa!</button>

 <!-- END Page Content -->
<% block('localScripts', '<script src="/view_specific/teamnews_index_misc.js"></script>') %>

```

再來是`assets\view_specific\teamnews_index_misc.js`

```js assets\view_specific\teamnews_index_misc.js

//when page loads, register this socket to receive notification of new fans(sockets) subscribed
$(document).ready(function () {
    io.socket.post('/teamnews/joinroomfans/', {});
});

/** event handling for Fan room */
io.socket.on('updatedblackhawks', function (response) {
    console.log(" fan response is " + JSON.stringify(response));
    $("#blackhawksFans").html(response);
});
io.socket.on('updatedlightning', function (response) {
    console.log(" fan response is " + JSON.stringify(response));
    $("#lightningFans").html(response);
});

//adds listener and what to do once receives event notification
io.socket.on('news', function (response) {
    console.log("response is " + JSON.stringify(response));
    //if it is a sails.socket.broadcast
    if (response.name) {
        if (response.name === 'blackhawks') {
            $("#blackhawks").append('<li>' + response.news + '</li>');
        } else {
            if (response.name === 'lightning') {
                $("#lightning").append('<li>' + response.news + '</li>');
            }
        }
        //if it is sails.socket.blast
    } else {
        $("#blackhawks").append('<li>' + response + '</li>');
        $("#lightning").append('<li>' + response + '</li>');
    }
})

//subscribes to Blackhawks messages (code on TeamNewsController.subscribe)
$("#gohawks").click(function () {
    io.socket.post('/teamnews/joinroom/', {teamName: 'blackhawks'});
    alert("subscribed to Hawks news!");
});
//unsubscribes to Blackhawks messages (code on TeamNewsController.subscribe)
$("#nohawks").click(function () {
    io.socket.post('/teamnews/leaveroom/', {teamName: 'blackhawks'});
    alert("unsubscribed to Hawks news!");
});

//subscribes to Lightning messages (code on TeamNewsController.subscribe)
$("#gotampa").click(function () {
    io.socket.post('/teamnews/joinroom/', {teamName: 'lightning'});
    alert("subscribed to Lightning news!");
});
//unsubscribes to Lightning messages (code on TeamNewsController.subscribe)
$("#notampa").click(function () {
    io.socket.post('/teamnews/leaveroom/', {teamName: 'lightning'});
    alert("unsubscribed to Lightning news!");
});

$("#submit").click(function () {
    if ($("#teamNameSelect").val() != "0") {
        io.socket.post('/teamnews/broadcastnews/', {
            team: $("#teamNameSelect").val(),
            news: $("#news").val()
        });
    } else {
        alert("please select team!");
    }
});

$("#blast").click(function () {
    io.socket.post('/teamnews/blastnews/', {
        news: $("#news").val()
    });

});

```

最後修改一下`config\routes.js`, 將首頁設定成TeamNews的index

```js config\routes.js

 '/':{
        controller: 'TeamNews',
        action: 'index'
     }

```

OK, 大功告成.

存檔, 從新啟動sails.

	sails lift

然後在瀏覽器輸入localhost:1337

就可以開始來玩玩看嚕!

記得開啟瀏覽器的除錯模式(例如firefox的firebug)

觀察前端以及後端的log


###參考資料

* <a href="http://sailsjs.org/documentation/reference/web-sockets/socket-client" target="_blank">Socket Client (sails.io.js)</a>
* <a href="http://sailsjs.org/documentation/concepts/controllers/generating-controllers" target="_blank">Generating controllers</a>