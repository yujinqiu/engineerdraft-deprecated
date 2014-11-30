+++
date = "2014-11-25T21:36:12+08:00"
draft = false
title = "Mac os X Draft"
description = "Mac os X 手稿"
+++

#Mac os X
##Yosemite 升级之后应该进行什么操作?  
1. 从 mavericks 升级到 mac os x 之后, 会遇到一些比较诡异的问题(比如找不到 clang), 下面的命令会帮助你提前避免一些问题. 

```bash
sudo xcode-select --install  
sudo xcode-select --switch /Applications/Xcode.app   
sudo xcodebuild -license  
sudo gem update --system  
sudo gem install xcodeproj   
sudo brew update   
```
<!--more-->

2. 同时 Yosemite 会收集用户的隐私数据, 建议通过[fix-macosx](https://fix-macosx.com/)进行修复


### Mac os X 窗体最小化之后恢复  
Mac os X 在最小化窗体之后, `command + tab` 是不能够直接恢复, 具体操作如下   
command + tab 定位到对应的 Application 然后 `alt` 然后同时放开 `alt command`


### 清除 DNS cache

	sudo dscacheutil -flushcache