---
title: GitLab Badge api 用 img.shield 更新
date: 2019-12-10 22:11:38
tags: ['gitlab']
categories:
---

## 情境

最近本來需要在 gitlab project 頁面上做一個更新 badge 資訊的功能，要將 badge 顯示成 環境|commit-sha1 點擊後會連到該 commit 頁面。
普遍作法看起來都是給一個固定網址連到 web api 由 api 來決定圖片樣式及連結資訊，這樣就需要自己寫個簡易的 api 維護就想試看看盡量減少需要維護的程式數量的方式，雖然後來這個方案沒有被使用，但還是紀錄一下這隻 script。

這隻 script 只要輸入 token, project, env, git_sha1 就可以自動幫你的 project badge 加上或更新成類似這樣 dev|955c03e4 ，點擊後還會自動連到該 commit 頁面

## img.shield

這個[服務](https://shields.io/)提供產生靜態或動態 badge 的服務

### static

https://img.shields.io/static/v1?label=<LABEL>&message=<MESSAGE>&color=<COLOR>
https://img.shields.io/badge/test-message-yellow

只要變動 test, message, yellow 為你想要的資訊，就可以產生一個 badge 給你

### dynamic

https://img.shields.io/badge/dynamic/json?url=<URL>&label=<LABEL>&query=<$.DATA.SUBDATA>&color=<COLOR>&prefix=<PREFIX>&suffix=<SUFFIX>

## gitlab badge api

只要修改對應的變數就可以產生或更新 gitlab 上的 project

### 新增

`curl -X POST --header "Private-Token: $GITLAB_TOKEN" "${GET_BADGES_URL}" --data "link_url=$LINK_URL&image_url=$IMG_URL"`

### 更新

`curl -X PUT --header "Private-Token: $GITLAB_TOKEN" "${GET_BADGES_URL}/${badge_id}" --data "link_url=$LINK_URL&image_url=$IMG_URL"`

## References

[script](https://gist.github.com/GhostTW/0763d0f6ef05ab4ce7931fdf43fc586f)