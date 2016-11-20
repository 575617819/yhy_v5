---
layout: post
title:  "ScalaNote"
date:   2016-11-12 18:24:00
categories: scala
---

* content
{:toc}

# Scala学习笔记

## 1.sealed trait
> 被sealed 声明的 trait仅能被同一文件的的类继承。
  除了这个，我们通常将sealed用于枚举中，因为编译器在编译的时候知道这个trait被哪些类继承过，因此我们在match时对sealed trait进行case 的时候，如果你没有判断全部编译器在编译时就会报错。

```
@DeveloperApi
sealed trait JobResult

@DeveloperApi
case object JobSucceeded extends JobResult

@DeveloperApi
private[spark] case class JobFailed(exception: Exception) extends JobResult
```

## 2.assert
> Scala的assert方法检查传入的Boolean并且如果是假，抛出AssertionError。如果传入的Boolean是真，assert只是静静地返回。

## 3.偏函数(Partial Function)
> 偏函数是只对函数定义域的一个子集进行定义的函数。 scala中用scala.PartialFunction[-T, +S]类来表示
> 因为偏函数都只对其定义域Int的部分值做了处理. 那么定义成偏函数的额外好处是, 你可以在调用前使用一个isDefinedAt方法, 来校验参数是否会得到处理.  或者在调用时使用一个orElse方法, 该方法接受另一个偏函数,用来定义当参数未被偏函数捕获时该怎么做. 也就是能够进行显示的声明. 在实际代码中最好使用PartialFunction来声明你确实是要定义一个偏函数, 而不是漏掉了什么.

比如定义了一个函数：

```
    def sum(x: Int)(y: Int) = x + y
```

当调用sum的时候，如果不提供所有的参数或某些参数还未知时，比如

```
    sum _
    sum(3)(_: Int)
    sum(_: Int)(3)
```

这样就生成了所谓的部分应用函数。部分应用函数只是逻辑上的一个表达，scala编译器会用Function1， Function2这些类来表示它.

下面这个变量signal引用了一个偏函数

```
    val signal: PartialFunction[Int, Int] = {
        case x if x > 1 => 1
        case x if x < -1 => -1
    }
```

这个signal所引用的函数除了0值外，对所有整数都定义了相应的操作。
signal(0) 会抛出异常，因此使用前最好先signal.isDefinedAt(0)判断一下。

+ 偏函数主要用于这样一种场景:
对某些值现在还无法给出具体的操作（即需求还不明朗），也有可能存在几种处理方式（视乎具体的需求）；我们可以先对需求明确的部分进行定义，比如上述除了0外的所有整数域，然后根据具体情况补充对其他域的定义，比如 :

```
    val composed_signal: PartialFunction[Int,Int] = signal.orElse{
    case 0 => 0
    }

    composed_signal(0)  // 返回 0
```

或者对定义域进行一定的偏移（假如需求做了变更,  1 为无效的点）

```
    val new_signal: Function1[Int, Int] = signal.compose{
      case x => x  - 1
    }

    new_signal(1)  // throw exception
    new_signal(0)   // 返回 -1
    new_signal(2)  // 返回 1
```

还可以用andThen将两个相关的偏函数串接起来

```
    val another_signal: PartialFunction[Int, Int] = {
       case 0 =>  0
       case x if x > 0 => x - 1
       case x if x < 0 => x + 1
    }

    val then_signal =  another_signal andThen  signal
```

这里的then_signal 剔除了-1, 0, 1三个点的定义

## 4.部分应用函数(Partial Applied Function)
> 部分应用函数,是指一个函数有N个参数,而我们为其提供少于N个参数,那就得到了一个部分应用函数.

比如我先定义一个函数

```
    def sum(a:Int,b:Int,c:Int)=a+b+c;
```

那么就可以从这个函数衍生出一个偏函数是这样的:

```
    def p_sum=sum(1,_:Int,_:Int)
```

于是就可以这样调用p_sum(2,3), 相当于调用sum(1,2,3) 得到的结果是6. 这里的两个_分别对应函数sum对应位置的参数. 所以你也可以定义成

```
    def p_sum = sum (_:Int, 1, _:Int)
```

这东西有啥用呢? 一个是当你在代码中需要多次调用一个函数, 而其中的某个参数又总是一样的时候, 使用这个可以使你少敲一些代码. 另一个呢?

## 5.函数与闭包

### 函数

> 函数式编程风格的一个重要设计原则:函数是一等公民,一个程序可以被解构成若干个小的函数,每个完成一个定义良好的任务

1.本地函数
Scala中提供了可以把函数定义在另一个函数中。就好象本地变量那样，这种本地函数仅在包含它的代码块中可见。

```
 def processFile(fileName:String,width:Int){
      // 本地函数
      def processLine(line:String){
          // 长度大数指定的长度，进行打印
          if(line.length() > width){
              println(fileName+":"+line)
          }
      }

      // 读取文件
      val source = Source.fromFile("E:\\input.txt");
      for(line <- source.getLines()){
          processLine(line);
      }
  }
```

processLine的定义放在processFile的定义里。作为本地函数，processLine的范围局限于processFile之内，外部无法访问。