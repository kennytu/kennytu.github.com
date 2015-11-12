---
layout: post
title: "Octopress的進階技巧"
date: 2015-11-12 20:06:58 +0800
comments: true
categories: octopress
---

這篇Blog我記錄了一些我在網路上找到的資料, 很多分享, 也滿受用的

我整理在底下, 內容有

1. 進階的安裝文章分享
2. 多台電腦上寫作同步
3. 加上分類雲
4. 程式碼高亮
5. imgcap的使用

<!--more-->

## Octopress一些進階的安裝文章分享

1. <a href="https://wwssllabcd.github.io/blog/2012/08/01/how-to-install-octopress-on-window/" target="_blank"> Octopress 安裝教學、心得筆記 (Windows)</a>
	* 提到如何分類, 以及分類模組安裝
2. <a href="http://shengmingzhiqing.com/blog/octopress-lean-modification-1.html/" target="_blank"> Octopress 精益修改 (1) </a>
	* 一些進階的應用


## Octopress 重新安裝或者在多台電腦上寫作同步

主要是參考這篇 <a href="http://ju.outofmemory.cn/entry/119189" target="_blank"> Set up Octopress environment on another computer </a>

簡單描述如下

1. 先將github上存放blog的repo, clone到本地端的octopress來, 注意clone的branch是source branch

		git clone -b source git@github.com:kennytu/kennytu.github.com.git octopress

2. 接下來進入octopress, 將master branch的文件cloen到_deploy
		
		cd octopress
		git clone git@github.com:kennytu/kennytu.github.com.git _deploy

不用加-b的選項, 預設就會是master

接著執行以下指令, 要在octopress目錄下

		gem install bundler
		bundle install

這樣就可以了

PS: 注意不要使用rake install 默認主題，不然會把自定義的主題恢復到默認狀態

最後, 在octopress目錄下, 執行
		
	cd octopress
	git pull origin source  
	cd ./_deploy
	git pull origin master 
  

OK, 大功告成


## 替octopress加上分類列表(分類雲)

參考 <a href="https://github.com/tokkonopapa/octopress-tagcloud" target="_blank"> octopress-tagcloud</a>

1. 右下方點選Download zip
2. 解壓縮到octopress相同目錄下, 覆蓋相同資料夾(不用擔心其他檔案會不見)
3. 修改`_config.yml`, 找到 `default_asides`, 加入 __custom/asides/category_list.html__ 這個項目到List當中, 如下
		
		default_asides: [custom/asides/category_list.html, ...]

4. 執行`rake generate`

在文章的分類部分, 有下列幾個分法

單一分類
categories: abc

多重分類: 使用LIST
	
	categories: [aaa, bbb, ccc]

多重分類: 使用 __-__ 

	categories:
	- aaa
	- bbb
	- ccc


## 程式碼高亮語法

參考 <a href="http://octopress.org/docs/blogging/code/" target="_blank"> Sharing Code Snippets </a>

這個功能需要安裝Python

``` ruby Discover if a number is prime http://www.noulakaz.net/weblog/2007/03/18/a-regular-expression-to-check-for-prime-numbers/ 點擊查看
	class Fixnum
	  def prime?
	    ('1' * self) !~ /^1?$|^(11+?)\1+$/
	  end
	end
```

語法是 三個``` [language] [title] [url] [link text]

	
	``` ruby Discover if a number is prime http://www.noulakaz.net/weblog/2007/03/18/a-regular-expression-to-check-for-prime-numbers/ 點擊查看

	class Fixnum

	  def prime?

	    ('1' * self) !~ /^1?$|^(11+?)\1+$/

	  end

	end
	
	```
 
## 連結圖片

參考 <a href="http://blog.zerosharp.com/image-captions-for-octopress/" target="_blank"> Image Captions for Octopress </a>

將相對應的程式碼修改一下, 就可以使用imgcap的語法, 例如

\{ % imgcap /images/dog.jpg 看三小朋友 % \}

就可以在圖片底下加上說明

若圖檔在本地端, 只要將圖檔放在source底下的images目錄, 然後引用就可以了

效果如下

{% imgcap /images/ttt.jpg 看三小朋友 %}


