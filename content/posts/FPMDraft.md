+++
date = "2015-01-21T00:08:20+08:00"
draft = true
title = "FPMDraft"

+++

## FPM

### FPM 入门

    fpm -s <source type> -t <target type> [options]
    source types:  
        "dir"  可以是文件或者是目录

### 常见问题和解决思路
1.  Process failed: rpmbuild failed (exit code 1). 
    解决思路:  fpm 运行增加 `--verbose` 选项, 查看日志输出
    
2. 如何把我的一个程序, 比如 jq 打包为 RPM 格式文件, 期望安装之后安装 到 `/bin` 目录下 

    
        fpm -s dir -t rpm -n jq -v 1.4.0 --prefix=/bin/  jq
        
        
        http://golang-basic.blogspot.jp/2014/10/dynamic-programming-problem-maximum.html