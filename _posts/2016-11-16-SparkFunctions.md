---
layout: post
title:  "Spark算子详解"
date:   2016-11-16 9:51:00
categories: spark
---

* content
{:toc}

> 详细解释spark各类重要算子

### aggregate

> def aggregate[U: ClassTag](zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U): U

```
    val conf=new SparkConf().setAppName("aggregate").setMaster("local[*]")
    val sc=new SparkContext(conf)
    val z=sc.parallelize(List(1,2,3,4,5,6),2)
//打印每个分区里面的内容
    z.glom().foreach(arr=>{
      println(arr.foldLeft(" ")((x,y)=>x+y))
    })

    // This example returns 16 since the initial value is 5 设置处理值5,注意这个初始值会出现在每个每个分区中作为初始值。就像下面的，每个分区都有5
    // reduce of partition 0 will be max(5, 1, 2, 3) = 5
    // reduce of partition 1 will be max(5, 4, 5, 6) = 6
    // final reduce across partitions will be 5 + 5 + 6 = 16  进行最后一个方法的时候，还会加上初始值5
    // note the final reduce include the initial value 最后得出的才是结果
    // 因此得出结论，aggregate聚合算子的三个参数分别是。第一个是设置初始值。第二个Sep函数是对每个分区中的内容进行操作的函数。第三个Combi函数是聚合每个分区的函数
    val result=z.aggregate(0)(math.max(_,_),_+_)
    val result1=z.aggregate(5)(math.max(_,_),_+_)
    println("result: "+result)
    println("result1: "+result1)
//    456
//    123
//    result: 9
//    result1: 16
    sc.stop()
```

### cartesian

>  def cartesian[U: ClassTag](other: RDD[U]): RDD[(T, U)]

将其中一个分区里面的元素跟另一个分区的每一个元素进行笛卡尔积计算.

```
    val conf = new SparkConf().setAppName("Cartesian").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data1 = Array[(String, Int)](("A1", 1), ("A2", 2),
      ("B1", 3), ("B2", 4),
      ("C1", 5), ("C1", 6))

    val data2 = Array[(String, Int)](("A1", 7), ("A2", 8),
      ("B1", 9), ("C1", 0))
    val pairs1 = sc.parallelize(data1, 3)
    val pairs2 = sc.parallelize(data2, 2)

    val resultRDD = pairs1.cartesian(pairs2)

    resultRDD.foreach(println)
```

### collectAsMap

> 把元组作为元素的List转化成Map,主要是调用可变的HashMap

```
    val conf = new SparkConf().setAppName("CollectAsMap").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data = Array[(String, Int)](("A", 1), ("B", 2),
      ("B", 3), ("C", 4),
      ("C", 5), ("C", 6))

    // as same as "val pairs = sc.parallelize(data, 3)" 底层源码,也还是调用的parallelize
    val pairs = sc.makeRDD(data, 3)
    val result = pairs.collectAsMap

    // output Map(A -> 1, C -> 6, B -> 3)
    print(result)

//    list2map(data)
  }

  def list2map(list: Array[(String, Int)]): mutable.HashMap[String, Int] = {
    val map = new mutable.HashMap[String, Int]
    list.foreach(value=> map.put(value._1,value._2))
    map
```

### flatMap

> def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U]

```
    val conf = new SparkConf().setAppName("FlatMap").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val array=Array[String]("Hello","World","hadoop")
    val strRDD=sc.parallelize(array)
    val str2arrayRDD=strRDD.flatMap(x=>x.toCharArray)
    str2arrayRDD.foreach(println)
```

### GroupByKey

> def groupByKey(numPartitions: Int): RDD[(K, Iterable[V])]

会造成shuffle的算子，根据key分组

```
    var numMappers = 10
    var numPartition=10
    var numKVPairs = 100
    var valSize = 100
    var numReducers = 3

    val conf = new SparkConf().setAppName("GroupBy Test").setMaster("local[*]")
    val sc = new SparkContext(conf)
//  造一些测试数据
//    0到10，10次循环，10个分区
//    每次循环里面造100个键值对
    val pairs1 = sc.parallelize(0 until numMappers, numPartition).flatMap { p =>
      val ranGen = new Random
      var arr1 = new Array[(Int, Array[Byte])](numKVPairs)
      for (i <- 0 until numKVPairs) {
        val byteArr = new Array[Byte](valSize)
        ranGen.nextBytes(byteArr)
        arr1(i) = (ranGen.nextInt(10), byteArr)
      }
      arr1
    }.cache
    // cache一下
//    因此一共是10X100=1000个
    println("pairs1.count: "+pairs1.count)

    val result = pairs1.groupByKey(numReducers)
//    造数据的时候，key的取值是10以内，所以分组肯定是10个，和结果一样
    println("result.count: "+result.count)
    println(result.toDebugString)

    sc.stop()
```

