# 语法参考

### 类型

------

php支持9种原始数据类型

**4种标量类型**

boolean

integer

float / double

string

**3种复合类型**

array

object

callable（可调用）

**2种特殊类型**

resource

NULL

**伪类型**

mixed（混合类型）

number（数字类型）

callback（回调类型，又称callable）

array|object（数组|对象类型）

void（无类型）

> 变量的类型通常不是程序员设定的，而是PHP根据该变量使用的上下文在运行时决定的

获取变量类型gettype()函数，校验类型使用is_type

**Boolean布尔类型**

true|false不区分大小写；

使用(bool)或(boolean)来强制转换；

当转为boolean时，以下为FALSE：

- 布尔值false本身
- 整型值0
- 浮点型值0.0
- 空字符串，以及字符串“0”
- 不包含任何元素的数组
- 特殊类型NULL（包含尚未复制的变量）
- 从空标记生成的SimpleXML对象

> 所有其他值都被认为是true（包括任何资源和NAN）

**Integer整型**

整型可以使用十进制、十六进制0x、八进制0或二进制0b表示。

最大值可用PHP_INT_MAX表示，php7之后可用PHP_INT_MIN

> php7以前的版本，如果向八进制数传递了一个非法数字（如8或9），则后面的数字会被忽略，php7以后，会产生parse error

如果给定的或运算结果一个数超出integer范围，会被解释为float。

可使用(int)或(integer)强制转换，还可以通过函数intval()来将一个值转为整型。

从浮点数转为整型，如浮点数超出了整数范围，则结果为未定义，无警告信息。

> 自php7起，NaN和Infinity在转换成integer时，不再是undefined或依赖于平台，而都会变成0

> 不要将未知的分数强制转换为integer，否则可能导致不可预料的结果。
>
> ```php
> echo (int) ( (0.1+0.7) * 10 ); //结果为7
> ```

**Float浮点型**

浮点型也叫浮点数float，双精度数double或实数real。

定义：1.234  1.2e3  7E-10

> 浮点数的精度
>
> 浮点数的精度有限。此外，以十进制能够精确表示的有理数如0.1或0.7，无论有多少尾数都不能被内部所使用的二进制精确表示，因此不能在不丢失一点点精度的情况下转换为二进制的格式。如floor((0.1+0.7)*10)通常返回7，因内部结果类似7.9999999991188...

比较浮点数：

要测试浮点数是否相等，要使用一个仅比该数值大一丁点的最小误差值。该值也被成为“机器极小值（epsilon）”或“最小单元取整数”，是计算中所能接受的最小的差别值。

```php
$a = 1.23456789;
$b = 1.23456780;
$epsilon = 0.00001;
if(abs($a-$b) < $epsilon){
    echo "true";
}
```

NaN 拿此值与其他任何值（除true）进行松散或严格比较的结果都是false。

由于NaN代表任何不同值，不应哪NaN去和其他值进行比较，包括其自身，应该用is_nan()检查。

**String字符串**

php只能支持256的字符集，不支持unicode。

string最大可达2GB。

字符串4种表达方式：

- 单引号
- 双引号
- heredoc语法结构
- nowdoc语法结构（自php5.3起）

heredoc句法结构：<<<。结束时所引用的标识符必须在该行的第一列，本行除分号外不可包含其他字符，且结束标识符的前面及后面必须有一个被本地操作系统认可的换行。

heredoc结构不能用来初始化类的属性，自php5.3起，此限制仅对heredoc包含变量时有效。

nowdoc结构是类似于单引号字符串的。nowdoc跟在<<<后的标识符要用单引号括起来。

nowdoc结构可以用在任意的静态数据环境中，最典型的示例是用来初始化类的属性或常量。

字符串可通过下标访问，如$str[42]或$str{42}。

用超出字符串长度的下标写入将会拉长该字符串并以空格填充。

非整数类型下标会被转换成整数（自php5.4，下标必须为整数或可转化为整数的字符串，否则发出警告，之前类似foo的下标会无声地转为0）。

非法下标类型会产生一个E_NOTICE级别错误。



可通过(string)或strval()函数来转为字符串。

数组array总是转为字符串“Array”

