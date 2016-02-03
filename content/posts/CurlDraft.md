+++
date = "2014-12-25T14:13:36+08:00"
draft = false
title = "CurlDraft"

+++

## CURL 有用操作
### curl post 数据

1. 提交 key:value 格式数据
	
		curl -d 'username=xyz&password=xyz' http://localhost:3000/api/login
2. 提交 json格式数据

		curl -d '{"username":"xyz","password":"xyz"}' http://localhost:3000/api/login
3. 参数内容较多,直接写入到一个文件中

		curl -d @param.json  http://localhost:3000/api/login
4. 上传附件

		curl -F "blob=@card.txt;type=text/plain" http://localhost:3000/upload
<!--more-->

5. 指定 curl 方法
我们在执行的时候, 需要指定 HTTP 对应的方法

        curl -X DELETE  <URL>