### groupBy

> def groupBy[K](f: T => K)(implicit kt: ClassTag[K]): RDD[(K, Iterable[T])]

```
    val conf = new SparkConf().setAppName("GroupByAction").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data = Array[(String, Int)](("A1", 1), ("A2", 2),
      ("B1", 6), ("A2", 4),
      ("B1", 3), ("B1", 5))

    val pairs = sc.parallelize(data, 3)
    pairs.foreach(println)
    val result1 = pairs.groupBy(K => K._1)
    // 设置分区数量
    val result2 = pairs.groupBy((K: (String, Int)) => K._1,1)
    // 定义分区的类
    val result3 = pairs.groupBy((K: (String, Int)) => K._1, new RangePartitioner(3, pairs))

    result1.foreach(println)
    result2.foreach(println)
    result3.foreach(println)
```

### groupWith

>  def groupWith[W](other: RDD[(K, W)]): RDD[(K, (Iterable[V], Iterable[W]))]

Alias for cogroup

```
    val conf = new SparkConf().setAppName("GroupWith").setMaster("local[*]")
    val sc = new SparkContext(conf)

    val data1 = Array[(String, Int)](("A1", 1), ("A2", 2),
      ("B1", 3), ("B2", 4),
      ("C1", 5), ("C1", 6)
    )

    val data2 = Array[(String, Int)](("A1", 7), ("A2", 8),
      ("B1", 9), ("C1", 0)
    )
    val pairs1 = sc.parallelize(data1, 3)
    val pairs2 = sc.parallelize(data2, 2)

    val result = pairs1.groupWith(pairs2)
    result.foreach(println)
```

### join

> def join[W](other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, W))]

```
    val conf = new SparkConf().setAppName("JoinAction").setMaster("local[*]")
    val sc = new SparkContext(conf)

    val data1 = Array[(String, Int)](("A1", 1), ("A2", 2),
      ("B1", 3), ("B2", 4),
      ("C1", 5), ("C1", 6)
    )

    val data2 = Array[(String, Int)](("A1", 7), ("A2", 8),
      ("B1", 9), ("C1", 0)
    )
    val pairs1 = sc.parallelize(data1, 3)
    val pairs2 = sc.parallelize(data2, 2)


    val result = pairs1.join(pairs2)
    result.foreach(println)
```

### lookup

> def lookup(key: K): Seq[V]

(针对pair类型的RDD)根据这个K值,找出相应的value值，放入到List里面返回出来

```

    val conf = new SparkConf().setAppName("LookUp").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data = Array[(String, Int)](("A", 1), ("B", 2),
      ("B", 3), ("C", 4),
      ("C", 5), ("C", 6))

    val pairs = sc.parallelize(data, 3)

    val finalRDD = pairs.lookup("B")

    finalRDD.foreach(println)
```

### mapPartitions

> def mapPartitions[U: ClassTag](f: Iterator[T] => Iterator[U],preservesPartitioning: Boolean = false): RDD[U]

是针对这个RDD的每个partition进行操作的，比如在进行数据库链接的时候，使用这个算子，可以减少

```
val conf = new SparkConf().setAppName("MapPartitionsRDD").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data = Array[(String, Int)](("A1", 1), ("A2", 2),
      ("B1", 1), ("B2", 4),
      ("C1", 3), ("C2", 4)
    )
    val pairs = sc.parallelize(data, 3)

    val finalRDD = pairs.mapPartitions(iter => iter.filter(_._2 >= 2))

    finalRDD.foreachPartition(iter => {
      while (iter.hasNext) {
        val next = iter.next()
        println(next._1 + " --- " + next._2)

      }
    })
```

### mapValues

> def mapValues[U](f: V => U): RDD[(K, U)]

不改变RDD原有的分区，也不改变Key值，修改value的值

