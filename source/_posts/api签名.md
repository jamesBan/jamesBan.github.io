title: api签名
date: 2014-11-27 23:32:40
tags: [api, php]
---

为了提高传输过程参数的防篡改性，必须使用签名参数sig.
本文只是一个签名的简单实现 验证参数完整性 请求超时.


```php
<?php
class SomeClass
{
    CONST API_PUBLIC_KEY = "asdadasdaxxxx";
    CONST URL = 'http://example.com/api/';
   

    public function signature(array $param, 
             $timeOut = 60)
    {
        $param['expire'] = time() + $timeOut;
        $param['sig'] = hash_hmac('sha256', 
                join('', $param), self::API_PUBLIC_KEY);
        $this->param = $param;

        return $this;
    }

    public function getRequestUrl()
    {
        return self::URL . '?' . http_build_query($this->param);
    }

    public function validateSignature($param, $sig)
    {
        return hash_hmac('sha256', join('', $param), self::API_PUBLIC_KEY)
             == $sig;
    }
}

$obj = new SomeClass();

$param = array('useranme' => 'zhangsan');

//http://example.com/api/?useranme=zhangsan&expire=1408628093&sig=27b11880e19691b7e110cc8665c55160fc2b321b74c33b4877fac2da0545640e
echo $obj->signature($param, 300)->getRequestUrl();


$sig = isset($_GET['sig']) ? $_GET['sig'] : '';
$expire = isset($_GET['expire']) ? $_GET['expire'] : '';
if(!$obj->validateSignature($param, $sig)){
    echo "签名验证错误";
    exit;
}

if($expire < time()) {
    echo "请求超时";
    exit;
}

echo "processing";
?>
```