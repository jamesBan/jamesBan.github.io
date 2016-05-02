title: mongodb注入攻击
date: 2014-12-01 20:54:47
tags: [mongodb, php]
---

一直以为注入只存在关系型数据库中，看了这篇[文章][1]才知道mongodb也会中招.
翻看php手册发现早就发现有这个[问题][2]，以前看手册一直没注意.

数组绑定时的注入

```php
<?php
$mongo = new mongoclient();
//选择数据库
$db = $mongo->myinfo; 
 //选择集合
$coll = $db->test;

$username = $_GET['username'];
$password = $_GET['password'];
$data = array(
        'username'=>$username,
        'password'=>$password
        );
$data = $coll->find($data);
$count = $data->count();
if ($count>0) {
    foreach ($data as $user) {
        echo 'username:'.$user['username']."</br>";
        echo 'password:'.$user['password']."</br>";
    }
}
else{
    echo '未找到';
}
?>
```

如果传入url为
```
http://127.0.0.1/2.php?username[$ne]=test&password[$ne]=test
```

mongodb最终执行：
```
db.test.find({username:{'$ne':'test'},password:{'$ne':'test'}});
```
php会把test集合内的所有数据全部便利出来。

防止注入的方法也很简单, 注意参数的检验.

```php
<?php
...
$username = $_GET['username'];
$password = $_GET['password'];

$username = is_string($username) ? $username : '';
$password = is_string($password) ? $password : '';
...
?>
```

接受参数拼接直接以命令行执行的注入可以参考[php手册][3] 使用MongoCode 进行转椅后再使用.


  [1]: http://drops.wooyun.org/tips/3939
  [2]: http://php.net/manual/zh/mongo.security.php
  [3]: http://php.net/manual/zh/mongo.security.php