```
    val conf = new SparkConf().setAppName("mapValues").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data1 = Array[(String, Int)](("K", 1), ("T", 2),
      ("T", 3), ("W", 4),
      ("W", 5), ("W", 6)
    )
    val pairs = sc.parallelize(data1, 3)
    val result = pairs.mapValues(V => 10 * V)
//    val result=pairs.map { case (k, v) => (k, v * 10) }
    result.foreach(println)
```

### partitionBy

> def partitionBy(partitioner: Partitioner): RDD[(K, V)]

用指定的分区类来处理RDD

```
    val conf = new SparkConf().setAppName("partitionBy").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data1 = Array[(String, Int)](("K", 1), ("T", 2),
      ("T", 3), ("W", 4),
      ("W", 5), ("W", 6)
    )
    val pairs = sc.parallelize(data1, 3)
    val result = pairs.partitionBy(new RangePartitioner(2, pairs, true))
    //val result = pairs.partitionBy(new HashPartitioner(2))
    result.foreach(println)
```

### pipe

> def pipe(command: String): RDD[String]

command命令在windows上测不了额

```
    val conf = new SparkConf().setAppName("Pip").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data1 = Array[(String, Int)](("K1", 1), ("K2", 2),
      ("U1", 3), ("U2", 4),
      ("W1", 3), ("W2", 4)
    )
    val pairs = sc.parallelize(data1, 3)
    val finalRDD = pairs.pipe("grep 2")
    finalRDD.foreach(println)
```

### reduce

> def reduce(f: (T, T) => T): T

迭代处理之前的元素

```
    val conf = new SparkConf().setAppName("reduce").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data1 = Array[(String, Int)](("K1", 1), ("K2", 2),
      ("U1", 3), ("U2", 4),
      ("W1", 3), ("W2", 4)
    )
    val pairs = sc.parallelize(data1, 3)
    val result = pairs.reduce((A, B) => (A._1 + "#" + B._1, A._2 + B._2))
    //val result = pairs.fold(("K0",10))((A, B) => (A._1 + "#" + B._1, A._2 + B._2))

    println(result)
```

### reduceByKey

> reduceByKey(func: (V, V) => V): RDD[(K, V)]

有好几个重载方法,可以指定分区数,分区的类

```
    val conf = new SparkConf().setAppName("reduceByKey").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data1 = Array[(String, Int)](("K", 1), ("U", 2),
      ("U", 3), ("W", 4),
      ("W", 5), ("W", 6)
    )
    val pairs = sc.parallelize(data1, 3)
    val result = pairs.reduceByKey(_ + _)
    result.foreach(println)
```

### sortByKey

>  def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length): RDD[(K, V)]

```
    val conf = new SparkConf().setAppName("sortByKey").setMaster("local[*]")
    val sc = new SparkContext(conf)

    val data1 = Array[(String, Int)](("K1", 1), ("K2", 2),
      ("U1", 3), ("U2", 4),
      ("W1", 5), ("W1", 6)
    )
    val pairs1 = sc.parallelize(data1, 3)

    //val result = pairs.fold(("K0",10))((A, B) => (A._1 + "#" + B._1, A._2 + B._2))

    val result = pairs1.sortByKey()
    result.foreach(println)
```

### take

> def take(num: Int): Array[T]

```
    val conf = new SparkConf().setAppName("take").setMaster("local[*]")
    val sc = new SparkContext(conf)
    val data1 = Array[(String, Int)](("K1", 1), ("K2", 2),
      ("U1", 3), ("U2", 4),
      ("W1", 3), ("W2", 4)
    )
    val pairs = sc.parallelize(data1, 3)
    val result = pairs.take(5)
    result.foreach(println)
```

### union

> def union(other: RDD[T]): RDD[T]

union聚合算子,不会去重,如果需要去重,调用distinct算子

```
    val conf = new SparkConf().setAppName("union").setMaster("local[*]")
    val sc = new SparkContext(conf)

    val data1 = Array[(String, Int)](("K1", 1), ("K2", 2),
      ("U1", 3), ("U2", 4),
      ("W1", 5), ("W1", 6)
    )

    val data2 = Array[(String, Int)](("K1", 7), ("K2", 8),
      ("U1", 9), ("W1", 0)
    )
    val pairs1 = sc.parallelize(data1, 3)
    val pairs2 = sc.parallelize(data2, 2)
    val result = pairs1.union(pairs2)
    result.foreach(println)
```