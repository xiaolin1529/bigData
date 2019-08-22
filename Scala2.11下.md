# Scala2.11下

## 面向对象编程

### 类

 - 定义类，包含field以及方法

    - ```scala
      class HelloWorld {
          private var name = "sacala"
          def sayHello(){println("hello, "+name)}
          def getName =name
      }
      ```

- 创建类的方法，并调用其方法

    - ` val helloWorld = new HelloWorld()` //方法也可以不加括号，如果定义方法时不带括号，则调用方式时也不能带括号
    - `helloWrold.sayHelo()`
    - `println(helloWorld.getName)`

- getter和setter

    - 定义不带priviate 的var field ，此时scala生成的面向JVM的类时，会定义为private的field字段，并提供public的getter和setter方法

    - 如果使用private修饰field，则生成的getter和setter也是private的。

    - 如果定义val field,则只会生成getter方法

    - 如果不希望生成setter和getter方法则及那个field声明为private[this]

- 如果只是希望拥有简单的getter和setter方法，那么按照scala的语法规则选择合适的修饰符即可（var，val，private，private[this]）

- 如果希望自己能对getter和setter进行控制，则可以自定义getter和setter方法。

- 自定义setter方法的时候一定要注意scala的语法限制，函数名、=、参数间不能有空格

- private[this]

    - private修饰field，field为类私有的，在类的方法中可以直接访问类的其他对象的private field
    - 这种情况下，如果不希望field被其他对象访问到，那么可以使用private[this]，意味着对象私有的field，只有本对象内可以访问到。

- 可以给field添加@BeanProperty 注解 生成java风格的getter和setter方法。

- 构造函数-辅助constructor

    - scala中可以给类定义多个辅助constructor，类似java中的构造函数重载
    - 辅助构constructor之间可以互相调用，而且必须第一行调用主constructor

- 构造函数-主constructor

    - scala中，主constructor是与类名放在一起的，与java不同
        - 注意：在类中，没有定在任何方法或是代码块，就是主constructor的代码
    - 主constructor可以通过使用默认参数，来给参数默认赋值。
#### 对象-object

   - object 相当于class的单个实例，通常在里面放一些静态的field或者method
   - 第一次调用object的方法时，就会执行object的constructor，也就是object内部不在method中的代码，但是object不能定义接受参数的constructor
   -  object的constructor指挥在其第一次被调用时执行一次，以后再次调用就不会再次执行constructor了
   - object通常用于作为单例模式的实现，或者放class的静态成员，比如工具方法
   - 对象可以直接使用不能new

#### 伴生对象

 - 如果有一个class，还有一个与class同名的object，那么就称这个object是class的伴生对象，class是object的伴生类
 - 伴生类与伴生对象必须存放在一个.scala文件之中。
 - 伴生类和伴生对象，最大的特点就在于互相可以访问private field。

#### object继承抽象类

 - object的功能和class类似，除了不能定义定义接收参数的constructor
 - object也可以继承抽象类，并覆盖抽象类中的方法。

#### apply方法

 - 通常在伴生对象中实现apply方法，并在其中实现构造伴生类的对象。
 - 创建对象时，通常不会使用new class方式，而是使用class()的方式，隐式调用伴生对象的apply方法，这样会让对象创建更加简洁。

#### 区分

 - new String("abc")(1)
    - 使用new关键字表示创建的是一个类的对象
    - 第一个括号表示的是主构造函数，第二个参数表示的是apply函数
- "abc"（1）
  - 如果不带new表示使用的是对应类的伴生对象，此时后面的第一个括号表示的是apply函数。

#### main方法

 - 和java一样，在scala中有个有main入口类
    - scala中main方法定义为def main(args:Array[String]),而且必须定义在object中



### Trait 【接口】

 - scala中的trait是一种特殊的概念。
    - 可以将trait作为接口来使用，此时的traint就与java中的接口类似
    - trait中可以定义抽象方法，就与抽象类中的抽象方法一样，但是不给出方法的具体实现
    - 类可以使用extends关键字继承trait，在scala里面无论是继承还是trait，通用都是extends
    - 类继承trait后，必须实现其中的抽象方法，实现时不需要使用override关键字
    - scala不支持类进行多继承，但是支持多重继承trait，使用with关键字即可。



