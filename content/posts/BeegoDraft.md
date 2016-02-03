+++
date = "2014-12-27T20:49:46+08:00"
draft = false
title = "BeegoDraft"

+++

## beego session 失效时间  
### 背景 
使用 beego 进行开发过程中为了减少大家登陆时间特定设置1个月有效期, 结果发现需要经常登陆
### 原因
session 的过期失效时间由两个方面决定    

1. 存储sessionid 的 cookie (SessionCookieLifetime) 
2. 存储在服务器端的 session 文件( SessionGCMaxLifetime 默认3600) 

<!--more-->
开始的时候只是设置了 SessionCookieLifetime, 虽然 cookie 的内容不会被删除, 但是在服务器端的 session 文件会被删除, 在代码中    

	ctx.Inut.Session("username").(string)
	
无法获取对应的内容, 重定向到登陆页面.
