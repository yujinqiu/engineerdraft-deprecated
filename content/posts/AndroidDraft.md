+++
date = "2015-01-03T13:20:13+08:00"
draft = false
title = "AndroidDraft"

+++

## 创建 Android Virtual Device(AVD)

    android list  targets 查看已经下载好的 sdks
    
## 查看
    
    adb devices 能够查看配置好的 android 物理设备
    ➜  bin git:(master) ✗ adb devices
    List of devices attached
    MX21CA1ALJZM4D2227	device
    
<!--more-->
   
## 破解 Android 程序通常方法是将apk   
文件利用 ApkTool反编译, 生成 Smali 格式的反汇编代码, 然后阅读 Smali 文件的代码来理解程序的运行机制, 找到突破口进行修改, 最后利用 ApkTool 重新编译生成apk 文件, 并签名, 最后进行测试

## apktool 安装  
### 注意点  
mac os x 利用 brew 安装之后,建议确认 apktool 的版本,使用2.0.0 以上版本, 1.5.2 版本个人测试存在问题.  
### mac os x  安装
    Mac OS X:
        Download Mac wrapper script (Right click, Save Link As apktool)
        Download apktool-2 (find newest here)
        Rename downloaded jar to apktool.jar
        Move both files (apktool.jar & apktool) to /usr/local/bin (root needed)
        Make sure both files are executable (chmod +x)
        Try running apktool via cli 

Note - Wrapper scripts are not needed, but helpful so you don't have to type java -jar apktool.jar over and over. 

### 反编译 apk 文件

    apktool d  <file_apk> -o  <output_dir>
    
#### 修改反编译文件之后重新打包
    
    apktool  b <output_dir> 
    
    #默认会生成在 output_dir/dist/ 下面
    
#### 重新签名 apk 
1. 在签名之前需要有 keystore   
创建 keystore 

        keytool -genkey -keystore <name.keystore> -alias <alias_name> -keyalg RSA -keysize 2048 -validity 1000  
        keystore:是用来存储keys 和 certificates
        其中 alias_name 用来标记 key 的名称
        name.keystore 是 keystore 的名称
    
对 apk 进行签名  
    
    jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore my_application.apk alias_name
    
    alias_name 对应上面的创建 keystore 的名字
    签名完成之后,会把签名信息直接写入 apk 文件中
    
验证 apk 签名

    jarsigner -verify -verbose -certs <name.apk>
  
对齐 apk

    zipalign -v 4 your_project_name-unaligned.apk your_project_name.apk
    
    zipalign 用来是apk 内存字节对齐, 减低内存消耗
    
 注意：

1. CN（Common Name – 名字与姓氏）：其实这个“名字与姓氏”应该是域名，比如说localhost或是blog.devep.net之类的。输成了姓名，和真正运行的时候域名不符，会出问题。浏览器访问时，弹出一个对话框，提示“安全证书上的名称无效，或者与站点名称不匹配”，用户选择继续还是可以浏览网页。但是用http client写程序访问的时候，会抛出类似于“javax.servlet.ServletException: HTTPS hostname wrong: should be ”的异常。

2. 在用keytool生成数字证书时必须保证：-keystore androidapp.keystore -alias androidapp.keystore 两者名称必须相同。否则下一步签名时会出现错误：jarsigner： 找不到 androidapp.keystore 的证书链。androidapp.keystore 必须引用包含专用密钥和相应的公共密钥证书链的有效密钥库密钥条目。


#### adb 重新安装 apk

    adb install  -r <name.apk> 
    -r : replace existing application
    -s : install application on sdcard
    
#### adb 重新安装 apk 报错和解决方案

INSTALL_PARSE_FAILED_INCONSISTENT_CERTIFICATES: 
是由于破解之后的 app 的签名和编译的原apk 签名不一致导致  
**解决方法:** 删除设备或者模拟器中的 apk, 然后 再 `adb install  -r <name.apk> ` 即可



#### android studio 常用技巧
1. command + shift + a : 输入对应的命令, 类似 alfred
2. 要修改一个控件的文件, go to definition( command + b)
3. 
