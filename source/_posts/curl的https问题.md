title: curl的https问题
date: 2014-11-28 19:10:27
tags: [php, curl]
---

php 使用curl访问https网站时遇到的问题。
代码如下
```
	<?php
		$ch = curl_init();
		curl_setopt($ch, CURLOPT_URL, $url);
		curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
		curl_setopt($ch, CURLOPT_TIMEOUT, 1);
		$return = curl_exec($ch);		
		curl_close($ch);

		var_dump($return);
	?>
```
返回false

修改代码 在curl_exec  后加入
```
    if(curl_errno($ch)){
        	var_dump(curl_error($ch));
    }
```
可以看到 证书获取不到的提示信息
> SSL certificate problem: unable to get local issuer certificate

 

 - 简单解决 加入一下选项 禁用证书检查
```
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
```
    
 - 添加证书解决
    1. 获取证书

    	1. 在浏览器(firefox)打开请求的接口地址，
    	1. 选择菜单（工具--查看页面信息）
    	1. 安全--查看证书--详细信息--选择顶级证书机构--导出(X.509(pem))


    2. 修改代码 添加如下选项
```
	curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, true);
	curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 2);
	curl_setopt($ch, CURLOPT_CAINFO, __DIR__ . '/BuiltinObjectToken-EquifaxSecureCA.crt');
```
SSL CA cert (path? access rights?) 问题 参考[stackoverflow][1]直接重启php-fpm


  [1]: http://stackoverflow.com/questions/7179216/php-problem-with-the-ssl-ca-cert-path-access-rights