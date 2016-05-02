title: (转)CentOS普通用户增加sudo权限的简单配置
date: 2014-12-24 23:13:27
tags: linux
---

查看sudo是否安装:
```shell
#rpm -qa|grep sudo
```
修改/etc/sudoers文件，修改命令必须为visudo才行
```shell
#visudo -f /etc/sudoers
```

在root ALL=(ALL) ALL 之后增加
```shell
tom ALL=(ALL) ALL
Defaults:tom timestamp_timeout=-1,runaspw
```
```
增加普通账户tom的sudo权限
timestamp_timeout=-1 只需验证一次密码，以后系统自动记忆
runaspw  需要root密码，如果不加默认是要输入普通账户的密码
```

修改普通用户的.bash_profile文件
vi /home/tom/.bash_profile
在PATH变量中增加
/sbin:/usr/sbin:/usr/local/sbin:/usr/kerberos/sbin

```shell
$ cat .bash_profile
# .bash_profile
# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi
# User specific environment and startup programs
PATH=$PATH:$HOME/bin:/sbin:/usr/sbin:/usr/local/sbin:/usr/kerberos/sbin
```
原文地址：[CentOS普通用户增加sudo权限的简单配置][1]


  [1]: http://www.chinasb.org/archives/2011/07/2919.shtml