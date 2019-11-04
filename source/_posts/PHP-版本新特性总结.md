---
title: PHP 版本新特性总结
date: 2019-10-28 15:11:00
tags:
	- PHP
categories:
	- 随记
---

闲来无事，就想着总结一下各个版本特性。然后就有下面的文档

#### PHP 5.6 新特性

##### 使用表达定义常量

在之前的版本里，必须使用**静态值**来定义常量，声明属性以及制定函数参数默认值。

现在当前版本可以使用包括**数值、字符串以及其他常量的数值表达式**来定义常量。

```php
<?php

const ONE = 1;

const TWP = ONE * 2;

echo TWP;

### PHP5.5
# PHP Parse error:  syntax error, unexpected '*'

### 在之后版本
# 2
```



##### 使用 ** 进行运算

加入右连接运算符 `**` 来进行运算，还支持简写的 `**=` 运算符，表示进行幂运算并赋值

```php
<?php
$a = 2;

$b = 2 ** 3;

echo $a **= 4;
echo "\r\n";
echo 2 ** 3 ** 2;
echo "\r\n";
echo $b;

### 输出
16
512
8
```

> 这里讲一下：`2 ** 3 ** 2` 这个表达式，先给后面的 `3 ** 2` 进行幂运算，然后才是给前面的进行运算



##### use function 以及 use const

`use` 运算符被进行了扩展，支持了在类中导入外部的函数和常量，对应的结构：`use function` 和 `use const`

```php
<?php
    namespace App {
    	const FOO = 1;
    
    	function test()
        {
            echo __FUNCTION__ . "\n";
        }
	}

	namespace {
        use const App\FOO;
        use function App\test;
        
        echo FOO."\r\n";
        test();
    }


### 输出
1
App\test
```



##### php://input 是可重用的

只要你需要，你可以多次打开并读取 `php://input` 。



##### 大文件上传

现在可以支持大于**2GB**的文件上传



##### GMP 支持运算符重载