php4中object总是被转成字符串“Object”，自php5起，适当使用__toString方法。

资源resource总会被转成“Resource id#1”这种结构的字符串。

NULL总是被转成空字符串。

大部分的php值可转为string永久保存，即串行化，可用函数serialize()实现。

**字符串类型详解**

php中的string的实现方式是一个由字节组成的数组再加上一个整数指明缓冲区长度。

字符串类型的此特性解释了为什么php没有单独的byte类型（已经用字符串来代替了）。



**Array数组**

php中的数组实际上是一个有序映射。

key会有如下的强制转化：

- 含合法整型值的字符串会转为整型，键名为"8"则转为8，“08”不会转化
- 浮点数转为整型，如键名8.7则实际存储8
- 布尔值转为整型，true为0，false为0
- Null转为空字符串
- 数组和对象不能用做键名（Illegal offset type）

php5.4 起可以直接对函数或方法调用的结果进行数组解引用，getArray()[1]。

unset() 函数允许删除数组中的某个键，如需删除后重建索引，则可用array_values()函数。



**Object对象**

对象转为对象不会有任何变化，如其他类型的值转为对象，将会创建一个内置类stdClass的实例。如果该值为NULL，则新的实例为空。

```php
$obj = (object)['1'=>'foo'];
var_dump(isset($obj->{'1'})); //php7.2后输出true，之前版本输出false
var_dump(key($obj)); //7.2后输出string 1，之前版本输出int 1
```

对于其他值，会包含进成员变量名scalar

```php
$obj = (object)'hello'; echo $obj->scalar;
```



**Resource资源类型**

资源resource是一种特殊变量，保存了到外部资源的一个引用。资源是通过专门的函数来建立和使用的。



**NULL**

下列情况下一个变量被认为是NULL：

- 被赋值为NULL
- 尚未被赋值
- 被unset()

使用(unset)$var将一个变量转为null将不会删除该变量或unset其值，仅是返回NULL值而已。



**Callback / Callable类型**

自php5.4起可用callable类型指定回调类型callback。一些函数如call_user_func()或usort()可以接受用户自定义的回调函数作为参数。回调函数不止可以是简单函数，还可以是对象的方法，包括静态类方法。

```php
//e.g. callback function
function my_callback_function(){
    echo "hello world";
}
//callback method
class MyClass{
    static function myCallbackMethod(){
        echo "hello world";
    }
}

//simple callback
call_user_func('my_callback_function');
//static class method call
call_user_func(array("MyClass", 'myCallbackMethod'));
//object method call
$obj = new MyClass();
call_user_func(array($obj, 'myCallbackMethod'));
//static class method call
call_user_func("MyClass::myCallbackMethod");
//relative static class method call
class A{
    public static function who(){
        echo "A\n";
    }
}
class B extends A{
    public static function who(){
        echo "B\n";
    }
}
call_user_func(array("B", "parent::who")); //A
//objects implementing __invoke can be used as callables
class C{
    public function __invoke($name){
        echo "hello ", $name, "\n";
    }
}
$c = new C();
call_user_func($c, 'PHP');

//our closure
$double = function($a){
    return $a * 2;
}
//this is our range of numbers
$numbers = range(1, 5);
//use the closure as a callback here to double the size of each element in our range
$newNums = array_map($double, $numbers);
print implode(' ', $newNums);
```

> 在函数中注册有多个回调内容时（如使用call_user_func()与call_user_func_array()），如在前一个回调中有未捕获的异常，其后的将不再调用。



**伪类型**

伪类型（pesudo-types）是php文档里用于指示参数可以使用的类型和值。它们不是php语言里的原生类型，所以不能把伪类型用于自定义函数里的类型约束（typehint）。

**mixed**

说明一个参数可以接受多种不同的类型，如gettype()可以接受所有的php类型，str_replace()可以接受字符串和数组。

**number**

可以是integer或float

**callback**

**array|object**

既可以是array也可以是object

**void**

void作为返回类型意味着函数的返回值是无用的，void作为参数列表意味着函数不接受任何参数。

**...**

在函数原型中，$...表示等等的意思。当一个函数可以接受任意个参数时使用此变量名。