## 函数式编程 【重要】

​	scala是一门既面向对象又面向过程的语言。因此在scala中有非常好的面向对象的特性，可以使用scala基于面向对象的思想开发大型复杂的系统和工程；而且scala也面向过程，因此scala中有函数的概念。在scala中，函数与类、对象一样都是一等公民。

java是完全面向对象的编程语言，没有任何面向过程编程语言的特性，因此java中的一等公民是类和对象，而且只有方法的概念。Java中的方法是绝对不可能脱离类和对象独立存在的。

- 1将函数赋值给变量

  - scala中的函数是一等公民，可以独立定义，独立存在，而且可以直接将函数作为值赋值给变量。

    - scala的语法规定，将函数赋值给变量时，必须在函数后面加上空格和下划线

    - ```scala
      def sayHello(name:String){println("Hello, "+name)}
      val sayHelloFunc=sayHello _
      sayHelloFunc("scala")
      ```

- 2 匿名函数

  - 匿名函数语法规则：(参数名:参数类型)=>函数体
  - 如果函数体有多行代码需要添加{}

- 3 高阶函数

  - scala中函数是一等公民，可以将某个函数作为参数传入其他函数。
  - 接受其他函数作为指定函数的参数，也被称作高阶函数（higher-order function）
  - 高阶函数的另一个功能是将函数作为返回值。

- 4 高阶函数的类型推断

  - 高阶函数可以自动推断出参数类型，而不需要写明类型
    - 对于只有一个参数的参数，还可以省去其小括号。

- 5 scala的常用高阶函数

  - map：对传入的每个元素都进行映射，返回一个处理后的元素。
  - foreach:对传入的每个元素都进行处理，但是没有返回值。
  - filter：对传入的每个元素都进行条件判断，如果对元素返回true，则保留该元素，否则过滤掉该元素。

  - reduceLeft：从左侧元素开始，进行reduce操作。

- 6 闭包

  - 函数在变量不处于其有效作用域时，还能够对变量进行访问，即为闭包。

- 7 Currying函数

  - Curring函数指的是，将原来接受两个参数的一个函数，转换为两个函数，第一个函数接受原来函数的第一个参数，返回第二个函数去接受第二个参数。

- 8 return

  - scala中，不需要使用return来返回函数的值，函数最后一行语句的值，就是函数的返回值。在scala中return用于在匿名函数中返回 值给包含匿名函数的带名函数，并作为带名函数的返回值。

  - 使用return的匿名函数，是必须给出返回类型的，否则无法通过编译

    ```scala
    def greeting(name:String)={
        def sayHello(name:String):String={
            return "Hello,"+name
        }
        sayHello(name)
    }
    ```

    

