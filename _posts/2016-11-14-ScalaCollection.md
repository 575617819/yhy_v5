---
layout: post
title:  "Scala集合"
date:   2016-11-14 13:37:00
categories: scala
---

* content
{:toc}

> 本篇提供Scala基础语法,供熟悉Java语言的开发者,快速上手Scala语言

----

# Scala集合

scala有一组丰富的集合库:List,Tuple,Option,Map

+ 集合可能是严格或懒惰
+ 集合可以是可变的或不可变
+ 不可变的集合可能包含可变项

----

# List

## List与Array区别

+ 相同点
    + 所有元素都具有相同的类型
+ 不同点
    + 列表是不可变的,这意味着一个列表的元素可以不被分配来改变.
    + 列表底层是一个链表,而数组是平坦的.

## 定义Scala的List列表

1.有以下几种常用的方式来定义列表

```
// List of Strings
val fruit: List[String] = List("apples", "oranges", "pears")

// List of Integers
val nums: List[Int] = List(1, 2, 3, 4)

// Empty List.
val empty: List[Nothing] = List()

// Two dimensional list
val dim: List[List[Int]] =
   List(
      List(1, 0, 0),
      List(0, 1, 0),
      List(0, 0, 1)
   )
```

2.所有的列表可以使用两种基本的构建模块来定义,一个无尾Nil和::,但是这有明显的缺点.Nil也代表了空列表.上述的例子,也可以这样写

```
// List of Strings
val fruit = "apples" :: ("oranges" :: ("pears" :: Nil))

// List of Integers
val nums = 1 :: (2 :: (3 :: (4 :: Nil)))

// Empty List.
val empty = Nil

// Two dimensional list
val dim = (1 :: (0 :: (0 :: Nil))) ::
          (0 :: (1 :: (0 :: Nil))) ::
          (0 :: (0 :: (1 :: Nil))) :: Nil
```

**::符号就可以简单的替代了List()方法**

## 列表的基本操作

> 主要针对列表的头尾合是否为空来说

|方法  |描述  |
|----|----|
| head |此方法返回的列表中的第一个元素|
| tail  |此方法返回一个由除了第一个元素外的所有元素的列表|
| isEmpty |如果列表为空，此方法返回true，否则为false|

```
object Test {
   def main(args: Array[String]) {
      val fruit = "apples" :: ("oranges" :: ("pears" :: Nil))
      val nums = Nil

      println( "Head of fruit : " + fruit.head )
      println( "Tail of fruit : " + fruit.tail )
      println( "Check if fruit is empty : " + fruit.isEmpty )
      println( "Check if nums is empty : " + nums.isEmpty )
   }
}
```

## 串联列表

> 可以使用::运算符或者列表List.:::()方法来添加两个或多个列表

eg:

```
object Test {
   def main(args: Array[String]) {
      val fruit1 = "apples" :: ("oranges" :: ("pears" :: Nil))
      val fruit2 = "mangoes" :: ("banana" :: Nil)

      // use two or more lists with ::: operator
      var fruit = fruit1 ::: fruit2
      println( "fruit1 ::: fruit2 : " + fruit )

      // use two lists with Set.:::() method
      fruit = fruit1.:::(fruit2)
      println( "fruit1.:::(fruit2) : " + fruit )

      // pass two or more lists as arguments
      fruit = List.concat(fruit1, fruit2)
      println( "List.concat(fruit1, fruit2) : " + fruit  )


   }
}
```

:: 元素之间进行连接,:::列表之间进行连接

## 创建统一列表
> 可以使用List.fill()方法来创建,包括相同的元素如下的零个或更多个拷贝的列表

eg

```
object Test {
   def main(args: Array[String]) {
      val fruit = List.fill(3)("apples") // Repeats apples three times.
      println( "fruit : " + fruit  ) //fruit : List(apples, apples, apples)

      val num = List.fill(10)(2)         // Repeats 2, 10 times.
      println( "num : " + num  ) //num : List(2, 2, 2, 2, 2, 2, 2, 2, 2, 2)
   }
}
```

## 反向列表顺序
> 可以使用List.reverse方法来扭转列表中的所有元素

