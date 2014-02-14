#Nginx

## nginx 499 状态码说明

1. nginx 499 状态码通常是由于 client -> nginx 其中由于server的处理时间较长,client 由于超时主动关闭连接. 
2. 在一个upstream出错，执行next_upstream时也会判断连接是否可用，不可用则返回499。
3. upstream 在收到读写事件处理之前时，会检查连接是否可用：ngx_http_upstream_check_broken_connection
	if (c->error) { //connecttion错误
　　　　 ……
       if (!u->cacheable) { //upstream的cacheable为false，这个值跟http_cache模块的设置有关。指示内容是否缓存。
       ngx_http_upstream_finalize_request(r, u, NGX_HTTP_CLIENT_CLOSED_REQUEST);
       }
	}
    
 ** 这个错误的比例升高可能表明服务器upstream处理过慢，导致用户提前关闭连接。而正常情况下有一个小比例是正常的。 **
 
 refer: http://www.cnblogs.com/xiaohuo/archive/2012/08/09/2630305.html
