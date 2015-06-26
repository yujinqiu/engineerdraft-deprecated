+++
date = "2015-06-22T23:18:49+08:00"
draft = false
title = "Rsyslog Draft"

+++

##Rsyslog
### Rsyslog 和 Syslog 关系  
In Red Hat Enterprise Linux 3/4/5, the default system log tool is syslogd which is provided by package sysklogd, but since Red Hat Enterprise Linux 6, the rsyslogd became the default. [refer](https://access.redhat.com/solutions/36328)  
简单的说可以理解为 Rsyslog 是 Syslog 的超集, 从RHEL 6 开始系统默认替换了.   

#### Rsyslog 查看版本  

    rsyslogd -version

