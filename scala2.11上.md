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

## 函数定义与调用



  - 在scala中定义函数时，需要定义函数的函数名、参数、函数体

-  def 函数名(变量名:变量类型):返回值类型={表达式语句}

  ​	如果函数体只有一行代码，则{}可以省略不写。代码块中最好一行的返回值就是整个函数的返回值，scala不使用return返回 值的。



## 函数参数

- 有时，我们调用某些函数时，不希望给出参数的具体值，而希望使用参数自身的默认值，此时需要在定义函数时使用默认参数。

  ​	 def 函数名(变量名1:变量类型1，变量名2:变量类型2 = "mid",变量名2:变量类型2 = "last"):返回值类型={表达式语句}

  ​	如果给出的参数不够，则会从左往右依次应用参数。

  

## 带名参数

 - 在调用函数时，也可以不按照函数定义的参数顺序来传递参数，而是使用带名参数的方式来传递。
 - 调用函数时还可以混合使用未命名参数和带名参数，但是未命名参数必须排在带名参数前面
 - 建议在参数多的时候，尽量使用带名参数的形式，这样不容易出错

## 变长参数

 - 在scala中，有时我们需要将函数定义为参数可变的形式，此时可以使用变长参数定义函数

   ```scala
   def sum(nums:Int*)={
       var res=0;
       for (num <- nums)
       	res += num
       res
   }
   sum(1,2,3,4,5)
   ```

## 过程

 - 在scala中，定义函数时，如果函数体直接包裹在了{}里面，而没有使用 = 连接，则函数的返回值类型就是Unit，这样的函数就称之为过程。

   ​		` def sayHello(name:String){"hello, "+name} `

 - 过程通常用于不需要返回值的函数。

 - 过程还有一种写法，就是将函数的返回值类型定义为Unit。

     ` def sayHello(name:Sring):Unit = "Hello, "+name`

## lazy值

 - 在scala中，提供了lazy值的特性，就是说如果将一个变量声明为lazy，则只有在第一次使用该变量时，变量对应的表达式才会发生计算。这种特性对于特别耗时的计算操作特别有用，比如打开问及那进行IO、网络IO等。



## 异常try-catch


```scala
try {
  throw new IllegalArgumentException("x should not be negative")
} catch {

  case e1: IllegalArgumentException => println("illegal argument")
  case e2: IOException => println("io exception")

  case _: IllegalArgumentException => println("Illegal Argument!")
} finally {
  print("release resources!")
}



```

# 数组操作

 - 在scala中，Array和Java中的数组类似，都是长度不可变的。

   ​	` val a = new Array[Int](10)`

- 可以直接使用Array() 创建数组，元素类型自动推断

  ​	

## ArrayBuffer

 - ArrayBuffer是scala里面长度可变的集合类

 - 使用前需要先导入 import scala.collection.mutable.ArrayBuffer

 - ArrayBuffer 也支持直接创建并初始化 如ArrayBuffer(1,2,3)

   ``` scala
   val b= new ArrayBuffer(1,2,3) //创建ArrayBuffer
   b +=(4,5) //添加一个或多个元素
   b ++=Array(6,7,8)	//添加其他集合中所有元素
   b.trimEnd(3)	//从尾部截断指定个数的元素
   b.insert(5,6)	//向指定位置插入元素，位置后面元素向后移动
   b.remove(1)	//移除角标为1的元素
   b.remove(1,3)	//移除角标1到3的元素
   ```

- Array和ArrayBuffer可以互相进行转换

  ​	b.toArray

  ​	arr.toBuffer

## 遍历Array和ArrayBuffer

 - 使用for循环和util遍历Array/ArrayBuffer
    - `for(i<-0 util b.length)println(b(i))`
    - 从尾部遍历 `for(i<- (0 util b.length).reverse )println(b(i))`
- 使用增强for循环遍历Array/ArrayBuffer
  - `for(e<- b)println(e)`

## 数组常见操作

 - 求和
    - `val a=Array(1,2,3); val sum=a.sum`
- 求最大值
  - `val max = a.max`
- 排序
  - `scala.util.Sorting.quickSort(a)`
- 获取数组中所有的元素
  - `a.mkString`
  - `a.mkString(",")` 生成字符串（String = 1,2,3,4,5）
  - `a.mkString("[",",","]")` String = [1,2,3,4,5]
- 使用yield和函数时编程转换数组
  - 对Array进行转换，获取的还是Array
    - `val a=Array(1,2,3);val a2 = for(ele<- a)yield ele * ele`
  - 对ArrayBuffer进行转换，获取的还是ArrayBuffer
    - `val b =ArrayBuferr(1,2,3,4);val b2=for(ele<-b) yield ele * ele`
  - 结合if守卫，仅转换需要的元素
    - `val b3 = for (ele <-b if ele %2 ==0 )yield ele*ele`
- 使用函数式编程转换数组
  - `a.filter(_%2==0).map(2*_)` 【推荐使用这种形式】
  - `a.filter {_%2==0} map {2*_} ` [中间需要复杂逻辑转换的时候使用这种格式] 

# Map和Tuple

## 创建map

 - 创建一个不可变的Map
    - `val ages = Map("li"->12,"sa"->12)` 不可修改
- 创建一个可变的Map
  - `val ages = scala.collection.mutable.Map("li"->12,"sa"->3)` 可以修改、添加元素
- 另一种方式定义Map元素
  - `val ages = Map(("li"->12),("sa"->2))` 不可修改
- 创建一个空的HashMap
  - `val ages = new scala.collection.mutable.HashMap[String,Int]()`

## 操作 map-查询

 - 获取指定key对应的value，如果key不存在，会报错
    - `val age = ages("li")`
- 使用contains函数检查key是否存在
  - `val age = if(ages.contains("li"))ages("li") else 0`
- 改良版：getOrElse函数（建议使用）
  - `val age = ages.getOrElse("li",0)` 

## 操作 map-修改

 - 更新map元素 `ages("li")=18`
 - 增加多个元素 `ages += ("ha"->23,"adw"->89)`
 - 移除元素    ` ages -= "ha" `
 - 不可变map的操作（，都是返回了一个新的map）
    - 更新不可变的map，其实是拼接不可变map，产生了新的map
       - `val ages2 = ages + ("lq"->1)`
   - 移除不可变的元素
     - `val ages3 = ages- "Tom"`

## 操作 map-遍历

 - 遍历map的entrySet
    - `for((key,value)<-ages)println(key+" "+value)`
- 遍历map的key
  - ` for(key<-ages.keySet)println(key)` 
- 遍历map的value
  - `for(value<-ages.values)println(value)` 
- 生成新map，反转key和value
  - `for((key,value)<-ages)yield(value,key) ` 

## SortMap和LinkedHashMap

 - SortedMap可以自动对Map的key进行排序【针对有序的map场景】
    - `val ages = scala.collection.immutable.SortedMap("li"->12,"zhang"->34)` 
- LinkedHashMap可以记住插入entry的顺序
  - `val ages = new scala.collection.mutable.LinkedHashMap[String, Int]()` Map内元素顺序为元素插入顺序

## tuple

 - 与Array一样，元组也是不可变的，但是与列表不同的是元组可以包含不同类型的元素

    - tuple元素角标从1开始
    - `val t = (1,3,14,"Fred")`
    - t._1 获取tuple中的第一个元素

- 目前scala支持的元组最大长度是22.对于更大长度，可以使用集合。

- zip操作

  - ```scala
    val names = Array("li","sa")
    val ages =Array(12,45)
    val nameAges = names.zip(ages)
    for((name,age)<-nameAges)println(name+":"+age)
    ```



# END

