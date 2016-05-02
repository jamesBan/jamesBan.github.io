title: pack函数的简单使用
date: 2014-12-12 21:24:42
tags: php
---

pack函数主要作用是把数据压缩为二进制字符串。qq纯净ip数据库数据存储就是使用pack进行封装。本文为pack的简单应用

```php
<?php

class Db {
	private $file = null;
	
	private $struct = array(
		'name' => 'a8',
		'age' => 'S',
		'email' => 'a30'
	);
	
	private $length = 40;
	
	private function init()
	{
		$file = fopen('./test.db', 'ab+');
		$this->file = $file;
	}
	
	public function __construct()
	{
		if($this->file === null) {
			$this->init();
		}
	}
	
	public function add($data) 
	{
		$binary = '';
		foreach($data as $k => $row) {
			$struct = $this->struct[$k];
			$binary .= pack($struct, $row);
		}
		
		fwrite($this->file, $binary);
	}
	
	
	public function read($id = 1) 
	{
		rewind($this->file);
		$data = array();
		
		if(!fseek($this->file, $this->length * ($id -1))) {
			$data['name'] = unpack('a8', fread($this->file, 8))[1];
			$data['age'] = unpack('S', fread($this->file, 2))[1];
			$data['email'] = unpack('a30', fread($this->file, 30))[1];
		}
		
		return $data;
	}
	
	public function __destruct()
	{
		fclose($this->file);
	}
}
```

存储数据

```php
$data = array(
	'name' => 'lisi' . time(),
	'age' => 100,
	'email' => 'jjjjj@qq.com'
);
$db = new Db();
$db->add($data);
```

读取数据

```php
$data = $db->read(1);
print_r($data);

/*
Array
(
    [name] => lisi1418
    [age] => 100
    [email] => jjjjj@qq.com
)
*/
```
