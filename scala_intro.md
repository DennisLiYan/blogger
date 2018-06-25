 10分钟简介Scala
============
因为最近项目的需要，构建一个项目组的数据平台用于搜集数据所以接触到了Scala这门语言。因此想写文章来总结自己最近看到的知识点，这篇文章会简介Scala这门语言中的一些与众不同的特性，能让刚接触这门语言的童鞋快速的对Scala有一个大体了解。

目录:
- Scala简介
- Scala中的变量与函数
- Scala的类定义相关

### Scala简介
Scala是一门非常灵活的编程语言，主要的特点是支持函数式的编程。在使用Scala的过程中支持嵌套调用Java代码，Scala本身会被编译为Java bytecode，可以被JVM执行。

### Scala中的变量与函数 
#### 变量定义
在Scala中可以使用val和var关键字来申明变量。val表名变量是immutable类型，有点类似于C++的常量只能在初始化的时候赋值一次，var关键字则表示mutable类型可以被多次赋值。最后面一种def关键字申明的是function value，同样是immutable类型，并且只用在使用到的时候才会正在分配该变量(lazily evaluated)。可以[参考链接](https://stackoverflow.com/questions/4437373/use-of-def-val-and-var-in-scala)
```scala
val k = 0
var n = 1
def m = 6
```
#### 函数定义
Scala中的函数定义方式真的是五花八门多种多样，下面列举一些常用的函数定义语法:
```scala
def f(x: Int) = x+1;   def g1(x: Int) = x+1
def g2(x: Int) = {x+1}; def h1(x: Int) = (x + 1)
def fun(x: Int): Int = 
  x +
  1
def printTest(x: Unit): Unit = {
  println(x)
  // return value is Unit
  x
}
// 下面是一个匿名函数的定义
(x: Int) ⇒ x + 1
// 等价于
{def f(x: Int) = x + 1; f _}
```
在Scala中函数都是有返回类型的，而且不需要使用关键字return，默认的返回值是Unit。最后一个函数是匿名函数，为了避免每次使用都需要命名。(x1: T1, ..., xn: Tn)⇒E 等价于 { def f (x1: T1, ..., xn: Tn) = E ; f _ }

#### 高阶函数
在Scala中函数的参数同样可以是函数，这样以其他函数作为参数的函数我们称之为高阶函数。
```scala
// 参数f为函数类型的变量,Test为高阶函数，函数会把一个List中的元素按照f来做映射
def Test(arr: List[Int], f: (Int) => Int): List[Int] = {
  arr.map(f)
}
Test(List(1,2,3,4,5), (x: Int) => x*x)
```
这里要注意区分参数为函数类型的申明以及匿名函数的具体定义。
#### 尾递归(Tail Recursion)
函数式编程的特性之一就是递归非常多，比如我们以下求阶乘的代码:
```scala
def factorial(n: Int): Int =
  if (n == 0) 1 else n * factorial(n - 1)
/* 
函数的调用方式如下
factorial(4)
if (4 == 0) 1 else 4 * factorial(4 - 1)
4 * factorial(3)
4 * (3 * factorial(2))
4 * (3 * (2 * factorial(1)))
4 * (3 * (2 * (1 * factorial(0)))
4 * (3 * (2 * (1 * 1)))
24
*/
```
可以发现上面这个函数会嵌到很多栈空间，Scala针对这种情况提出了尾递归的概念，下面我们换一种函数实现方式。
```scala
// 
def factorial(n: Int): Int = {
  @tailrec
  def iter(x: Int, result: Int): Int =
    if (x == 0) result
    else iter(x - 1 , result * x) 
    iter(n, 1)
}
// factorial(3) shouldBe 6
/*
函数的调用方式如下
factorial(3)
iter(3,1)
if(3==0) 1 else iter(2,3*1)
if(2==0) 3 else iter(1,3*2)
if(1==0) 6 else iter(0,3*2*1)
if(0==0) 6 else iter(...)
return 6
*/
```
这两个版本最大的区别在于第二种实现方式可以做到多次递归调用的时候不需要保存之前的调用堆栈，因为result作为一个参数传递下去了。这样在Scala的编译器会自动优化，避免过多的堆栈调用开销，这种优化就叫做尾递归。所以在实现递归函数时最好实现能让编译器优化的尾递归版本。 例子来源[传送门](https://www.scala-exercises.org/scala_tutorial/tail_recursion)
#### 柯里化(Currying)
Currying也是一个Scala的语言特性，简单来说就是Scala函数的返回值依然为一个函数:
```scala
def add(x:Int): (Int)=>Int = {
  def square(t:Int): Int = {
    t*t
  }
  x + square(_)
}
// add(1)(2) is 5
// 等价于下面的实现方式
def add(x:Int, y:Int): Int = {
    x + y*y
}
```
Currying通过把函数作为函数值，所以支持类似f(x1)(x2)(x3)这种调用方式。这样做能够简化函数的定义方式。
#### 函数参数的延迟加载(lazily evaluated)
Scala支持只有在变量真正使用的才开始给变量分配存储空间，这个叫做延迟延迟加载也是Scala的语言特性之一。之前其实已经介绍了比如使用def定义的变量就是延迟加载。同样函数也支持延迟加载，使用:=>的方式来标明延迟加载。
```scala
def function(x: Int, y: ⇒ Int): Int {
    x*2
}
```
在上面这个例子中y这个变量在函数中就没有没使用到所以不会分配空间。
#### 函数内部嵌套定义函数(NESTED METHODS)
在一个Scala函数内部是可以嵌套定义其他函数的，这样可以避免函数名的泛滥,在一个函数内部定义相关的工具函数也非常好理解。有一个和C++或Python不一样的地方是在嵌套的函数中可以直接引用外部定义的变量。
```scala
def NestFunction(number: Int): Unit = {
  val number_inside = number + 1
  def PrintInput(): Unit = {
    println (number)
    println (number_inside)
  }
  PrintInput()
}
NestFunction(4) // 4,5
```
### Scala的类定义相关
Scala的类定义和大多数语言一样使用Class关键字，可以做访问权限的控制比如private变量，支持继承，多态等等。下面介绍的是几个Scala比较有特点的和类相关的特性。 
#### Case Classes and Pattern Match-ing
Scala中的类可以使用Case来作为关键字，使用case定义的类，可以在实例化的时候不显示的使用new关键字，并且Case Class可以和match case配合在一起使用。
Pattern Match-ing有点类似于C++中的switch()..case表达式，但是比其更强大。
```scala
import scala.util.Random
val x: Int = Random.nextInt(10)
x match {
  case 0 => "zero"
  case 1 => "one"
  case 2 => "two"
  case _ => "many"
}
```
case _表示默认值,下面的例子是case class和pattern match-ing一起使用的例子。
```scala
abstract class Device
case class Phone(model: String) extends Device{
  def screenOff = "Turning screen off"
}
case class Computer(model: String) extends Device {
  def screenSaverOn = "Turning screen saver on..."
} 
case class Telephone(model: String) extends Device {
  def screenSaverOn = "Turning screen saver on..."
} 
def goIdle(device: Device) = device match {
  case p: Phone => p.screenOff
  case c: Computer => c.screenSaverOn
  case t: Telephone if !t.screenSaverOn.empty() => t.screenSaverOn  
}
```
我们可以发现在Scala中match case还能匹配出类型一个的变量。[传送门](https://docs.scala-lang.org/tour/pattern-matching.html)
#### Generate Types
有点类似C++中的泛型模板，很多编程语言都有这种类似的语言特性:
```scala
class Stack[A] {
  private var elements: List[A] = Nil
  def push(x: A) { elements = x :: elements }
  def peek: A = elements.head
  def pop(): A = {
    val currentTop = peek
    elements = elements.tail
    currentTop
  }
}
// Stack[Int], Stack[String]
```
#### Variances
这个我不太清楚应该怎么翻译比较好，但是是一个很有意思的语言特性，可以让并不是直接继承的类成为父类和子类的关系，请看下面的例子：
```scala
class Stack[+A] {
  private var elements: List[A] = Nil
  def push(x: A) { elements = x :: elements }
  def peek: A = elements.head
  def pop(): A = {
    val currentTop = peek
    elements = elements.tail
    currentTop
  }
}

abstract class Animal {
  def name: String
}
case class Cat(name: String) extends Animal
case class Dog(name: String) extends Animal 

def printAnimalNames(animals: Stack[Animal]): Unit = {
  println(animal.peek)
} 
val cats = new Stack[Cat]
val dogs = new Stack[Dog] 
cats.push(Cat("this is cat"))
dogs.push(Cat("this is dog"))

printAnimalNames(cats)
printAnimalNames(dogs)
``` 
使用Class[+A]的作用在于如果Type1是Type2子类，那么Class[Type1]同样也是Class[+Type2]的子类，上面看起来非常合理的调用就不会报错。
[-A]和[+A]刚好相反，如果Type1是Type2的子类，那么Class[Type2]是Class[Type1]的子类。
#### Upper/Lower Type Bounds
同样属于Scala的语言特性，可以限制模板类中的类型,class Type[A :< T]表示A必须要是T的子类。
```scala
abstract class Animal {
 def name: String
}
abstract class Pet extends Animal {}
class Cat extends Pet {
  override def name: String = "Cat"
}
class Lion extends Pet {
  override def name: String = "Lion"
}
class PetContainer[P <: Pet](p: P) {
  def pet: P = p
}
// new PetContainer[Cat](new Cat) is Ok
// new PetContainer[Lion](new Lion) Error
```
反之class Type[A :> T]表示A必须是T的父类。

#### 总结
这里主要介绍以及浏览了一些Scala中的语言特性，这些特性往往在其他编程语言中并不存在(C++/Python)，而之所以设计这些语言特性并没有深入的介绍，后面会继续完善这个Scala系列的文章。
#### 引用
- [ScalaByExample](https://legacy.gitbook.com/book/anildigital/scala-by-example/details) by Martin Odersky
- [Scala的官方说明文档](https://docs.scala-lang.org/tour/tour-of-scala.html)