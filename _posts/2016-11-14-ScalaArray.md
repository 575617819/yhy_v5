---
layout: post
title:  "Scala数组"
date:   2016-11-14 10:59:00
categories: scala
---

* content
{:toc}

> 本篇提供Scala基础语法,供熟悉Java语言的开发者,快速上手Scala语言

----

# Scala数组

> 说明：存储相同类型的元素的固定大小的连续集合

+ 相同类型
+ 固定大小

## 声明数组变量

在程序中要使用数组,那么就需要先声明一个变量来引用数组.

```
var arr:Array[String]=new Array[String](3)
or
var arr=new Array[String](3)
or
var arr=Array("hello","world","scala")
```

在这里,arr被声明为字符串数组,最多可以容纳三个变量.

## 给数组变量赋值

有以下的方式给数组变量赋值

```
arr(0)="hello"
arr(1)="world"
arr(4/2)="scala"
创建的时候就赋值
```

## 处理数组

当要处理数组元素的时候,根据数组元素具有相同的类型和数组的大小已知这些特点,我们经常就会使用循环来处理数组.

**创建,初始化,处理数组**

```
object Test {
   def main(args: Array[String]) {
      var myList = Array(1.9, 2.9, 3.4, 3.5)

      // Print all the array elements
      for ( x <- myList ) {
         println( x )
      }

      // Summing all elements
      var total = 0.0;
      for ( i <- 0 to (myList.length - 1)) {
         total += myList(i);
      }
      println("Total is " + total);

      // Finding the largest element
      var max = myList(0);
      for ( i <- 1 to (myList.length - 1) ) {
         if (myList(i) > max) max = myList(i);
      }
      println("Max is " + max);

   }
}
```

## 可变数组

> 在程序中，我们常常需要可变的数组，ArrayBuffer

```
    val buffer=new ArrayBuffer[Int]
    for (i<- 0 until 10 if i%2==0) {
      buffer.+=(i)
    }
    println(buffer.mkString(" - "))
```
----

# scala多维数组

>+ 在很多情况下,仅仅只用一维数组是没法解决问题的.因此我们需要定义多维数组(即数组的元素也是数组)
>+ 例如,矩阵
>+ scala不直接支持多维数组,但提供各种方法来处理任何尺寸数组

## 创建scala多维数组

```
import Array._
var myMatrix=ofDim[Int](3,3)
```

myMatrix是一个具有每个元素都是整数,它是一个3X3的数组

## 处理scala多维数组

```
import Array._

object Test {
   def main(args: Array[String]) {
      var myMatrix = ofDim[Int](3,3)

      // build a matrix
      for (i <- 0 to 2) {
         for ( j <- 0 to 2) {
            myMatrix(i)(j) = j;
         }
      }

      // Print two dimensional array
      for (i <- 0 to 2) {
         for ( j <- 0 to 2) {
            print(" " + myMatrix(i)(j));
         }
         println();
      }

   }
}
```

# 联接scala数组

> 使用concat()方法来连接两个数组

```
import Array._

object Test {
   def main(args: Array[String]) {
      var myList1 = Array(1.9, 2.9, 3.4, 3.5)
      var myList2 = Array(8.9, 7.9, 0.4, 1.5)

      var myList3 =  concat( myList1, myList2)

      // Print all the array elements
      for ( x <- myList3 ) {
         println( x )
      }
   }
}
```

----

# 创建具有范围的数组(常用)

>+ 该方法很实用,可以方便的创建一个数组供测试或者开发用
>+ range() 产生包含在给定的范围内增加整数序列的数组.最后一个参数是步长,默认是1

```
import Array._

object Test {
   def main(args: Array[String]) {
      var myList1 = range(10, 20, 2)
      var myList2 = range(10,20)

      // Print all the array elements
      for ( x <- myList1 ) {
         print( " " + x )
      }
      println()
      for ( x <- myList2 ) {
         print( " " + x )
      }
   }
}
```
