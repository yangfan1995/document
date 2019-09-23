# 基础

## 声明和变量定义

`val` 声明了常量，`var`声明了可变变量

## 算数和运算符重载

> 运算符重载：对已有的*运算符*重新进行定义，赋予其另一种功能，以适应不同的数据类型。

根据具体的程序的可读性来判断需要什么风格代码

> scala代码没有具体的++，--，只能使用+=1，-=1

引包方式：`import scala.math._`_,_类似于*，全匹配

apply()方法是scala构建对象常用的方式，`Array(1,2,3,4,5) == Array.apply(1,2,3,4,5)`

# 控制结构和函数

## 变长参数

```scala
  def sum(args: Int*): Int = {
    var result = 0
    for (arg <- args) result += arg
    result
  }
```

如果传入一个参数，不能是一个区间，只能具体化

```scala
int result = sum(1 to 5: _*)
```

## 过程

没有具体的返回值，只是为了副作用结果，`println()`

## 懒值

对于开销比较大的初始化操作影响较大，只有实际调用时才会触发相关操作，但是存在着额外的开销，每次访问都会调用一个线程安全的方法去判断该变量是否已经初始化。

> 只能用来修饰val静态变量，`lazy val`

## 异常

类同Java，try{}catch{}finally{}

# 数组

> 定长使用Array，变长使用ArrayBuffer

### until和to的区别

指定到整数n，until是到n-1(不包含n)，to是到n(包含n)

## 定长数组

当数组元素确定时候不用使用new 关键字

```scala
var array = Array("hello","world")
```

## 变长数组(数组缓冲)

Scala中ArrayBuffer类似于Java中ArrayList，首尾操作比较快，中间插入需要后移

## 遍历数组和数组缓冲

```scala
//数组倒叙遍历
for (elem<- array.indices.reverse) {
    println(elem)
}
```

## 数组转换

for()yeild方式会产生一个新的数组或数组缓冲，不会改变原数据。或者通过for中的if守卫进行过滤操作生成新的数组或者数组缓冲。

## 常用算法

sum求和，min最小值，max最大值，mkString会报告具体数据，可以指定分隔符或者前后缀

## 多维数组

可以使用`ofDim`方法，使用`array(row,column)`方式访问

```scala
Array.ofDim[Double](3,4)
```

# 映射与元组

## 构造

默认可以使用

```scala
var map = Map("张三"->20,"李四"->21)
```

如果单纯的声明为可变的map可以使用，需要具体指定key，value的类型

```scala
var map = new scala.collection.mutable.HashMap[String,Int]
```

## 取值

可以直接使用key方式取值

```scala
var age = map("张三")
```

如果需要判断数值不存在，涉及到Option类

```scala
var age = map.getOrElse("张三",15)
```

## 更新

放在等号左侧可以赋值，`+=`方式可以添加键值对

## 迭代

可以使用keySet和values进行数据的遍历

```scala
val tuplesToStringToInt = new scala.collection.mutable.HashMap[String, Int]
    tuplesToStringToInt += ("张三" -> 20)
    tuplesToStringToInt += ("李四" -> 21)
    tuplesToStringToInt += ("王五" -> 15)

    val keySet = tuplesToStringToInt.keySet
    val values = tuplesToStringToInt.values
    for (key <- keySet) println(key)
    for (value <- values) println(value)
```

## 元组

对偶是最简单的元组，可以使用`Tuple[N]`来定义，可以使用`_1`,`_2`方式访问。

## 拉链操作

将多个数组进行对偶操作，变成元组数组进行操作，使用`zip`函数

```scala
val temp1 = Array("name1", "name2", "name3")
val temp2 = Array("value1", "value2", "value3")

val tuples = temp1.zip(temp2)

val map = tuples.toMap

for ((k, v) <- tuples) println(k + ":" + v)
```

# 类

## 简单类和无参构造

一般是改值器的方法调用会加上`()`，对于取值器是不加，可以在对象中使用

```
def method = value
```

强制使用不包含括号

# 对象

## 单例对象

Scala中没有静态方法和静态字段，可以使用object方式达到同样的目的，如果该对象没有使用过，就不会调用构造函数

## 伴生对象

object 修饰跟类对象存在一个文件中，属性可以相互访问。

## apply方法

Array定义了相关的apply方法，Array(100)是包含一个元素值为100的数组，调用的是`apply(100)`；`new Array(100)`是100个为Nothing的数组，调用的是`this(100)`