**类型转换的判别**

允许的强制转换有：

- (int) (integer)
- (bool) (boolean)
- (float) (double) (real)
- (string)
- (array)
- (object)
- (unset) 转化为NULL
- (binary)转换和b前缀转换支持php5.2.1新增



### 变量

------

变量名区分大小写。

可以使用中文。

只有有名字的变量才可以引用赋值。

**预定义变量**

自动全局变量 autoglobals

超全局变量 superglobals

php中全局变量在函数中使用时必须声明为global。

```php
$a = 1;
$b = 2;
function test(){
    global $b;
    echo $a;
    $b += 1; //也可使用$_GLOBALS['b']
}
test(); //无输出，echo引用的是局部变量$a
```

**静态变量**

```php
function test(){
    static $a = 0; //仅第一次调用时被初始化
    echo $a; //逐渐递加
    $a++;
}
```

静态声明是在编译时解析的，如果在声明中用表达式的结果对其赋值会导致解析错误。

**可变变量**

```php
$a = 'hello';
$$a = 'world'; //$hello = 'world'
```

变量名中的点和空格被转换成下划线，如:

input name='a.b' 变成 $_REQUEST['a_b']



### 常量

不要使用双下划线来定义常量，避免未来php定义同名的魔术常量。

常量一旦被定义，就不能再改变或取消定义。

常量只能包含标量数据（boolean integer float string）

> 常量和（全局）变量在不同的名字空间中，即：TRUE和$TRUE是不同的。

常量和变量不同：

- 常量前面没有$
- 常量只能通过define()定义
- 常量可以不用理会变量的作用域而在任何地方定义和访问
- 常量一旦定义就不能被重新定义或取消定义
- 常量的值只能是标量

> 和使用define()来定义常量相反的是，使用const关键字定义常量必须处于最顶端的作用区域，因为用此方法是在编译时定义的。这就意味着不能在函数内、循环内以及if语句之内用const来定义常量。

**魔术常量**

几个php的“魔术常量”（不区分大小写）

- __ LINE __ 当前行号
- __ FILE __ 文件的完整路径和文件名，如用在被包含文件中，则返回被包含的文件名
- __ DIR __ 文件所在的目录
- __ FUNCTION __ 函数名称
- __ CLASS __ 类的名称，类名包括其被声明的作用区域（如Foo\Bar），自php5.4起也对trait起作用
- __ TRAIT __ Trait的名字（5.4+），包括其被声明的作用区域
- __ METHOD __ 类的方法名
- __ NAMESPACE __ 当前命名空间的名称，此常量是在编译时定义的，php5.3+



### 表达式



### 运算符

**运算符优先级**

![1661758568999](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1661758568999.png)

**算术运算符**

除法运算符总是返回浮点数，只有在下列情况例外：两个操作数都是整数（或字符串转换成的整数）并且正好能整除，这时它返回一个整数。

取模运算符的操作数在运算之前都会转成整数（除去小数部分），取模运算符的结果和被除数的符号相同（即$a % $b的结果同$a）。

**赋值运算符**

赋值运算将原变量的值拷贝到新变量中（传值赋值），所以改变其中一个并不影响另一个。这也适用于在密集循环中拷贝一些值例如大数组。

> 在php中普通的传值赋值行为有个例外就是碰到对象object时，在php5中是以引用赋值的，除非明确使用了clone关键字来拷贝。

php支持引用赋值，使用$var = &$other;语法，引用赋值意味着两个变量指向了同一个数据，没有拷贝任何东西。

> 自php5起，new运算符自动返回一个引用，因此再对new的结果进行引用赋值在php5.3以及以后版本中会发出一条E_DEPRECATED错误信息，在之前版本会发出一条E_STRICT错误信息。

**位运算符**

位运算符允许对整型数中指定的位进行求值和操作。

![1661760114050](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1661760114050.png)

位移在php中是数学运算，向任何方向移出去的位都被丢弃。左移时右侧以零填充，符号位被移走意味着正负号不被保留。右移时左侧以符号位填充，意味着正负号被保留。

