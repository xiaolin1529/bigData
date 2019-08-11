# scala课程（上）

- scala 介绍
- 基础语法
- 条件控制与循环
- 函数入门
- 数组操作
- map与tuple

**scala是一门多范式的编程语言，集成了面向对象编程和函数式编程的各种特性。**

**函数式编程是一种典范，将电脑的运算视为函数的运算。与过程化编程相比，函数式编程的函数计算可随时调用，函数式编程中，函数与类，对象一样都是一等公民。**

**scala命令行**

 	也称REPL Read（取值），Evaluation（求值），Print（打印），Loop（循环）

## 基础语法

	### 声明变量

val 不可变变量，val声明的常量，值不能被修改

var 可以随时修改var声明的变量的值

*无论声明val变量还是声明var变量，都可以手动指定其类型，如果不指定的话，scala会自动根据值，进行类型的判断。*

### 数据类型与操作符。 ###

#### 基本数据类型 ####

![img](https://img-blog.csdn.net/20180529091555973)

Byte、Char、Int、Long、Float、Double、Boolean

scala没有基本数据类型与保证类型的概念，统一都是类。scala自己会负责基本数据类型和引用类型的转换操作。

使用以上类型就可以调用大量函数。

#### 加强版类型 ####

scala使用很多加强类给数据类型增加了上百种增强的功能函数 例如String通过StringOps类增强了大量的函数。scala还提供了Rich'Int、RichDouble、RichChar等类型，RichInt提供了to函数，1.to(10),此处int先隐式转换为RichInt，然后再调用其to函数。

#### 基本操作符 ####

scala的算术操作符基本和java的算数操作符基本一样。

**注意scala没有提供++、--操作符，只能使用count+=1等格式 **

#### 函数调用与apply() ####

在scala中，函数可以直接由对象调用，如import scala.math_ min(3,Pi)，不同的一点是，如果调用函数时，不需要传递参数，则scala允许调用函数时省略括号，如“Hello”.distinct

apply函数

​	scala中的apply函数是一种函数，在scala中，可以声明apply函数。而使用 类名（） 的形式，其实是类名.apply() 的一种缩写

例如： “Hello”（6） 因为在StringOps类中有def apply（n：Int）：Char的函数，所以“Hello”（6）实际上是“Hello”.apply(6)的缩写。