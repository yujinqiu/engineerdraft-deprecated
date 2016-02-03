#Nginx

## nginx 499 状态码说明

1. nginx 499 状态码通常是由于 client -> nginx 其中由于server的处理时间较长,client 由于超时主动关闭连接. 
2. 在一个upstream出错，执行next_upstream时也会判断连接是否可用，不可用则返回499。
3. upstream 在收到读写事件处理之前时，会检查连接是否可用：`ngx_http_upstream_check_broken_connection`

        if(c->error) { //connecttion错误
        ……
             if (!u->cacheable) { //upstream的cacheable为false，这个值跟http_cache模块的设置有关。指示内容是否缓存。
            ngx_http_upstream_finalize_request(r, u, NGX_HTTP_CLIENT_CLOSED_REQUEST);
            }
	    }
    
 **这个错误的比例升高可能表明服务器upstream处理过慢，导致用户提前关闭连接。而正常情况下有一个小比例是正常的。**
 
 refer: [http://www.cnblogs.com/xiaohuo/archive/2012/08/09/2630305.html](http://www.cnblogs.com/xiaohuo/archive/2012/08/09/2630305.html)

## nginx 实现简单密码验证  
### 背景 
大数据组同学在吃饭的时候问了我一个问题, 他们有一个HTTP服务想要发布给部分同学使用,主要用来查看信息 想要加入简单用户名,密码功能.  又不想加入代码, 问在 nginx 层面能不能实现.   
### 解决方案  
开始就想强大到一塌糊涂的 nginx 应该可以的, 不行搞一个 module 也就 ok 了.  简单找了一下, 发现果然可以.  

1. 使用 htpasswd 工具生成对应的用户名密码, 保存在 .htpasswd 文件中
2. 修改 nginx 配置, 在需要添加验证的地方引入以下配置  

	 	auth_basic "Restricted";
	    auth_basic_user_file /etc/nginx/.htpasswd;
	  
3. reload nginx 配置即可


## nginx 根据 http 方法进行分流 
### 背景  
开发项目中使用了 Restful 原则, 然后为保证数据的一致性同时为了提高查询的负载能力, 采用了 1Master NSlave 的模型,  因此需要将 http GET 方法流量均衡到多个 Slave 上.  

         location /v2 {
            if ($request_method = GET ) {
                proxy_pass http://tree-slaves;
                break;
            }
            proxy_pass http://tree-master;
            break;
        }
