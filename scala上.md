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

## 条件控制与循环 ##

#### if表达式 ####

- 在scala中if表达式是有返回值的，为if或者else最后一行语句返回的值。

  ` val isAdult = if(age > 18) 1 else 0 //isAdult为1或0` 

- if表达式的类型推断

   如果if和else字句的值类型不同，scala会自动进行判断，取两个类型的公共父类型

  ​	如：if (age >18) 1 else 0,表达式返回值的类型是Int，因为1和0都是Int

  ​	if （age> 18） "adult" else 0,此时if和else的返回值分别是String和Int，表达式返回值类型为Any，Any是String和Int的公共父类型。

  ​	如果if后面没有else，默认else的值是Unit，也用()表示，类似java中的void或者null。

  ​		如 if（age>18） "adult"等价于 if (age >18 ) "adult" else ().此时表达式返回值为AnyVal。

- 如何在Scala repl中执行多行代码

  ​	：paste进入编辑模式，ctrl-D完成输入

  

#### 语句终结符、块表达式 ####

 - 默认情况下，scala不需要语句终结符，默认将一行作为一个语句。

 - 如果一行要发多条语句，必须使用语句终结符。scala使用分号作为语句终结符。

   ` var a,b,c=0; if (a < 10) { b= b+1; c+ c+1}`

- 块表达式，在scala里面使用{},其中可以包含多条语句，最后一个语句的值就是块表达式的返回值。

  *scala表达式返回值不需要return*

#### 输入与输出 ####

- print打印时不加换行符，println打印时会加一个换行符
- readLine允许我们从控制台读取用户输入的数据，类似java中的System.in和Scanner的作用
- Console.readLine()不建议使用
- 建议使用java中的BufferReader中的readLine

####  循环 ####

- while循环，用法和java类似

  ​	` var n=10;while(n>0){println(n);n-=1}`

- for循环

  ​	` var n=10;for (i<-1 to n) println(i)`//打印1到n

  ​	`for(i<-1 until n) println(i) `//打印1到n-1

  ​	` for(i<-"Hello,world")println(i)`//打印每个char

- 高级for循环

  - if守卫：取偶数

    ` for(i<-1 to 10 if i %2 == 0)println(i)`

  - for推导式：构造集合

    ` for(i<- 1 to 10) yield i* 2`//生成一个indexedSeq

## 函数入门