title: nginx二级域名匹配根目录下的子文件夹
date: 2014-12-12 21:26:33
tags: nginx
---

使用泛域名映射子目录

配置文件如下

```
server
{

        listen 80;
        server_name *.test.com;

        if ( $host ~* (.*)\.test\.com) {
             set $domain $1;
        }

        root /usr/share/nginx/$domain;
        index index.html index.php;

        include php.conf;
}
```

配置域名

```
#vim /etc/hosts
127.0.0.1   tom.test.com
127.0.0.1   jake.test.com
```

创建测试文件
在相应目录内新建测试文件即可看到效果
