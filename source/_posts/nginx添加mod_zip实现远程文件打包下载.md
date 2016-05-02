layout: nginx
title: 实现远程文件打包下载
date: 2015-01-10 17:00:34
tags: [nignx, mod_zip, php]
---


- 传统实现方法 
    下载远程文件到本地服务器--->压缩-->下载-->删除。
    这样实现 消耗内存、cpu、大量磁盘空间、速度太慢(取决于网络).

- 使用第三方mod_zip 实现打包下载。
    内存消耗极低。
    通过输出``X-Archive-Files``和文件列表，自动将远程文件打包下载避免了临时文件的创建和删除。

# 安装过程

## 下载mod_zip 源文件
```
#cd /usr/local/src
#git clone https://github.com/evanmiller/mod_zip.git
```

## tengine安装过程
使用[dso-tool编译动态模块][1]
```
/usr/local/tengine/bin/dso-tool \
--add-module=/usr/local/src/mod_zip/ \
--dst=/usr/local/tengine/modules/
```

报错1

![此处输入图片的描述][2]
```
#vi +331 /usr/local/src/mod_zip/ngx_http_zip_file.c
```
修改为
![此处输入图片的描述][3]
保存
重新编译
仍然报错 ```ngx_str_t ngx_http_zip_header_charset_name defined but not use```
```
#vi +11 /usr/local/src/mod_zip/ngx_http_zip_file.c
```
注释掉
```
//static ngx_str_t ngx_http_zip_header_charset_name = ngx_string("upstream_http_x_archive_charset");
```

重新编译通过

查看动态模块是否生成
```
#ls /usr/local/tengine/modules/
```

- 修改nginx配置文件测试

```
#vi nginx.conf 

dso {
	...
	load ngx_http_zip_module.so;
	...
}
```


- 编写测试代码

```
vi zip.php

<?php
    header('X-Accel-Chareset: utf-8');
    header('Content-Type: application/octet-stream');
    header('Content-Disposition: attachment; filename=test.zip');
    header('X-Archive-Files: zip');
		
    $crc32 = "-";
    //格式 crc32(可以用-忽略)、文件大小、文件uri、文件名
		
    printf("%s %d %s %s\n", $crc32, 688746, '/lms/M00/00/11/qMB6HVOaakWARZXJAAqCatIt43Y37.pptx', '1.pptx');
    printf("%s %d %s %s\n", $crc32, 5236, '/default.conf', 'd.conf');
?>
```

- tips

    -  文件大小一定要正确 否则压缩打不开 
    - 文件uri (非url 远程文件需要配置nginx跳转).
        例子中的lms(fastdfs)远程文件 需要在nginx内做配置跳转到正确的地址。
    - 如果不需要断点续传crc32参数可以使用``-``忽略。
    ```
    #vi nginx.conf
    server {
        ... 	
        location /lms {
            proxy_pass http://fastdf_server; 	
        }
        ... 
    }
    ```

## nginx 安装

- 查看旧的编译参数
```
#/usr/local/nginx/sbin/nginx -V
```

- 编译

```
#/usr/local/nginx/configure  \
--prefix=/usr/local/nginx  \
--user=www --group=www  \
--add-module=/usr/local/src/mod_zip

#make
```
报错级处理参考tengine内的处理方法.

#升级nginx
```
#cp objs/nginx /usr/local/nginx/sbin

#/usr/local/nginx/sbin/nginx -t
#/usr/local/nginx/sbin/nginx 
```

参考文档
 [NgxZip][4]
 [利用Nginx第三方模块，实现附件打包下载][5]
 [mod_zip][6]

  [1]: http://tengine.taobao.org/document_cn/dso_cn.html
  [2]: http://i2.tietuku.com/848ad5c3e104817f.png
  [3]: http://i2.tietuku.com/e7c8982e03665f39.png
  [4]: http://wiki.nginx.org/NgxZip
  [5]: http://www.pchou.info/open-source/2014/07/28/nginx-mod-zip.html
  [6]: http://www.v5b7.com/webserver/nginx/nginx_mod_zip.html