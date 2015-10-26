+++
date = "2015-10-26T18:37:38+08:00"
draft = false
title = "RpmDraft"

+++

##如何定制puppet rpm
###背景
最近在梳理初始化脚本,其中发现一个问题. 我们需要安装 puppet 可以到机器上,  其中 rpm 是从官方网站获取到.   
然后初始化脚本会做一下2件事情.  

1:  安装 puppet agent 的 rpm.  类似` rpm -ivh  puppet-version.rpm`  
2:  更新`/etc/puppet/puppet.conf` 

其实第二步是希望更新`puppet.conf`, 问题是这个文件在 puppet 的 rpm 里边,  我们是否有办法直接将` puppet.conf` 直接更新 `puppet-version.rpm` 内,  这样我们可以简化安装 puppet 步骤, 直接  `yum install -y puppet` 就可以搞定, 同时提高初始化效率和正确性, 毕竟多次 copy 文件交互需要消耗时间同时可能由于网络问题导致安装异常. 

###尝试 
第一次尝试采用这样的方法:   
1: 将  puppet-version.rpm 解压开`rpm2cpio  puppet-version.rpm | clio -idmv `   
2: 更新配置文件, 将 `puppet.conf` 文件更新  
3: 将 `puppet-version` 目录利用 `fpm` 工具重新打包. 

####问题
puppet 的依赖很多, 利用 fpm 制定依赖太麻烦, 而且我们不确定 puppet 这个包是否有 hook 之类操作, 所以上面这种方案仅适合我们对打包方法十分了解, 依赖简单的方案.   

```
rpm -qpR puppet-3.8.3-1.tl1.noarch.rpm
/bin/bash
/bin/sh
/bin/sh
/bin/sh
/bin/sh
/bin/sh
/usr/bin/env
/usr/bin/ruby
chkconfig
config(puppet) = 3.8.3-1.tl1
facter >= 1:1.7.0
hiera >= 1.0.0
initscripts
libselinux-utils
rpmlib(CompressedFileNames) <= 3.0.4-1
rpmlib(FileDigests) <= 4.6.0-1
rpmlib(PartialHardlinkSets) <= 4.0.4-1
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
rpmlib(VersionedDependencies) <= 3.0.3-1
ruby >= 1.8
ruby >= 1.8.7
ruby(selinux)
ruby-augeas
ruby-shadow
rubygem-json
shadow-utils
rpmlib(PayloadIsXz) <= 5.2-1   
```

##最终解决方案
考虑到 puppet 是成熟的的开源软件, 打包应该符合开源的规范, 学习了解之后, 要打包出 **RPM**, 需要 **SRPM** 包.   
从@坤神了解到下载 puppet 的地址是 `yum.puppetlabs.com/el/6.2/products/$basearch`, 那么猜测可以找到 SRPM. 果然在 `http://yum.puppetlabs.com/el/6.2/products/SRPMS/` 找到对应的 puppet 的** SRPM** 

####具体操作
1. 下载对应的 SRPM ` wget http://yum.puppetlabs.com/el/6.2/products/SRPMS/puppet-3.8.3-1.el6.src.rpm`  
2. 修改SRPM 安装的prefix 路径, 避免影响系统.  创建`~/.rpmmacros` 文件, 新增如下内容  

```
%_topdir /root/puppet/agent/src/rpm
```

3. 创建编译需要的对应目录  

```
cd /root/puppet/agent/src/rpm
mkdir BUILD RPMS SOURCES SPECS SRPMS
mkdir RPMS/{i386,i486,i586,i686,noarch,athlon}
```
其中   
BUILD 目录用在编译的适合, SOURCES 解压存放的地址, 是一个临时的目录  
SOURCES 用来存放source tarball, patches 

4. SRPM 文件 

```
rpm -ivh  puppet-3.8.3-1.el6.src.rpm
```
由于我们 已经设定`%_topdir`, 因此解压之后, 会**自动**将源代码放到 SOURCES 下面,  将 rpm 的 specfile 放到 SPECS. 得到结构如下:  

```
[ ~/puppet/agent/src/rpm]# tree -L 2
.
|-- BUILD
|   `-- puppet-3.8.3
|-- BUILDROOT
|-- RPMS
|   |-- athlon
|   |-- i386
|   |-- i486
|   |-- i586
|   |-- i686
|   `-- noarch
|-- SOURCES
|   `-- puppet-3.8.3.tar.gz
|-- SPECS
|   `-- puppet.spec
`-- SRPMS
    `-- puppet-3.8.3-1.tl1.src.rpm
```

5. 替换对应文件
由于rpmbuild 在打包的时候会将`SOURCES` 下面的 `puppet-3.8.3.tar.gz` 进行打包,  因此我们只需要将`puppet-3.8.3.tar.gz` 替换新文件, 然后重新打包即可.  

6. 重新编译生成 RPM   

```
pmbuild -ba SPECS/package.spec
```
执行完成之后, 在 `RPMS/noarch` 下面就可以得到我们需要的 package 

[How to patch and rebuild an RPM package](http://bradthemad.org/tech/notes/patching_rpms.php)



