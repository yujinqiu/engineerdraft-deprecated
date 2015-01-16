+++
date = "2015-01-11T11:31:48+08:00"
draft = false
title = "JqueryDraft"
categories = [
    "jquery"
]

+++

## Jsonp
Unexpected token colon JSON after jQuery.ajax#get
### 背景  
FE 同学由于联调环境原因, 通常希望在本地调用服务端的接口, 为了解决跨域问题, 通常会使用`jsonp`, 然后今天突然反馈说和其他 RD 联调的时候发现console log 报错:  

    Unexpected token :
    
#### 追查
开始怀疑是返回的 json 格式有问题, 利用 json 工具校验之后, 排除了这种可能性. 后面突然想到问题可能出现在 server 端,  联系相关的 RD, 看了代码之后, 发现没有提供对应的 jsonp 接口.  don't argue, just show me the code

#### 原因
To support [JSONP request](http://json-p.org/), the server will have to include the `P`, or `Padding`, in the response.  
Server return   

    {"Name":"Tome", "Description": "Hello it's me!"}

but it should be  

    jQuery111108398571682628244_1403193212453({"Name":"Tom","Description":"Hello it's me!"})
    
The syntax err, `"Unexpected token :"`, is because **JSONP is parsed as JavaScript**, where  `{...}` also represents [blocks](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/block).  
blocks syntax should be:  

    {
        statement_1;
        statement_2;
        ...
        statement_n;
    }
    
so we get `Unexpected token :` error  

<!--more-->

#### 解决方案
在 server 端增加 JSONP 返回接口. 

    if (callback) {
        res.setHeader('Content-Type', 'text/javascript'); #注意这里是 javascript
        res.end(callback + '(' + data + ')');
    } else {
        res.setHeader('Content-Type', 'application/json');
        res.end(data);
    }
    
