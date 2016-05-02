title: 不使用strrev和数组函数实现字符串反转
date: 2014-12-09 19:04:19
tags: php
---

反转一个字符串很容易使用[strrev][1]函数即可,假如没有strrev函数也没有数组函数怎么显示字符串反转。

* 背景知识
```
string 中的字符可以通过一个从 0 开始的下标，用类似 array 结构中的方括号包含对应的数字来访问和修改，比如 $str[42]。
可以把 string 当成字符组成的 array。函数 substr() 和 substr_replace() 可用于操作多于一个字符的情况。 -- php手册
```
* 代码
```php
<?php
function reverseString($string) 
{
	$length = strlen($string);
	$reverseString = '';
	
	while($length--) {
		$reverseString .= $string[$length];
	}
	
	return $reverseString;
}


$str = "Hello world";
echo reverseString($str); //dlrow olleH
```php

使用str_split 函数也可以。
```php
$array = str_split($str);
print_r($array);

/**

Array
(
    [0] => H
    [1] => e
    [2] => l
    [3] => l
    [4] => o
    [5] =>  
    [6] => w
    [7] => o
    [8] => r
    [9] => l
    [10] => d
)
*/
```

参考 [字符串反转，神奇的算法-读《编程珠玑（第二版）》][2] 有更高效的写法。
```php
<?php
function ReverseString(&$s, $from, $to)
{
    while ($from < $to)
    {
        $t = $s[$from];
        $s[$from++] = $s[$to];
        $s[$to--] = $t;
    }
}

function LeftRotateString($s, $n, $m)
{
    $m %= $n;               //若要左移动大于n位，那么和%n 是等价的  
    ReverseString($s, 0, $m - 1); //反转[0..m - 1]，套用到上面举的例子中，就是X->X^T，即 abc->cba

    ReverseString($s, $m, $n - 1); //反转[m..n - 1]，例如Y->Y^T，即 def->fed

    ReverseString($s, 0, $n - 1); //反转[0..n - 1]，即如整个反转，(X^TY^T)^T=YX，即 cbafed->defabc。

    return $s;
    
}

echo LeftRotateString('abcdef', 6, 3); //defabc
```

  [1]: http://php.net/manual/en/function.strrev.php
  [2]: http://haolloyin.blog.51cto.com/1177454/560597