eg

```
object Test {
   def main(args: Array[String]) {
      val fruit = "apples" :: ("oranges" :: ("pears" :: Nil))
      println( "Before reverse fruit : " + fruit )
      // Before reverse fruit : List(apples, oranges, pears)
      println( "After reverse fruit : " + fruit.reverse )
      // After reverse fruit : List(pears, oranges, apples)
   }
}
```

## Scala列表常用方法

|方法| 描述|
|----|----|
| def +(elem: A): List[A]  |前置一个元素列表|
| def ::(x: A): List[A] | 在这个列表的开头添加的元素。返回一个新的列表|
| def :::(prefix: List[A]): List[A] | 增加了一个给定列表中该列表前面的元素。返回一个新的列表|
| def contains(elem: Any): Boolean | 测试该列表中是否包含一个给定值作为元素。|
| def distinct: List[A] | 建立从列表中没有任何重复的元素的新列表。|
| def drop(n: Int): List[A] | 返回除了第n个的所有元素。|
| def endsWith[B](that: Seq[B]): Boolean  | 测试列表是否使用给定序列结束。
| ......| ......|

----

# Set

+ Set集合是不包含重复元素的集合
+ 默认情况下,Scala中使用不可变的集.如果想使用可变集,必须明确导入sala.collection.mutable.Set类

## 声明Set集合类

```
// Empty set of integer type
var s : Set[Int] = Set()

// Set of integer type
var s : Set[Int] = Set(1,3,5,7)
or
var s = Set(1,3,5,7)
```

## 集合基本操作

> 集合所有操作可以体现在以下三个方法

|方法|描述|
|----|----|
|head|此方法返回集合的第一个元素。|
tail|该方法返回集合由除第一个以外的所有元素。|
|isEmpty|如果设置为空，此方法返回true，否则为false。|

eg:

```
object Test {
   def main(args: Array[String]) {
      val fruit = Set("apples", "oranges", "pears")
      val nums: Set[Int] = Set()

      println( "Head of fruit : " + fruit.head )
      println( "Tail of fruit : " + fruit.tail )
      println( "Check if fruit is empty : " + fruit.isEmpty )
      println( "Check if nums is empty : " + nums.isEmpty )
   }
}
```

## 串联Set集合

> 可以使用++运算符或集。++()方法来连接两个或多个集，但同时增加了集它会删除重复的元素

eg

```
object Test {
   def main(args: Array[String]) {
      val fruit1 = Set("apples", "oranges", "pears")
      val fruit2 = Set("mangoes", "banana")

      // use two or more sets with ++ as operator
      var fruit = fruit1 ++ fruit2
      println( "fruit1 ++ fruit2 : " + fruit )

      // use two sets with ++ as method
      fruit = fruit1.++(fruit2)
      println( "fruit1.++(fruit2) : " + fruit )
   }
}
```

## ScalaSet集合常用方法

|方法|描述
|-----|----|
|def +(elem: A): Set[A] | 创建一组新的具有附加元件，除非该元件已经存在|
|def -(elem: A): Set[A] | 创建一个新的从这个集合中删除一个给定的元素|
|def contains(elem: A): Boolean | 如果elem包含在这个集合返回true，否则为false。|
|def take(n: Int): Set[A] | 返回前n个元素|
|......|......|

----

# Map

+ 在scala中的映射是键/值对的集合即是Map
+ 任何值可以根据它的键进行检索
+ 键是在映射中唯一的,但值不一定是唯一的
+ 映射也被称为哈希表
+ 有两种映射,不可变的以及可变的.不可变的意味着对象本身是不可变的.
+ 默认情况下,scala中使用不可变的映射.要想使用可变集,必须明确导入scala.collection.mutable.Map类

## 声明Map类

```
// Empty hash table whose keys are strings and values are integers:
var A:Map[Char,Int] = Map()

// A map with keys and values.
val colors = Map("red" -> "#FF0000", "azure" -> "#F0FFFF")
```

## 映射的基本操作

> 在映射上的所有操作可被表示在下面的三种方法