- 9 集合操作

  - scala的集合体系主要包括Iterable、seq、Set、Map。其中Iterable是所有集合trait的根trait。这个结构与Java的集合体系非常相似。

  - scala的集合是分成可变和不可变两类集合的。

    - 可变集合就是说，集合的元素可以动态修改。
      - ![scala可变集合继承层级](https://raw.githubusercontent.com/wangxiaolin123/bigData/master/img/scala可变集合继承层级.png)
    - 不可变集合在初始化之后，就无法修改了。
      - ![scala不可变集合继承层级](https://raw.githubusercontent.com/wangxiaolin123/bigData/master/img/scala不可变集合继承层级.png)
    - 分别对应scala.collection.mutable和scala.collection.immutable两个包

  - Seq下包含了Range、ArrayBuffer、List等trait。

    - 其中Range就代表一个序列，通常可以使用“1 to 10”这种语法来产生一个Range。
    - ArrayBuffer就类似于Java中的ArrayList。

  - List

    - List代表不可变列表
      - 创建List `val list = List(1,2,3)`
      - List的head和tail
        - head代表List的第一个元素list.head
        - tail代表第一个元素之后的所有元素 list.tail
      - list的::
        - ::操作符，可用于将head和tail合并成一个List，`list.head::list.tail`
        - ::这种操作符要清楚，在Spark源码中有体现的。
        - 如果一个List只有一个元素，那么它的head就是这个元素，它的tail就是Nil

  - Set

    - Set代表没有重复元素的集合
      - 将重复元素插入Set是没有作用的。
      - Set元素无序
      - `val s = new scala.collection.mutable.HashSet[Int]();s+=1;s+=2`
    - LinkedHashSet会用一个链表维护插入顺序
      - `val s = new scala.collection.mutable.LinkedHashSet[Int]();i+=1;s+=2`
    - SortedSet自动根据key来进行排序
      - `val s = scala.collection.mutable.SortedSet("c","b","a")`

  - 集合的函数式编程

    - 集合经常配合高阶函数使用
      - map：为List中每个元素都添加一个前缀
        - `List("jessca","Jen","Peter","Jack").map("name is "+ _)`
      - flatMap:将List中的多行句子拆分成单词
        - `List("Hello you","hello me").flatMap(_.split(" "))`
      - zip:对学生姓名和学生成绩进行关联
        - `List("jesska","Jen","Peter").zp(List(100,40,30))`

    



## 模式匹配

 - 模式匹配是Scala中非常有特色，非常强大的一种功能。模式匹配，其实类似于Java中的switch case语法，即对一个值进行条件判断，然后针对不同的条件，进行不同的处理。

 - 但是Scala的模式匹配的功能比Java的switch case 语法的功能要强大地多，Java的switch case语法只能对值进行匹配。但是Scala的模式匹配除了可以对值进行匹配外，还可以对类型进行匹配、对Array和List的元素情况进行匹配、对case class进行匹配、甚至对有值或没值(Option)进行匹配。

 - Scala是没有Java中的switch case语法的，相应的，Scala提供了更加强大的match case 语法，即模式匹配来替代switch case，match case也被称为模式匹配。

 - Scala的match case与Java的switch case最大的不同点在于，Java的switch case仅能匹配变量的值，比如1、2、3等；而Scala的match case可以匹配各种情况，比如变量的类型、集合的元素、有值或无值

 - match case的语法如下

   - 变量match{case 值=> 代码}
   - 如果值为下划线，则代表了不满足以上所有情况下的默认情况如何处理
   - 此外，match case中，只要一个case分支满足并处理了，就不会继续判断下一个case分支了（与Java不同java的switch case）
   - match case语法最基本的应用，就是对变量的值进行模式匹配。

- 对类型进行模式匹配

   - Scala的模式匹配一个强大之处就在于可以直接匹配类型，而不是值，这是Java的switch case做不到的。
   - 其他语法与匹配值其实是一样的，但是匹配类型的话，就是要用 case 变量:类型 => 代码 这种语法，而不是匹配值的 case 值 => 代码 这种语法

- 案例：异常处理

   ```scala
   import java.io._
   
   def processException(e: Exception) {
     e match {
       case e1: IllegalArgumentException => println("you have illegal arguments! exception is: " + e1)
       case e2: FileNotFoundException => println("cannot find the file you need read or write!, exception is: " + e2)
       case e3: IOException => println("you got an error while you were doing IO operation! exception is: " + e3)
       case _: Exception => println("cannot know which exception you have!" )
     }
   }
   
   ```

- case class 与模式匹配

   - Scala中提供了一种特殊的类，用case class进行声明，中文也可以称作样例类。case class其实有点类似与Java中的JavaBean的概念。即只定义field，并且由scala编译时自动提供getter和setter方法，但是没有method
   - case calss的主构造函数接收的参数通常不需要var或val修饰，Scala自动就会使用val修饰(如果你自己使用var修饰，那么还是会按照var来。)
   - Scala自动为case class 定义了伴生对象，也就是object，并且定义了apply()方法，该方法接收主构造函数中相同的参数，并返回case 对象。

- Option与类型匹配

   - scala有一种特殊的类型，叫做Option。Option有两种值，一种时Some，表示有值，一种时None，表示没有值。
   - Option通常会用于模式匹配中，用于判断某个变量是有值还是没值，这比null更加简洁明了。
   - Spark源码中大量使用了Option，比如Some(a),None这种语法，必须看得懂Option模式匹配，才能够读懂Spark源码。

## 类型参数

 - 类型参数-泛型

    - Scala的类型参数和Java泛型一样，也是定义一种类型参数，比如在集合，在类在函数中，定义类型参数，然后就可以保证使用到该类型参数的地方，就肯定也只能是这种类型。从而实现程序更好的健壮性。

    - 泛型可以用在类上或者函数上。

       - 案例1 新生报到，id可能为Int，也可能是String

         ```scala
         class Student[T](val localId: T) {
           def getSchoolId(hukouId: T) = "S-" + hukouId + "-" + localId
         }
         
         val stu= new Student[Int](111)
         
         ```

      - 案例2 卡片售卖机 内容可以是String类型或Int类型

        ```scala
        def getCard[T](content: T) = {
          if(content.isInstanceOf[Int]) "card: 001, " + content
          else if(content.isInstanceOf[String]) "card: this is your card, " + content
          else "card: " + content
        }
        
        getCard[String]("hello world")
        
        
        ```

        

## 隐式转换与隐式参数

### 简介

 - Scala提供的隐式转换和隐式参数功能，是非常有特色的功能。是Java等编程语言所没有的功能。它可以允许你手动指定，将某种类型的对象转换成其他类型的对象。通过这些功能，可以实现非常强大，而且特殊的功能。
 - Scala的隐式转换，其实最核心的就是定义隐式转换函数，即  **implicit conversion function **。定义的隐式转换函数，只要在编写的程序内引入，就会被Scala自动使用。Scala会根据隐式转换函数的签名，在程序中使用到隐式转换函数接收的参数类型定义的对象时，会自动将其传入隐式转换函数，转换为另外一种类型的对象并返回。这就是“隐式转换”
 - 隐式转换函数叫什么名字是无所谓的，因为通常不会由用户手动调用，而是由Scala进行调用。但是如果要使用隐式转换，则需要对隐式转换函数进行导入。因此通常建议将隐式转换函数的名称命名为“one2one”的形式
 - Spark源码中有大量的隐式转换和隐式参数，因此必须熟悉这种语法



### 隐式转换

 - 要实现隐式转换，只要程序可见的范围内定义隐式转换函数即可。Scala会自动使用隐式转换函数。隐式转换函数与普通函数唯一的语法区别就是，要以implicit开头，而且最好要定义函数返回类型

 - 隐式转换非常强大的一个功能，就是可以在不知不觉中加强现有类型的功能。也就是说，可以为某个类定义一个加强版的类，并定义互相之间的隐式转换，从而让源类在使用加强版的方法时，由Scala自动进行隐式转换为加强类，然后再调用该方法

 - 案例：特殊售票窗口（只接受特殊人群，比如学生、老人等）

   ```scala
   class SpecialPerson(val name: String)
   class Student(val name: String)
   class Older(val name: String)
   
   implicit def object2SpecialPerson (obj: Object): SpecialPerson = {
     if (obj.getClass == classOf[Student]) { val stu = obj.asInstanceOf[Student]; new SpecialPerson(stu.name) }
     else if (obj.getClass == classOf[Older]) { val older = obj.asInstanceOf[Older]; new SpecialPerson(older.name) }
     else Nil
   }
   
   var ticketNumber = 0
   def buySpecialTicket(p: SpecialPerson) = {
     ticketNumber += 1
     "T-" + ticketNumber
   }
   ```



### 隐式转换函数作用域与导入

 - Scala默认会使用两种隐式转换，
    - 一种是源类型，或者目标类型的伴生对象内的隐式转换函数；
    - 一种是当前程序作用域内的可以用唯一标识符表示的隐式转换函数
 - 如果隐式转换函数不在上述两种情况下的话，那么就必须手动使用import语法引入某个包下的隐式转换函数，比如import test._。通常建议，仅仅在需要进行隐式转换的地方，比如某个函数或者方法内，用import导入隐式转换函数，这样可以缩小隐式转换函数的作用域，避免不需要的隐式转换