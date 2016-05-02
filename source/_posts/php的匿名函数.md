title: php的匿名函数
date: 2015-03-11 22:13:25
tags: 
- php
- closures
---


```
匿名函数
 
匿名函数（Anonymous functions），也叫闭包函数（closures），
允许 临时创建一个没有指定名称的函数。最经常用作回调函数（callback）参数的值。
当然，也有其它应用的情况。
```

闭包的写法

## 5.2本版以下

看起来好怪异

    <?php
    $newfunc  =  create_function('$a, $b',
      'return "ln($a) + ln($b) = " . log($a * $b);' 
    );
    echo  "New anonymous function:  $newfunc \n" ;
    echo  $newfunc ( 2 ,  M_E ) .  "\n" ;
     // outputs
    // New anonymous function: lambda_1
    // ln(2) + ln(2.718281828459) = 1.6931471805599
    ?> 


##5.3版本以上
和js很相像。

    <?php
    $greet  = function( $name )
    {
         printf ( "Hello %s\r\n" ,  $name );
    };
    
     $greet ( 'World' );
     $greet ( 'PHP' );

打印$greet变量结果一下

    object(Closure)#1 (1) {
      ["parameter"]=>
      array(1) {
        ["$name"]=>
        string(10) "<required>"
      }
    }
可以看出$greet其实一个[Closure](http://php.net/manual/zh/class.closure.php)对象实例。
$greet()其实执行的是魔术方法[__invoke()](http://php.net/manual/zh/language.oop5.magic.php#object.invoke)。

代码如下:

    <?php
    
    /**
     * 
     */
    class Obj
    {
        /**
         * [__invoke description]
         * @param  [type] $name [description]
         * @return [type]       [description]
         */
        public function __invoke($name)
        {
            printf ( "Hello %s\r\n" ,  $name );
        }

        /**
         * [test description]
         * @return [type] [description]
         */
        public function test()
        {
            return function($name){
                printf ( "Hello %s\r\n" ,  $name );
            };
        }
    }
    
    $greet = new Obj();
    $greet("World"); //Hello World
    $hello = $greet->test();
    $hello('zhang san'); //Hello zhang san

##作用域
匿名函数闭包使用``use``关键字使用外部变量。

    <?php
    class Obj
    {
        public function hello()
        {
            $name = 'zhangsan';
            $greet = function($prefix) use ($name) {
                echo "{$prefix}, {$name}";
            };
    
            $greet('hi');
        }
    }
    
    
    $obj = new Obj();
    $obj->hello(); //hi, zhangsan

``$this``是一个特殊变量不能直接使用(作用域的原因参考js的[this 的工作原理](https://bonsaiden.github.io/JavaScript-Garden/zh/#function.this))。可以使用如下代码:

    <?php
    class Obj
    {
        public $name = 'zhangsan';
    
        public function hello()
        {
            $that = $this;
            $greet = function($prefix) use ($that) {
                echo "{$prefix}, {$this->name}";
            };
    
            $greet('hi');
        }
    }
    
    
    $obj = new Obj();
    $obj->hello(); //hi, zhangsan

##绑定函数到一个对象
绑定使用的是``Closure::bind/bindTo`` — 复制一个闭包，绑定指定的$this对象和类作用域。

    <?php
    $f = function()
    {
        return $this->value + 2;
    };
    
    class NewValueHolder
    {
        public $value;
    }
    
    $vh = new NewValueHolder();
    $vh->value = 3;
    $c = $f->bindTo($vh);
    var_dump($c()); //Fatal error: Cannot access protected property NewValueHolder::$value

上述代码会报错是因为$this的作用并没有发生变化。
```
public Closure  Closure::bindTo  ( object $newthis  [, mixed  $newscope  = 'static'  ] )

 newscope  
关联到匿名函数的类作用域，或者 'static' 保持当前状态。如果是一个对象，则使用这个对象的类型为心得类作用域。 这会决定绑定的对象的 保护、私有成员 方法的可见性。 
```

修改bindTo为``$c = $f->bindTo($vh, $vh);``则可以按照预期运行。

[带你穿越带你飞～，一秒钟PHP变JS ＝^ ＝!!!](http://levi.cg.am/archives/3602)这篇文章有更淫荡的用法。


## 闭包的应用
1. 作为回调函数(``array_map, array_reduce,array_walk``等)

        <?php
        $data [] = array( 'volume'  =>  67 ,  'edition'  =>  2 );
        $data [] = array( 'volume'  =>  86 ,  'edition'  =>  1 );
        $data [] = array( 'volume'  =>  85 ,  'edition'  =>  6 );
        $data [] = array( 'volume'  =>  98 ,  'edition'  =>  2 );
        $data [] = array( 'volume'  =>  86 ,  'edition'  =>  6 );
        $data [] = array( 'volume'  =>  67 ,  'edition'  =>  7 );
        
        $volumes = array_map(function($row){
            return $row['volume'];
        }, $data);
        
        
        
        $volumeSum = 0;
        $volumeSum = array_reduce($volumes, function($volumeSum, $current){
            $volumeSum += $current;
            return $volumeSum;
        });
        
        print_r($volumes); //volume列的所有值
        
        print_r($volumeSum); //volume列的总和

2. 复用函数或方法的代码片段
        <?php
        
        /**
         * 
         */
        class Obj
        {
            public function calc($num)
            {
                $space = function($num) {
                    return $num + 1;
                };
        
                if(逻辑1) {
                    $num1 = $space($num);
                } elseif(逻辑2) {
                    $num2 = $space($num2);
                }
            }
        }


参考

[Closure 类](http://php.net/manual/zh/class.closure.php)

[匿名函数](http://php.net/manual/zh/functions.anonymous.php)

[$this in PHP closures](http://research.quanbit.com/2013/01/04/this-in-php-closures/)

