title: DateTime类的常见用法
date: 2014-12-04 19:36:11
tags: [date, php]
---


- 初始化

```php
//默认为当前时间
$tz = new DateTimezone('Asia/Shanghai');
echo (new DateTime('now', $tz))->format('Y-m-d H:i:s'); //2014-12-04 12:04:55
$d = new DateTime('+1 day', $tz); //2014-12-05 12:04:55
```
- 设置时区
```php
$d = new DateTime(null, new DateTimezone('Asia/Shanghai'));
$d->setTimezone('Asia/Shanghai');
```

- 设置/获取时间戳
```php
echo $d->getTimestamp();  //1417664087
echo $d->setTimestamp(1417664087)->format('Y-m-d H:i:s');  //2014-12-04 11:34:47
```

- 修改时间
```php
//明天当前时间 (使用关键字day、week、month year 进行修改)
使用modify
echo $d->modify('+1 day')->setTime(0, 0, 0)->format('Y-m-d H:i:s');
```

或者使用 sub、add
```php
echo $d->sub(new DateInterval('P1D'))->format('Y-m-d H:i:s');
echo $d->add(new DateInterval('P1D'))->format('Y-m-d H:i:s');
```

- 比较日期大小 
```php
$t = new DateTimezone('Asia/Shanghai');
$d1 = new DateTime('2014-12-01', $t);
$d2 = new DateTime('2014-12-02', $t);

echo $d1 > $d2 ? 'd1 bigger than d2' : 'd1 less than d2';
```

- 获取日期间的差异
```php
$diff = $d1->diff($d2);

print_r($diff);

DateInterval Object
(
    [y] => 0
    [m] => 0
    [d] => 1
    [h] => 0
    [i] => 0
    [s] => 0
    [weekday] => 0
    [weekday_behavior] => 0
    [first_last_day_of] => 0
    [invert] => 0
    [days] => 1
    [special_type] => 0
    [special_amount] => 0
    [have_weekday_relative] => 0
    [have_special_relative] => 0
)
```