> php的ini设定error_reporting使用了按位的值，提供了关闭某个位的真实例子。
>
> 要显示除了提示级别之外的所有错误：E_ALL & ~E_NOTICE；
>
> 还可用按位异或来取得只在其中一个值中设定了的位：E_ALL ^ E_NOTICE；
>
> 只显示错误和可恢复错误的方法：E_ERROR | E_RECOVERABLE_ERROR；

**比较运算符**

$a<=>$b 太空船运算符（组合比较符）：当$a小于、等于、大于$b时，分别返回一个小于、等于、大于0的integer值，php7+。

$a ?? $b ?? $c Null合并操作符：从左往右第一个存在且不为NULL的操作数，如果都没有定义且部位NULL，则返回NULL，php7+。

> 如果比较一个数字和字符串或者比较涉及到数字内容的字符串，则字符串会被转换位数值并且比较按照数值来进行。此规则也适用于switch语句。当使用===或!==进行比较时则不进行类型转换，因为此时类型和数值都要比对。

```php
var_dump(0 == 'a'); //0==0 true
var_dump("1" == "01"); //1==1 true
var_dump("10" == "1e1"); //10==10 true

switch("a"){
    case 0:
        echo "0";
        break;
    case "a": //never reached because "a" is already matched with 0
        echo "a";
        break;
}

//objects
$a = (object)["a"=>"b"];
$b = (object)["a"=>"b"];
echo $a<=>$b; //0
//only values are compared
$a = (object)['a'=>'b'];
$b = (object)['b'=>'b'];
echo $a<=>$b; //1
```

![1661762652011](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1661762652011.png)

```php
//数组是用标准比较运算符这样比较的
function standard_array_compare($op1, $op2){
    if(count($a) < count($b)){
        return -1;
    }elseif(count($op1) > count($op2)){
        return 1;
    }
    foreach($op1 as $key=>$val){
        if(!array_key_exists($key, $op2)){
            return null; //uncomparable
        }elseif($val < $op2[$key]){
            return -1;
        }elseif($val > $op2[$key]){
            return 1;
        }
    }
    return 0;
}
```

> 比较浮点数：由于浮点数的内部表达方式，不应比较两个浮点数float是否相等。

> 三元运算符是个语句，因此其求值不是变量，而是语句的结果。如果想通过引用返回一个变量这点就很重要。在一个通过引用返回的函数中语句 return $var==42?$a:$b; 将不起作用，以后的php版本会为此发出一个警告。

**错误控制运算符**

将@符号放置到一个php表达式前，该表达式可能产生的任何错误信息都被忽略掉。

如果用set_error_handler()设定了自定义的错误处理函数，仍然会被调用，但是此错误处理函数可以（并且也应该）调用error_reporting()，而该函数在输出语句前有@时将返回0。

如果激活了track_errors特性，表达式所产生的任何错误信息都被存放在变量$php_errormsg中。此变量在每次出错时都会被覆盖，所以如果想用它的话就要尽早检查。

```php
$myFile = @file("non_exsitent_file") or die("failed opening file: error was '$php_errormsg'");
```

> 目前@错误控制运算符前缀甚至使导致脚本终止的严重错误的错误报告也失效，这意味着如果在某个不存在或者敲错了字母的函数调用前使用了@来抑制错误信息，那脚本会没有任何迹象显示原因而死在那里。

**执行运算符**

```php
$output = `ls -al`;
echo "<pre>$output</pre>";
```

> 反引号运算符在激活了安全模式或者关闭了shell_exec()时是无效的。反引号不能在双引号字符串中使用。

**递增/递减运算符**

> 递增或递减运算符不会影响布尔值，递减null值也没效果，递增null的结果为1

