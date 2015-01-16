+++
date = "2014-12-12T12:17:54+08:00"
draft = false
title = "GolangDraft"
descriptions = "Golang 相关知识"

+++

## Golang 相关 tools 安装
### 背景 
在学习 golang 过程中, 看到 Rob Pike 大神的 [slide](https://talks.golang.org/2012/waza.slide#19) , 感觉炫酷.  安装过程如下:  

	go get golang.org/x/tools/cmd/present
	
**注意** 如果期望手工 build, 进入目录:  `go/src/golang.org/x/tools/cmd/present`  不要搞错.


## 输出前缀为0的数字
### 背景  
在运维过程中为了管理大量的机器, 通常会对机器进行编号, 比如 `web001.$idc` `web0002.$idc`,
在 golang 中的解决方案:  

    fmt.Printf("%03d", seq)
    
说明:   
%3d 表示输出数字的宽度至少为 3  
0   填充前导的0而非空格；对于数字，这会将填充移到正负号之后

<!--more-->

## Golang 字符串 split
### 背景
通常我们需要对一个字符串进行 split, 通常反应是利用` strings.Split()` 来进行切分

    var  s = "a b  c"
    fmt.Println("%#v", strings.Split(s," "))
    
    output: 
    []string{"a", "b", "", "c"}
    
可以看出来` strings.Split()` 只是简单的利用 SEP 来进行切分.  
### 解决方案
目前想到的解决方案是利用` regexp` 来进行切分  

    func main() {
        var s = "a b  c"
        fmt.Printf("%#v\n", regexp.MustCompile(" +").Split(s, -1))
    }
## Golang 时间处理
时间主要分为 date + time(gaoling 中叫 clock)  
### 获取当前时间信息

    now  := time.Now()
    
### 获取 epoch time (timestamp)

    now.Unix()
 
### 获取月份

    now.Month()
    curMonth := now.Month()
    if curMonth == time.June {
        ...
     }
### 获取日期部分  

    year, month, day := now.Date()  
    
### 获取时间部分  

    hour, min, sec  := now.Clock()
    
### 获取当前时区

    fun (Time)Zone() (name string, offset int)
    offset: 表示和 UTC 的时差(单位 s), 比如东八区(28800 8h)
    
    now := time.Now()
    zone, offset := now.Zone()
    
    fmt.Println("time zone:%s", zone)
    time zone:CST
### 转换为 UTC 时间

    func (t Time) UTC() Time
    UTC returns t with the location set to UTC
    
### 相对时间  
时间相关还有一个问题是是**时间间隔**, 比如我们需要 profile 一个函数的耗时. 

    stime := time.Now()
    expensiveCall()
    time := time.Now()
    
    var duration Duration = etime.Sub(stime)
**其实本质上Duration 就是一个表示两个时刻之间的纳秒数( int64)**   
其中 duration.Seconds(), duration.Minutes() 表示duration 转换为对应的秒数, 分钟, 注意 这里并不是只是获取 duration 的秒数或者分钟部分, duration 只是表示间隔, 并没有 clock 中 minutes 和 second. 

    stime := time.Now()
    time.Sleep(3 * time.Second)
    etime := time.Now()
    
    duration :=  etime.Sub(stime)
    fmt.Printf("elapsed: %s minutes", duration.Minutes())
    
#### 科普 CST 含义  
CST: 中部标准时间 (Central Standard Time)
同时表示下面4个时区  
CST Central Standard Time(USA) UT-6:00
CST Central Standard Time(Australia) UT 9:30
**CST China Standard Time UT 8:00**
CST Cuba Standard Time UT-4:0  
我们常遇到的应该就是 China Standard Time.
#### GMT 和 UTC 的关系
1. UTC (Universal Time Coordinated), 以子午初线(经度0)上的评价太阳时为依据, 也就是英国伦敦的平均太阳时  
2. GMT (Greenwitch Mean Time) 格林威治平均时间, 由于地球绕太阳的运行的轨道是椭圆, 导致 UTC 表示的时间, 不是很准确, 因此提出了 GMT 时间, 每年或者2年对 UTC 增加一个闰秒, 来完成修正.  一般上我们可以认为 GMT 和 UTC 是一样的. 

### Golang 设置时区
#### 背景
最近在给运营同学做一个定时和实时配送策略文件的时候, 遇到一个问题: 对于运营同学自然不可能要求他们会 crontab, 因此和运营同学约定设置配送格式为  

    File  Time
    a.txt  08:30:00  #早上8点定时配送
后台程序的设计思路其实很简单, 就是通过**对比当前时间和配置时间** 如果当前时间大于配置时间的话, 就采用最近的时间进行配置. 

发现问题是: 

    func main() {
        layout := "2006-01-02 15:04:05"
        settingTime := "2015-01-11 15:04:05"
        t, _ := time.Parse(layout, settingTime)
        fmt.Println(t)
    }
返回时间如下, 也就是说默认返回的是 UTC 时间

    2015-01-11 15:04:05 +0000 UTC

而使用 `time.Now()` 得到的是 `2015-01-11 22:37:03.437033928 +0800 CST` CST 时间.  
UTC 和 CST 的时间相差8小时, 因此为了方便运营同学配置, 需要将加载的配置时间设置为 CST.

#### 解决方案  
    func StrToTime(timeStr string) (time.Time, error) {
        loc, _ := time.LoadLocation("Asia/Chongqing")
        layout := "2006-01-02 15:04:05"
        fmt.Println("time star parse is : ", timeStr)
        return time.ParseInLocation(layout, timeStr, loc)
    }

方案2:  
  
    func StrToTime(timeStr string)(time.Time, error) {
        loc := time.FixedZone("CST", 3600 * 8) 
        layout := "2006-01-02 15:04:05"
        return time.ParseInLocation(layout, timeStr, loc)
    }

### Golang 获取文件 mtime, ctime, atime


    func statTimes(name string)(atime, ctime, time time.Time, err error) {
        fi, err := os.Stat(name)
        if err != nil {
            return 
        }
        mtime = fi.ModTime()
        stat := fi.Sys().(*syscall.Stat_t)
        atime = time.Unix(int64(stat.Atim.Sec), int64(stat.Atim.Nsec))
        ctime = time.Unix(int64(stat.Ctim.Sec), int64(stat.Ctim.Nsec))
    }

### Golang 多文件上传  
最近计划做一个 Babel项目, 该项目解决的问题是: 办公环境和 IDC 之间由于安全问题, 封锁了网络链路, 只允许部分端口直连, 通过 rz/sz 传输代码也极其不方便, 因此系统提供一个上传和下载功能的简单工具.   
在上传的时候希望能够支持多个文件上传, 调研了一下发现 html 里边对于多个文件上传其实有两种形式的.  

1: 单个上传通道, 支持多个文件上传.  
HTML 代码:  

    <form id="uploadForm"  enctype="multipart/form-data" action="/upload" method="POST">
    #注意 enctype 必须为 multipart/form-data
	    <p>Golang Upload</p> <br/>
	    <input type="file" id="file1" name="uploadFile" multiple /> #注意这里的 multiple 表明支持多个文件上传.	    
	    <br/>
	    <input type="submit" value="Upload">
    </form>
    
Server 代码: 


    func UploadServer(w http.ResponseWriter, r *http.Request) {
        r.ParseMultiPartForm(32 << 20) // 32M, 在使用 MultiPartForm 之前必须要先调用ParseMultiPartForm  
        if r.MultiPartForm != nil && r.MultiPartForm.File != nil {
            fhs := r.MultiPartForm.File["uploadFile"]
            
            for index, header := range fhs {
                filename := header.Filename
                file, _ := header.Open()
                defer file.Close()
                
                //save file to disk
                f, err := os.Create(filename)
                defer f.Close()
                
                io.Copy(f, file)
                
                fstat, _ := f.Stat()
                fmt.Fprintf(w, "No: %d, Size:%d KB Name:%s", index, fstat.Size()/1024, filename)
            }
        }
    }
    
2. 多个上传通道, 本质上其实和方案一样, 差异在 HTML.
HTML 代码: 

            <form action="/v1/dispatch" method="post" enctype="multipart/form-data">
            <input type="file" name = "uploadFiles[]"><br/>
            <input type="file" name = "uploadFiles[]"><br/>
            <input type="file" name = "uploadFiles[]"><br/>
            <input type="submit" class="btn" value="上传文件" />
        </form>
Server 端代码: 

        this.Ctx.Request.ParseMultipartForm( 10 * (8 << 20))
        fhs := this.Ctx.Request.MultipartForm.File["uploadFiles[]"]
        var forgetMeta bool = true
        var realtime bool = false

        for index, f := range fhs {
            file, err := f.Open()
            defer file.Close()

            if err != nil {
                beego.Warn("open file failed.")
            }else{
            beego.Trace("ok, I get ",index,"th file:", f.Filename)
            }
其它知识点: 

         options := r.MultipartForm.Value["options[]"] #获取HTML数组内容.
 如果form里面变量都是唯一的，直接用parseFormValue，和parseFile就可以，因为返回的都是单个变量而不是一个数组了，省的另外操作数组。
 
 ## Golang post https 服务
 ### 背景 
 公司有一个服务是利用 https 提供服务的, 在改域名下提供一个 http 接口.  利用 curl POST  
 
     curl  -d 'name=foo' https://foo.bar.com 
     提示错误: Get https://golang.org/: certificate is valid for *.appspot.com, *.*.appspot.com, appspot.com, not golang.org
     
     curl -d 'name=foo' -k  https://foo.bar.com 才可以  
     
     -k, --insecure
              (SSL)  This option explicitly allows curl to perform "insecure" SSL connections and transfers. All SSL connections are attempted to be made secure
              by using the CA certificate bundle installed by default. This makes all connections considered "insecure" fail unless -k, --insecure is used.

              See this online resource for further details: http://curl.haxx.se/docs/sslcerts.html  
             
### golang 代码

    package main
    
    import (
        "fmt"
        "net/http"
        "crypto/tls"
    )

    func main() {
        transport := &http.Transport{
            TLSClientConfig : &tls.Config{InsecureSkipVerify: true},
        }
        
        client := &http.Client{Transport: transport}
        
        _, err := client.Get("https://golang.org/")
    }
    
    
[refer](http://stackoverflow.com/questions/12122159/golang-how-to-do-a-https-request-with-bad-certificate)


## Go get 工具  
今天在使用第三方服务的时候, 在使用一个项目的时候, 需要获取对应的 package   

    go  get  ./...
    
`go get` 命令了解, 可是有`threee dot` 没有看到对应的文档说明, 只能阅读对应的代码. 发现原来是这样的.  

    // downloadPaths prepares the list of paths to pass to download.
    // It expands ... patterns that can be expanded.  If there is no match
    // for a particular pattern, downloadPaths leaves it in the result list,
    // in the hope that we can figure out the repository from the
    // initial ...-free prefix.
    func downloadPaths(args []string) []string {
	args = importPathsNoDotExpansion(args)
	var out []string
	for _, a := range args {
		if strings.Contains(a, "...") {
			var expand []string
			// Use matchPackagesInFS to avoid printing
			// warnings.  They will be printed by the
			// eventual call to importPaths instead.
			if build.IsLocalImport(a) {
				expand = matchPackagesInFS(a)
			} else {
				expand = matchPackages(a)
			}
			if len(expand) > 0 {
				out = append(out, expand...)
				continue
			}
		}
		out = append(out, a)
	}
	return out
    }
go get 在获取到对应的下载的 path 的时候, 对 `...` 进行特殊处理, `...` 作用就是**对该目录下的所有文件进行 go get**
