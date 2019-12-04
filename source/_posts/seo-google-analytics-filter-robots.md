---
title: Google Analytics 設定過濾 bot 及爬蟲流量
date: 2019-12-04 22:20:07
tags: google-analytics,seo
categories:
---

幾年前在第一份工作時，有幫忙公司研究 SEO 相關設定，最近因為架了部落格，重新對部落格再作調整，將之前的東西慢慢補上來。

這篇其實是 Google Analytics 的 master 預設設定，所以只是簡單說明一下在哪開關及作用。

基本上 master view 的是留原始資料的，我們如果要去蒐集過濾不同的資料檢視，最好是新建 view 來使用，如果有特別需求想蒐集爬蟲資訊的話可以開啟這個選項。

網路上有許多人或公司在蒐集各種網站資訊，至於他們怎麼使用資訊就不得而知了，但是要蒐集資訊用人力蒐集是最慢的，所以就有機器人 (robot) 或爬蟲 (crawler) 程式來負責蒐集各網站資訊。

而好的機器人或爬蟲都會遵守規則，在蒐集資訊時告知對方 (伺服器) 說我是程式來檢索資訊的，以便伺服器判斷如何使用，通常會有 robots.txt 規則給檢索程式知道，哪些可以蒐集、哪些不用蒐集，其實也只是建議而已，如果程式不遵守規定、不自報家門、不理規則，他一樣可以把你公開在網路上沒擋權限的資訊蒐集回家。

Google Analytics 這個設定就是依此判斷是否為活人或蒐集程式來過濾資訊，讓報表呈現更符合真實情況，才不會報表數據非常好看，但其實沒有半個活人在用你的網站。

## 步驟

1. 進入左邊選單最下面的管理頁面

{% asset_img ga-excludes-all-hits-from-bots-0.png ga-excludes-all-hits-from-bots-0 %}

2. 選擇希望變更的 View 資料檢視設定

{% asset_img ga-excludes-all-hits-from-bots-1.png ga-excludes-all-hits-from-bots-1 %}

3. 開啟或關閉 `排除所有來自已知漫遊器和自動尋檢程式的匹配(Exclude all hits from known bots and spiders)`

{% asset_img ga-excludes-all-hits-from-bots-2.png ga-excludes-all-hits-from-bots-2 %}
