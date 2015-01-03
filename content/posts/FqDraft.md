+++
date = "2014-12-28T11:42:25+08:00"
draft = false
title = "fq"

+++

## FQ
> Across the Great Wall we can reach every corner of the world.  


### shadowsocks (ss)
#### 原理
	app <- 本地 client 解密 <- 墙外 vps(加密) <- 目标 site  
本质上 sshd -D 就是一个 Shadowsocks . 

	ssh -D 0.0.0.0:10086 user@vps
	
#### 注意事项

1. 
ssh -D 的本质上是一个 sock5 的代理, 不是 http/https 代理, 所以在 firefox 配置里边**不能**够把 HTTP Proxy 和 SSL Proxy 配置为`127.0.0.1:10086`, 只能设置socks 代理
2. 为了避免 [DNS 污染](http://en.wikipedia.org/wiki/DNS_spoofing), 建议在 firefox 中设置`Remote DNS`, 进行远端解析

### 相关命令代理
#### curl

	--socks5-hostname <host[:port]> 
		Use the specified SOCKS5 proxy (and let the proxy resolve the hostname) 避免 DNS 污染
	
	--socks5 <host[:port]>
		Use the specified SOCKS proxy - but resolve the hostname locally. 
		
建议在使用` curl` 的时候习惯加上 `-v` 这样对应 DNS污染问题很容易发现.  

	直接访问(DNS 污染) 
	➜  local git:(master) ✗ curl -v  www.facebook.com
	* Rebuilt URL to: www.facebook.com/
	* Hostname was NOT found in DNS cache
	
	远程DNS, 防火墙直接 reset
		➜  local git:(master) ✗ curl -v --socks5-hostname 127.0.0.1:10086 http://www.facebook.com
	* Rebuilt URL to: http://www.facebook.com/
	* Hostname was NOT found in DNS cache
	*   Trying 127.0.0.1...
	* Connected to 127.0.0.1 (127.0.0.1) port 10086 (#0)
	> GET / HTTP/1.1
	> User-Agent: curl/7.37.1
	> Host: www.facebook.com
	> Accept: */*
	>
	< HTTP/1.1 302 Found
	< Location: https://www.facebook.com/
	< Content-Type: text/html; charset=utf-8
	< X-FB-Debug: fHuVWgR/eAX8mxPiJViAIVoy4HBehbfOkYb7ngC2xmLWoEfEwT0O+gTVFzHI4TJdzaxw8s9IGn4fzKDFN5GtmA==
	< Date: Sun, 28 Dec 2014 06:22:56 GMT
	< Connection: keep-alive
	< Content-Length: 0
	<
	* Connection #0 to host www.facebook.com left intact

	使用 HTTPS 访问, 正常
	