[GMP]( https://www.php.net/manual/zh/book.gmp.php ) 支持运算符重载，并且造型成数值类型。

>  GMP是The GNU MP Bignum Library，是一个开源的数学运算库，它可以用于任意精度的数学运算，包括有符号整数、有理数和浮点数。它本身并没有精度限制，只取决于机器的硬件情况。 



##### 使用 hash_equals() 比较字符串避免时序攻击

加入 [hash_equals()]( https://www.php.net/manual/zh/function.hash-equals.php) 函数，以恒定的时间消耗来进行字符串比较，以避免时序攻击。比如当比较 `crypt()` 密码三列值的时候，就可以使用此函数。（[password_hash()](https://www.php.net/manual/zh/function.password-hash.php) 和 [password_verify()](https://www.php.net/manual/zh/function.password-verify.php) 也可以抵抗时序攻击）

> **时序攻击：**  属于侧信道攻击/旁路攻击(Side Channel Attack)，侧信道攻击是指利用信道外的信息，比如加解密的速度/加解密时芯片引脚的电压/密文传输的流量和途径等进行攻击的方式，一个词形容就是“旁敲侧击” 。
>
> 举一个最简单的计时攻击的例子，某个函数负责比较用户输入的密码和存放在系统内密码是否相同，如果该函数是从第一位开始比较，发现不同就立即返回，那么通过计算返回的速度就知道了大概是哪一位开始不同的，这样就实现了电影中经常出现的按位破解密码的场景。密码破解复杂度成千上万倍甚至百万千万倍的下降。



##### pgsql 异步支持

[pgsql]( https://www.php.net/manual/zh/book.pgsql.php) 扩展现在支持异步方式连接数据库及执行查询，也可以使用非阻塞的方式和 `PostgreSQL` 数据库进行交互。



#### PHP 7.0 新特性

##### 标量类型声明

标量 [类型声明]( https://www.php.net/manual/zh/functions.arguments.php#functions.arguments.type-declaration ) 有两种模式：**强制（默认）**和 **严格模式** 。现在可以使用：字符串、整数、浮点数、布尔值。她们扩充了 PHP5 中引入的其他类型：类名、接口、数组和回调类型。

```php
<?php
    function sumOfInts(int ...$ints)
	{
    	return array_sum($ints);
	}

	var_dump(sumOfInts(2, '3', 4.1));

### 输出
int(9)
```



##### 返回值类型声明

PHP7 增加了对返回类型声明的支持。指明了函数返回值的类型。

```php
<?php
function arraysSum(array ...$arrays): array
{
    return array_map(function (array $array): int {
        return array_sum($array);
    }, $arrays);
}

print_r(arraysSum([1, 2, 3], [4, 5, 6]));

### 输出
Array
(
    [0] => 6
    [1] => 15
)
```



##### null合并运算符

日常使用中存在大量同时使用三元表达式和 `isset()` 的情况，PHP7 添加了**null合并运算符（??）** 。如果变量存在且值不为 **NULL** ，它就会返回自身的值，否则返回他的第二个操作数

```php
<?php
$username = $_GET['user'] ?? 'nobody';

echo $username;

##### 输出
nobody
```



##### 太空船操作符（组合比较符）

太空船操作符用于比较两个表达式。当 `$a` 小于、等于或者大于 `$b` 时他分别返回 -1、0 或 1。

```php
<?php
# 整数
echo 1 <=> 2;  // -1
echo 1 <=> 1;  // 0
echo 2 <=> 1;  // 1

# 浮点数
echo 1.5 <=> 2.5;  // -1
echo 1.5 <=> 1.5;  // 0
echo 2.5 <=> 1.5;  // 1

# 整数
echo "a" <=> "b";  // -1
echo "a" <=> "a";  // 0
echo "b" <=> "a";  // 1
```



##### 通过 define() 定义常量数组

Array 类型的常量现在可以通过 `define()` 来定义。在 PHP 5.6 中仅能通过 `const` 定义。

```php
<?php
define('ANIMALS', [
    'dog',
    'cat',
    'bird'
]);

echo ANIMALS[2];

### 输出
bird
```



##### Closure::call()

`Closure::call()` 有着更好的性能，剪短干练的暂时绑定一个方法到对象上闭包并调用它。

```php
<?php
class A {
    private $x = 1;
}

// PHP 7 之前版本的代码
$getXCB = function () {
    return $this->x;
};

$getX = $getXCB->bindTo(new A, 'A'); // 中间层闭包

// PHP 7+ 
$getX = function() {
    return $this->x;
};

echo $getx->call(new A);

```



##### 为 unserialize() 提供过滤

这个特性旨在提供更安全的方式解包不可靠的数据。他通过白名单的方式来防止潜在的代码注入。



##### 预期

[预期]( https://www.php.net/manual/zh/function.assert.php#function.assert.expectations ) 是向后兼用并增强之前的 `assert()` 的方法。它使得在生产环境中启用断言为零成本，并提供断言失败时抛出特定异常的能力。

```php
<?php
ini_set('assert.exception', 1);

class CustomError extends AssertionError() {}

assert(false, new CustonError('some error message'));
```



##### Group use declarations

从同一个 `namespace` 导入的类、函数和常量现在可以通过单个 `use` 语句一次性导入。

```php
<?php
    
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
```



##### 整数除法函数 intdiv()

**新加**的函数 `intdiv()` 用来进行整数的除法运算

```php
<?php

var_dump(intdiv(10, 3));

### 输出
int(3)
```



##### 会话选项

[session_start()](https://www.php.net/manual/zh/function.session-start.php) 可以接受一个 `array` 作为参数，用来覆盖 php.ini 文件中设置的**会话配置选项**。



##### CSPRNG Functions

新加入两个跨平台的函数：`random_bytes()` 和 `random_int()` 用来产生高安全级别的随机字符串和随机整数。

```php
<?php
$bty = random_bytes(4);

var_dump(bin2hex($bty));

$ints = random_int(1, 1000);

var_dump($ints);

### 输出
string(8) "093a14a4"
int(746)
```

>  [bin2hex()](https://www.php.net/manual/zh/function.bin2hex.php) - 函数把包含数据的二进制字符串转换为十六进制值 



##### 可以使用 list() 函数来展开实现了 ArrayAccess 接口的对象



#### PHP 7.2 新特性

##### 可为空（Nullable）类型

参数以及返回值的类型现在可以通过在类型前加上一个问号使之允许为空。当启用这个特性时，传入的参数或者函数返回的结果要么是给定的类型，要么是 null。

```php
<?php
function testReturn(): ?string
{
    return 'elePhPant';
}

var_dump(testReturn());

function testReturn1(): ?string
{
    return null;
}

var_dump(testReturn1());

function test(?string $name)
{
    var_dump($name);
}

test('elePHPant');
test(null);
test();


### 输出
string(9) "elePhPant"
NULL
string(9) "elePHPant"
NULL
PHP Fatal error:  Uncaught ArgumentCountError: Too few arguments to function test(), 0 passed
```



##### void 函数

一个新的返回值类型 void 被引入。返回值声明为 void 类型的方法要么干脆省去 return 语句，要么使用一个空的 return 语句。对于 void 函数来说，NULL 不是一个合法的返回值



##### Symmetric array destructuring

短数组语法（`[]`）现在作为 `list()` 语法的一个备选项，可以用于将数组的值赋给一些变量（包括在 foreach 中）。

```php
<?php
$data = [
    [1, 'Tom'],
    [2, 'Fred']
];

// 使用 list()
list($id1, $name1) = $data[0];

foreach($data as list($ids, $name)) {
    
}

// 使用 []
[$id1, $name1] = $data[0];

foreach($data as [$id, $name]) {
    
}
```



##### 类常量可见性

现在起支持设置类常量的可见性。

```php
<?php
class ConstDemo
{
    const PUBLIC_CONST_A = 1;
    public const PUBLIC_CONST_B = 2;
    protected const PROTECTED_CONST = 3;
    private const PRIVATE_CONST = 4;
}
```



##### list() 现在支持键名

现在 `list()` 和它的新的 `[]` 语法支持在它内部指定键名。



##### 多异常捕获处理

一个 `catch` 语句块现在可以通过管道字符 （`/`）来实现多个异常的捕获。

```php
<?php
try {
    
} catch (FirstException | SecondException $e) {
    
}
```



##### 支持为负的字符串偏移量

现在所有支持偏移量的 [字符串操作函数](https://www.php.net/manual/zh/book.strings.php) 都支持接收负数作为偏移量，包括通过 `[]` 或 `{}` 操作字符串下标。在这种情况下，一个负数的偏移量会被理解为一个从字符串结尾开始的偏移量

```php
<?php
var_dump("abcdef"[-2]);
var_dump(strpos("aabbcc", 'b', -3));

### 输出
string(1) "e"
int(3)
```

​	

。。。。。

PHP7.3、、、PHP7.4。。。官方还没有给出中文文档所以

之后跟进



***文档参考：*** [PHP官方文档]( https://www.php.net/manual/zh/appendices.php )