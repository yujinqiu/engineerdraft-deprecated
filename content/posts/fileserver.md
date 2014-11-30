+++
date = "2014-11-23T10:43:43+08:00"
draft = false
title = "fileserver"
categories = [
	"golang"
]
+++

# 文件服务器
## 背景
我们经常需要下载远程的一台服务器的一个文件, 比如临时线上的某台服务器的一个日志文件需要下载到本地 windows, 没有办法使用 scp, 搭建一个 ftp 又太费劲.  
### 解决方案
#### python   

	python -m SimpleHTTPServer 8080
	
<!--more-->

优点是: 简单 缺点是: 可定制性差.  可能你中间经过代理转发, 需要通过特殊的前缀来区分.  

#### golang

	pakcage main
	
	import (
		"net/http"
		"log"
	)
	
	func main () {
		fileHander := http.FileServer(http.Dir("/tmp"))
		log.Fatal(http.Handle("/tmpfiles/", http.StripPrefix("/tmpfiles/", fileHandler))))
	}