在处理字符变量的算术运算时，PHP沿用了perl的习惯，如在perl中$a='Z';$a++变成'AA'，而c中a='Z';a++将把a变成'['（Z的ascii值是90，[的ASCII值是91）。

字符变量只能递增不能递减，并且只支持纯字母（a-z A-Z）。递增或递减其他字符变量则无效，原字符串没有变化。

```php
$s = 'W';
for($n=0; $n<6; $n++){
    echo ++$s.PHP_EOL; //X Y Z AA AB AC
}

$d = 'A8';
for($n=0; $n<6; $n++){
    echo ++$d . PHP_EOL; //A9 B0 B1 B2 B3 B4
}

$d = 'A08';
for($n=0; $n<6; $n++){
    echo ++$d . PHP_EOL; //A09 A10 A11 A12 A13 A14
}
```

**逻辑运算符**

逻辑与：and &&

逻辑或：or ||

逻辑异或：xor 如果两者任一为true，但不同时是。

逻辑非：!

> 与 和 或 有两种不同形式运算符的原因是它们的优先级不同。||比or优先级高；&&同。

```php
$e = false || true; // $e = (false || true)
$f = false or true; // ($f = false) or true
```

**字符串运算符**

连接运算符“.”

**数组运算符**

![1661938799029](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1661938799029.png)

```php
$a = ['a'=>'apple', 'b'=>'banana'];
$b = ['a'=>'pear', 'b'=>'strawberry', 'c'=>'cherry'];
$c = $a + $b; //union of $a and $b apple banana cherry
$d = $b + $a; //union of $b and $a pear strawberry cherry

$arr1 = ['apple', 'banana'];
$arr2 = [1=>'banana', '0'=>'apple'];
var_dump($a == $b); //true
var_dump($a === $b); //false
```

**类型运算符**

instanceof用于确定一个php变量是否属于某一类class的实例。

```php
class MyClass
{
}

class NotMyClass
{
}

$a = new MyClass;
var_dump($a instanceof MyClass); //true
var_dump($a instanceof NotMyClass); //false

//instanceof 也可用来确定一个变量是不是继承自某一父类的子类的实例
class ParentClass{}
class MyClass extends ParentClass{}
$a = new MyClass;
var_dump($a instanceof MyClass); //true
var_dump($a instanceof ParentClass); //true

//检查对象是不是某个类的实例
class MyClass{}
$a = new MyClass;
var_dump(!($a instanceof stdClass));

//也可用于确定一个变量是不是实现了某个接口的对象的实例
interface MyInterface{}
class MyClass implements MyInterface{}
$a = new MyClass;
var_dump($a instanceof MyClass); //true
var_dump($a instanceof MyInterface); //true
$b = new MyClass;
$c = 'MyClass';
$d = 'NotMyClass';
var_dump($a instanceof $b); //true
var_dump($a instanceof $c); //true
var_dump($a instanceof $d); //false
```

> 如果被检测的变量不是对象，instanceof并不发出任何错误信息而是返回false。不允许用来检测常量。
>

```php
$a = 1;
$b = null;
$c = imagecreate(5, 5);
var_dump($a instanceof stdClass); //false
var_dump($b instanceof stdClass); //false
var_dump($c instanceof stdClass); //false
var_dump(false instanceof stdClass); //php fatal error:instanceof expects an object instance, constant given
```

在php5.1之前，如果要检查的类名称不存在，instanceof会调用__autoload()。另外，如果该类没有被装载则会产生一个致命错误。可以通过使用动态类引用或用一个包类名的字符串变量来避开这种问题：

```php
$d = 'NotMyClass';
var_dump($a instanceof $d); //false
```



### 流程控制

switch和第一个case之间的任何输出（含空格）将导致语法错误。

```php
//错误
<?php switch($foo): ?>
    <?php case 1: ?> //行首不可存在空格
    ...
<?php endswitch; ?>
```

```php
//do{}while(0);
//一段代码想要执行到某个条件后面的代码不继续执行，保证只执行一次，可以使用break跳出循环，后续语句不再执行
do{
    if($i<5){
        echo "i is not big enough";
        break;
    }
    $i *= $factor;
    if($i < $minimun_limit){
        break;
    }
    echo "i is ok";
    /**process i**/
}while(0);
```

**for**

for(expr1; expr2; expr3){ statement }

expr1在循环开始前无条件执行一次；

expr2在每次循环开始前求知，如值为false则终止；

expr3在每次循环后执行；

每个表达式都可以为空或包括逗号分隔的多个表达式。表达式expr2中，所有用逗号分隔的表达式都会计算，但只取最后一个结果。expr2为空意味着将无限循环下去。

