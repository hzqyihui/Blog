## 1. __autoload
这是一个自动加载函数，在PHP5中，当我们实例化一个未定义的类时，就会触发此函数。看下面例子：
```
./myClass.php
<?php
class myClass {
    public function __construct() {
        echo "myClass init'ed successfuly!!!";
    }
}
?>
./index.php
<?php
// we've writen this code where we need
function __autoload($classname) {
    $filename = "./". $classname .".php";
    include_once($filename);
}
// we've called a class ***
$obj = new myClass();
?>
```
从上面能看到这是两个文件，下面的index.php 中，new了个  myClass类，但是明显本文件不存在，现在就会自动调用 __autoload函数，并 把 “myClass”这个类名字符串 直接作为参数传给__autoload， 此时自动加载函数内部就可以引入该文件了，引入后就正常初始化该类了。 该函数在PHP 7.2.0后被废弃了。

## 2. spl_autoload_register
spl_autoload_register 可以将 函数自动注册，也就是说，当PHP文件内访问了一个不存在的类时，会自动去调用该函数，然后执行该函数内部的函数，看起来和 **autoload的作用是一样的。但是其实spl_autoload_register 这个函数功能更强大，** autoload的参数 仅仅是一个函数名，这是定死的。并且只能声明一次， 使用了autoload后，就不能再次使用该函数了。

请注意：一个项目中只能有一个__autoload， 如果在PHP在执行过程中遇到两个__autoload会直接报错的。

很明显，**autoload无法满足要求， 所以就有了SPL扩展，spl_autoload_register接受函数名或闭包，或数组作为参数，在闭包内部，即可引入对应的文件了。并且spl_autoload_register可以注册一个 自动加载队列，先注册的，先调用。**
```
参数 
autoload_function
欲注册的自动装载函数。如果没有提供任何参数，则自动注册 autoload 的默认实现函数spl_autoload()。
throw
此参数设置了 autoload_function 无法成功注册时， spl_autoload_register()是否抛出异常。
prepend
如果是 true，spl_autoload_register() 会添加函数到队列之首，而不是队列尾部。
```

可以结合require_once一起使用。如：
```
function_1(){
 	$clsName = str_replace("\\",DIRECTORY_SEPARATOR, $class_name);
    if (is_file(__DIR__.DIRECTORY_SEPARATOR."src".DIRECTORY_SEPARATOR.$clsName . '.php')) {
        //文件内部有类名 为 TestClass_1的类
        require_once(__DIR__.DIRECTORY_SEPARATOR."src".DIRECTORY_SEPARATOR.$clsName.'.php'); 
    }
}
function_2(){
 	$clsName = str_replace("\\",DIRECTORY_SEPARATOR, $class_name);
    if (is_file(__DIR__.DIRECTORY_SEPARATOR."Module".DIRECTORY_SEPARATOR.$clsName . '.php')) {
        //文件内部有类名为TestClass_2的类
        require_once(__DIR__.DIRECTORY_SEPARATOR."Module".DIRECTORY_SEPARATOR.$clsName.'.php');
    }
}
spl_autoload_register('function_1');
spl_autoload_register('function_2');
$obj = new TestClass_2();  //当前没有TestClass_2这个类，于是自动调用function_1， 引入了文件，但是引入的文件中仍然没有TestClass_2这个类，于是又自动调用function_2， 引入了文件，此时正常初始化
```

## 3.相关的其他SPL函数
![file](https://graph.baidu.com/resource/2127e06b38dadd7f22e1701570778876.png)
### 3.1 spl_autoload_call
该函数是需要用户显示调用所有已注册的 autoload函数的。 作用在 spl_autoload_register之后。 传入函数名字即可。即可手动引入文件了。
### 3.2 spl_autoload_functions
可以获取到所有已经注册的autoload函数， 也是作用在  spl_autoload_register之后的。
### 3.3 spl_autoload_extensions
注册并返回spl_autoload函数使用的默认文件扩展名， 但是此接口和spl_autoload函数，用处不大。spl_autoload 是autoload的默认实现，意思就是spl_autoload对autoload进行了又一次封装，在默认情况下，本函数先将类名转换成小写，再在小写的类名后加上 .inc 或 .php 的扩展名作为文件名，然后在所有的包含路径(include paths)中检查是否存在该文件。

__autoload 函数是用来处理自动加载的函数，在 PHP 找不到指定类时就会去调用自动加载类，加载所需要的类。
__autoload 只是一个抽象定义，实现（实现就是定义如何加载，加载的规则是什么，加载的文件是什么等等）是交给用户的，而 spl_autoload 则是 SPL 所定义的 autoload 一种实现。spl_autoload 函数所实现的加载规则就是去 include paths 中查找对于的类。spl_autoload 遵循是是 psr-0 的载入规则，而 include paths 就是载入时被查询的路径。
其他自己实现的 autoload 类都可以通过 spl_autoload_register 进行注册，注册之后就可以在需要类时自动调用被注册的方法进行加载了。 spl_autoload 也是 autoload 的一种实现，按理也是需要注册的，只不过因为是内部的默认实现，所有已经自动注册在 PHP 里了。

spl_autoload 如今来看并没有太多用处，应该是因为历史问题残留在 PHP 中的，目前绝大多数程序都没有使用 spl_autoload 去做自动加载，因为它的规则已经定死，并不适合衍生一些功能。

因为 PHP 只有一个自动加载方法，所以 SPL 的 spl_autoload 和 spl_autoload_register 要争抢这个方法，所以在 SPL 的 C 实现中，用了好多折衷的办法。在没有使用 spl_autoload_register 注册任何自定的自动加载函数时， PHP 的自动加载方法是挂在 spl_autoload 下的，而 spl_autoload_register 注册了自动加载函数后，PHP 的自动加载方法是挂在 spl_autoload_call 这个方法下的，而 spl_autoload 也会成为一个备选项进入 spl_autoload_register 的自动加载队列。