|方法|描述|
|-----|------|
|keys|这个方法返回一个包含映射中的每个键的迭代。|
|values|这个方法返回一个包含映射中的每个值的迭代。|
|isEmpty|如果映射为空此方法返回true，否则为false。|

### 举例

```
object Test {
   def main(args: Array[String]) {
      val colors = Map("red" -> "#FF0000",
                       "azure" -> "#F0FFFF",
                       "peru" -> "#CD853F")

      val nums: Map[Int, Int] = Map()

      println( "Keys in colors : " + colors.keys )
      println( "Values in colors : " + colors.values )
      println( "Check if colors is empty : " + colors.isEmpty )
      println( "Check if nums is empty : " + nums.isEmpty )
   }
}
```

### 给可变映射添加元素

```
A += ('I' -> 1)
A += ('J' -> 5)
A += ('K' -> 10)
A += ('L' -> 100)
```

## 串联映射

> 可以使用++运算符或映射。++()方法来连接两个或更多的映射，但同时增加了映射，将删除重复的键

eg

```
object Test {
   def main(args: Array[String]) {
      val colors1 = Map("red" -> "#FF0000",
                        "azure" -> "#F0FFFF",
                        "peru" -> "#CD853F")
      val colors2 = Map("blue" -> "#0033FF",
                        "yellow" -> "#FFFF00",
                        "red" -> "#FF0000")

      // use two or more Maps with ++ as operator
      var colors = colors1 ++ colors2
      println( "colors1 ++ colors2 : " + colors )

      // use two maps with ++ as method
      colors = colors1.++(colors2)
      println( "colors1.++(colors2)) : " + colors )

   }
}
```

## 遍历Map

> 方法很多,模式匹配最方便了

```
    val colors = Map("red" -> "#FF0000",
      "azure" -> "#F0FFFF",
      "peru" -> "#CD853F")

    println("第一种遍历")
    colors.keys.foreach{key=>
    print("key= "+key)
    println(" value= "+colors(key))}

    println("第二种遍历")

    for(key <- colors.keySet.toArray) {
      println(key+" :　"+colors.get(key))
    }

    println("第三种遍历")

    colors.map{
      case (k,v)=>println(k+" : "+v)
    }
```

## scala中Map的常用方法

|方法|描述|
|-----|------|
| def contains(key: A): Boolean | 如果有一个绑定在该映射的键返回true，否则为false。|
| def clone(): Map[A, B] | 创建接收器对象的副本|
| def clear(): Unit | 从映射中删除所有绑定。在此之后操作已完成时，映射将是空的。|
| .....|.....|

----

# Tuple

> Scala的元组结合多个固定数量在一起,使它们可以被传来传去作为一个整体.

## 特点

+ 不像数组或列表,元组可以容纳不同类型的对象
+ 同时记住,元组也是不可改变的
+ scala目前的元组上限在22个

## 声明Tuple元组

一个元组的实际类型取决于它包含的元素和这些元素的类型的数目

```
val t = (1, "hello", Console)

这是语法修饰(快捷方式)以下：

val t = new Tuple3(1, "hello", Console)

//类型:
// (99, "Luftballons") 是 Tuple2[Int, String]
// ('u', 'r', "the", 1, 4, "me") 的类型是 Tuple6[Char, Char, String, Int, Int, String]
```

## 访问元组中的元素

```
object Test {
   def main(args: Array[String]) {
      val t = (4,3,2,1)

      val sum = t._1 + t._2 + t._3 + t._4

      println( "Sum of elements: "  + sum )
   }
}
```

## 遍历元组

> 可以使用Tuple.productIterator()方法来遍历一个元组的所有元素

```
object Test {
   def main(args: Array[String]) {
      val t = (4,3,2,1)
      t.productIterator.foreach{ i =>println("Value = " + i )}
   }
}
```

## 元组的简单方法

1.转换为字符串
2.交换元素

```
object Test {
   def main(args: Array[String]) {
      val t1 = new Tuple3(1, "hello", Console)
      println("Concatenated String: " + t1.toString() )

      val t2 = new Tuple2("Scala", "hello")

      println("Swapped Tuple: " + t2.swap )
   }
}
```