## 1.字符串的使用
```
可将字符串当作一个字符的集合来使用，可独立访问每个字符。仅适用于单字节字符（字母、数字、半角标点符号），像中文等不可用（utf8下，中文3字节表示）
$str = "abcd";
echo $str[3];   // d
echo $str{0};   // a
//另一种方式
也可以使用str_split把所有字符分割成数组。
$strArr = str_spilt($str);   //同样不适合中文
Array
(
    [0] => a
    [1] => b
    [2] => c
    [3] => d
)
function mb_str_split($str){
    return preg_split('/(?<!^)(?!$)/u', $str );   //反向预搜索
}
$str='博客';
mb_str_split($str);
打印结果如下：
Array
(
    [0] => 博
    [1] => 客
)
```

## 2.转换类型

平时 我们知道在变量前，加上(int), (string), (float) 之类的即可进行显示转换，下面介绍一种，可直接转换为null的 转换，。
(unset) //转换为NULL
```
$a = 'a';
$a = (unset)$a;  //此时$a 为 NULL
//换种方式
$a = 'a';
unset($a);
dd($a);
//报错
PHP Notice:  Undefined variable: a in test.php on line 39

Notice: Undefined variable: a in test.php on line 39
```


## 3.isset 和 empty, is_null 对变量的检测
3.1 isset 要检测变量不存在则， 返回 false； 但需要注意，变量是否本身为 Null值， 如果是，也返回 false； 都不满足则返回true了 。
为了更方便检测变量是否存在，可使用
建议用array_key_exists判断
```
$a = null;
$varExists = array_key_exists('a', get_defined_vars());
if($varExists){
	    echo '变量a是存在的';
}
```
![file](https://graph.baidu.com/resource/212bf18de0815260cb01f01570773807.png)
3.2 empty， 检测变量不存在，与各个空值，0值都为true， 否则为 false
3.3 is_null 
当参数满足下面三种情况时，is_null()将返回TRUE，其它的情况就是FALSE
1、它被赋值为NULL
2、它还没有赋值
3、它未定义，相当于unset(),将一个变量unset()后，不就是没有定义吗

## 4.static 的部分用法

我们都知道 static 可以直接定义静态变量和 静态方法， 无论是调用静态变量和静态方法， 用 :: 即可调用， 如果是本类，则使用self 调用， 如果是在类
外部，则使用类名进行调用， 但这里对其不进行过多赘述，这些都是比较常用。下面讲下不常用的。

static在类中的延迟静态绑定；

```
延迟静态绑定是指允许在一个静态继承的上下文中引用被调用类。
延迟绑定的意思为：static::不再为定义当前方法所在的类，而是实际运行时所在的类。
(总的意思就是子类继承了父类后，父类中使用了一个静态方法或变量，如static::A()， 但方法A不存在于父类中，
存在于子类中，这样写是正确的，不会因为方法不存在而报错，因为static 修饰的A 会在执行过程中，绑定到具体的子类
上，只要保证子类有此方法即可)
注：它可以用于(但不限于)静态方法的调用。
self::，代表本类(当前代码所在类)
    永远代表本类，因为在类编译时已经被确定。
    即，子类调用父类方法，self却不代表调用的子类。
parent:: 代表父类
static::，代表本类(调用该方法的类)
    用于在继承范围内引用静态调用的类。
    运行时，才确定代表的类。
    static::不再被解析为定义当前方法所在的类，而是在实际运行时计算的。
除了简单的static延迟绑定的用法，还有一种转发调用，
forward_static_call_array()(该函数只能在方法中调用)将转发调用信息（不过多赘述，类似于本节的第8点，PHP反射），
概括实例如下：
```
```
class A {
    public static function fooStatic() {
        static::who();
    }
    public static function who() {
        echo __CLASS__."aaa\n";
    }
}
class B extends A {
    public static function testStatic() {
        A::fooStatic();
        B::fooStatic();
        C::fooStatic();
        parent::fooStatic();
        self::fooStatic();
    }
    public static function who() {
        echo __CLASS__."bbb\n";
    }
}
class C extends B {
    public static function who() {
        echo __CLASS__."ccc\n";
    }
}
C::testStatic();
大家一起来得到答案：
```
## 5. array_merge 和 + ， 合并数组

5.1 array_merge 合并两个或多个数组的时候， 如果键名是非数字，则会自动合并到一起，但若出现重复的键名，后面的会覆盖前面的。若键名为数字，其会根据前一个数组的索引值，自动给后面的更改索引值为前面的递增，也就是说会改变原有的键名。

5.2 + 合并数组， 使用+ 号 合并数组，键名相同的情况下，最先出现的键名会被保留， 并且即使键名是数字，也不会被重新编排，这个特性对部分业务场景会非常有用。

5.3 array_push, 仅仅是把对应值完全放到数组末尾， 原来的结构会被保留，整个作为一个新的值， 键名跟着前一个递增。这与array_merge 有所区别

5.4 array_combine， 两个参数， 第一个参数，作为新数组的键， 第二个参数作为新数组的值。 并不是给原有数组增添新值。所以其实他不同于 上面三个的作用。 光从名字上看容易混淆。
```
$a = ['a', 'b', 'c'];
$b = ['5' => 'test'];
$c = [1, 2, 3];

$arrayMerge   = array_merge($a, $b);
$arrayPlus    = $a + $b;
$arrayCombine = array_combine($a, $c);
array_push($a, $b);
// $arrayMerge
Array
(
    [0] => a
    [1] => b
    [2] => c
    [3] => test
)
// $arrayPlus
Array
(
    [0] => a
    [1] => b
    [2] => c
    [5] => test
)
// $arrayCombine
Array
(
    [a] => 1
    [b] => 2
    [c] => 3
)
// array_push
Array
(
    [0] => a
    [1] => b
    [2] => c
    [3] => Array
        (
            [5] => test
        )

)
```

## 6. 指针

```
数组的内部指针：
current/pos 返回当前被内部指针指向的数组单元的值，并不移动指针。
key         返回数组中当前单元的键名，并不移动指针
next        将数组中的内部指针向前移动一位，并返回移动后当前单元的值。先移动，再取值。
prev        将数组的内部指针倒回一位，并返回移动后当前单元的值先移动，再取值。
end         将数组的内部指针指向最后一个单元，并返回最后一个单元的值
reset       将数组的内部指针指向第一个单元，并返回第一个数组单元的值
each        返回数组中当前的键/值对并将数组指针向前移动一步。
            返回的是一个由键和值组成的长度为4的数组，下标0和key表示键，下标1和value表示值
在执行each()之后，数组指针将停留在数组中的下一个单元或者当碰到数组结尾时停留在最后一个单元。如果要再用 each 遍历数组，必须使用 reset()。
1. 以上指针操作函数，除了key()，若指针移出数组，则返回false。而key()移出则返回null。
2. 若指针非法，不能进行next/prev操作，能进行reset/end操作
3. current/next/prev     若遇到包含空单元（0或""）也会返回false。而each不会！

list    把数组中的值赋给一些变量。list()是语言结构，不是函数。
仅能用于数字索引的数组并假定数字索引从0开始
/* 可用于交换多个变量的值 */ 
list($a, $b) = array($b, $a);
例：list($drink, , $power) = array('coffee', 'brown', 'caffeine');
1. 复制数组，其指针位置也会被复制。
    特例：如果数组指针非法，则拷贝的数组指针会重置，而原数组的指针不变。
    【指针问题】
        谁第一个进行写操作，就会开辟一个新的值空间。与变量(数组变量)值传递给谁无关。
        数组函数current()被定义为写操作，故会出现问题。
        foreach遍历的是数组的拷贝，当被写时，才会开辟一个新的值空间。
            即，foreach循环体对原数组进行写操作时，才会出现指针问题。
            如果开辟新空间时指针非法，则会初始化指针。
2. 如果指针位置出现问题，则reset()初始化一下就可解决。
```
## 7. 序列化（串行化）
```angular2html
# 数据传输均是字符串类型
# 除了资源类型，均可序列化
# 序列化在存放数据时，会存放数据本身，也会存放数据类型
作用：1.在网络传输数据时；2.为了将数组或对象放在磁盘时
# 序列化
serialize        产生一个可存储的值的表示
string serialize ( mixed $value )
- 返回字符串，此字符串包含了表示value的字节流，可以存储于任何地方。
- 有利于存储或传递 PHP 的值，同时不丢失其类型和结构。
# 反序列化
unserialize        从已存储的表示中创建PHP的值
mixed unserialize ( string $str [, string $callback ] )
- 对单一的已序列化的变量进行操作，将其转换回PHP的值。
```

## 8.PHP反射
有时候我们我们为了能建立更好的代码结构，并能复用代码，就需要用到一些语言的特性来帮助我们完成任务，比如我们这次新建的仓库 trinity_middle_service， 我们需要利用PHP自带函数， 如call_user_func_array， 即可简单实现分发。使用方式如下：
```
class Order{
    public function getBasicInfo($orderId, $write = false)
    {
    }
}
$detailServiceInstance = new Order();
$functionName = 'getBasicInfo';
$params[0] = 1000001;    //或 $params['orderId'] = 1000001
$params[1] = true;       //或 $params['write'] = true
call_user_func_array([$detailServiceInstance, $functionName], $params);
```
从以上代码可看出， call_user_func_array 第一个参数是个数组， 该数组中0下标的值  为 一个类的实例， 1下标的 是这个实例中存在的方法； 第二个参数即 对应函数需要的参数了，该参数为数组形式，call_user_func_array 会自动 按顺序对应每一个参数，所以在这里，参数的顺序很重要，无论是使用数字键名或字符键名，都必须一一对应函数具体的参数，这样才能正确实现函数的分发。

### 8.1 反射的应用
使用call_user_func_array，我们可以简单的实现分发，并且各个参数，也都很清晰。但是如果需要更加灵活的运用这个函数，我们还需要引入PHP的反射机制，如 PHP自带的 ReflectionMethod 类，
    我们可以通过这个类，来直接获取到某个类中具体方法的各类信息，如参数个数，参数名，是否有默认值等等。代码如下：
```
$object = new Order();
$functionName = 'getBasicInfo';
$reflectionObj = new \ReflectionMethod($object, $functionName);
//通过以上的代码后，就可以直接通过$reflectionObj获取相关信息了，如下
$reflectionObj->isPublic();   //可判断该方法是否为公有方法， 还有isPrivate, isProtected, isStatic等等
$reflectionObj->getParameters();   //可获取该方法所有参数，是个数组，可 foreach获取具体的参数
foreach($reflectionObj->getParameters() as $arg) {
    if(array_key_exists($arg->name, $params)) {
        $callParams[$arg->name] = $params[$arg->name];
    } else {
        $paramVal = null;
        if ($arg->isOptional()) {  // 或使用isDefaultValueAvailable， 检测是否有可用的默认值
            $paramVal = $arg->getDefaultValue();
        }
        $callParams[$arg->name] = $paramVal;
    }
}
具体参数： https://www.php.net/manual/zh/class.reflectionmethod.php
```
 还有 ReflectionClass 可以获取一些类的信息，用法都是类似的。
### 8.2  调用不存在的方法
 __call & __callStatic (魔术方法)
 当调用类中一个不存在或者没有权限访问的方法的时候，就会自动调用__call()方法。和__call对应静态方法的调用是__callStatic， 配合PHP反射，对代码进行优化，可对一些特殊的基础业务功能做到，
 如工单, trinity 内部都是用了类似方法。

## 9. move_uploaded_file
该函数是前端文件上传文件到后台后可能用到的函数，上传后，我们会通过$_FILES['file']['tmp_name'](文件上传后地址)， 使用
```
move_uploaded_file($tmp_name, $savePath);
```
会把临时存到数据库的 如： /tmp/phpwTJzUw  文件给移动走，导致如果后面的代码还要用到$_FILES全局变量的时候，会出现找不到文件的问题。

## 10.引用传递
在使用array_pop等函数时， array_pop的参数 是引用传递，在编码过程中有时候会把 参数直接写成函数返回值，这样在 5.3以后是不允许的， 会报出致命错误（严格模式下）。
```
You will see:
PHP Strict Standards:  Only variables should be passed by reference in - on line 3

Strict Standards: Only variables should be passed by reference in - on line 3
d
以上来自于： https://www.php.net/manual/zh/function.array-pop.php

按引用传递参数的函数在被按值传递调用时行为发生改变. 此前函数将接受按值传递的参数, 现在将抛出致命错误. 之前任何期待传递引用但是在调用时传递了常量或者字面值 的函数, 需要在调用前改为将该值赋给一个变量。
以上来自于：PHP手册附录从PHP 5.2.x 移植到 PHP 5.3.x 部分
https://www.php.net/manual/zh/migration53.incompatible.php
```
而实际上看起来没有错， 只需要把那个函数返回值赋值给一个变量，再把变量当做参数传入就可以了。
如，在代码中经常会写到：

```
error_reporting(E_STRICT);
$testArr = ['a','b','c','c','d'];
$popValue = array_pop(array_unique($testArr));  
// 以上代码会报错，必须使用
$testArr = ['a','b','c','c','d'];
$testArr = array_unique($testArr)
$popValue = array_pop($testArr);
```