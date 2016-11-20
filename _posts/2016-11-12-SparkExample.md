---
layout: post
title:  "PySpark做日志分析"
date:   2016-11-12 18:24:00
categories: spark
---

* content
{:toc}


> 用Python做日志分析,这个笔记用Python做Apache access 日志分析

----

## 第一步:给你的样本日志目录修改这个参数

在这个例子里面，我们会用到这个样本文件"/databricks-datasets/sample_logs",样本数据已经存储到DBFS(Databricks的文件系统)
> DBFS_SAMPLES_LOGS_FOLDER="/databricks-datasets/samples_logs" #path to the log file to be analyzed

## 第二步:为Apache访问日志的每一行创建一个解析器以此来创建一个Row对象

### 每行日志的格式

为了分析这些数据,我们需要去把他们解析成可看的格式
对与这个实验，我们要用的日志数据在[Apache Common Log Format( clf)](http://httpd.apache.org/docs/1.3/logs.html#common).
CLF生产的日志文件条目将如下所示: 127.0.0.1 - - [01/Aug/1995:00:00:01 -0400] "GET /images/launch-logo.gif HTTP/1.0" 200 1839
以下是摘要，描述日志记录的每一部分

+ 127.0.0.1 这个是访问服务器的客户端(远程主机)的IP地址(或者是主机名,可选)
+ \- 这个"连字符"在输出中表示继续请求的信息(从远程机器来的身份)是不可用的
+ \- 这个"连字符"在输出中表示继续请求的信息(从本地登录的用户身份)是不可用的
+ [01/Aug/1995:00:00:01 -0400] 服务器完成请求的时间.格式是这样的:
    - day=2 digits
    - month=3 letters
    - year=4 digits
    - hour=2 digits
    - minute=2 digits
    - second=2 digits
    - zone=(+|-)4 digits
+ "GET /images/launch-logo.gif HTTP/1.0" 这个是从客户端来的请求字符串的第一行。它是由三部分组成的:访问的方式(e.g.,GET,POST,etc),终端(统一资源标识符),
还有客户端的协议版本
+ 200 这个是服务器返回给客户端的状态码.这个信息是非常重要的,因为它显示了该请求是否是一个成功的响应(状态码以2开始),
一个重定向(状态码以3开始),一个由客户端导致的错误(状态码以4开始),或者一个在服务器的错误(状态码以5开始).可获得的状态代码的完整列表可以在HTTP规范中找到[RFC 2616 section 10](https://www.ietf.org/rfc/rfc2616.txt)
+ 1839 这最后一个记录显示返回给客户端的对象的大小,不包括响应headers.如果没有内容返回给客户端,那么这个值可能是"-"(或者有时候是0)
因此,对于有恶意的客户端可能会插入控制日志文件中的字符,所以必须注意处理源日志.

----

#### 日志示例

```
64.242.88.10 - - [07/Mar/2004:16:05:49 -0800] "GET /twiki/bin/edit/Main/Double_bounce_sender?topicparent=Main.ConfigurationVariables HTTP/1.1" 401 12846
64.242.88.10 - - [07/Mar/2004:16:06:51 -0800] "GET /twiki/bin/rdiff/TWiki/NewUserTemplate?rev1=1.3&rev2=1.2 HTTP/1.1" 200 4523
64.242.88.10 - - [07/Mar/2004:16:10:02 -0800] "GET /mailman/listinfo/hsdivision HTTP/1.1" 200 6291
64.242.88.10 - - [07/Mar/2004:16:11:58 -0800] "GET /twiki/bin/view/TWiki/WikiSyntax HTTP/1.1" 200 7352
64.242.88.10 - - [07/Mar/2004:16:20:55 -0800] "GET /twiki/bin/view/Main/DCCAndPostFix HTTP/1.1" 200 5253
```

----

```
import re
from pyspark.sql import Row

APACHE_ACCESS_LOG_PATTERN='^(\S+) (\S+) (\S+) \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+) (\S+)" (\d{3}) (\d+)'

def parse_apache_log_line(logline):
    match=re.search(APACHE_ACCESS_LOG_PATTERN,logline)
    if match is None:
        raise Exception("Invalid logline: %s" % logline)
    return Row(
        ipAddress=match.group(1),
        clientIdentd=match.group(2),
        userId=match.group(3),
        dateTime=match.group(4),
        method=match.group(5),
        endpoint=match.group(6),
        protocol=match.group(7),
        responseCode=int(match.group(8)),
        contentSize=long(match.group(9)))
```

-----

## 第三步:把日志文件的所有行加载到Spark RDD中(弹性式分布数据集)

```
    conf=SparkConf().setAppName("Parse Apache Log").setMaster("local[*]")
    sc=SparkContext(conf=conf)
    path="hdfs://ncp162:8020/hsw/access_log"
    access_logs=sc.textFile(path).map(parse_apache_log_line).cache()
    print access_logs.count()
```

-----

## 第四步:计算统计通过GET请求返回的内容大小

```
    content_sizes=access_logs.map(lambda row:row.contentSize).cache()
    # 计算页面大小的平均值
    average_content_size=content_sizes.reduce(lambda x,y:x+y)/content_sizes.count()
    # 计算最小的页面
    min_content_size=content_sizes.min()
    # 计算最大的页面
    max_content_size=content_sizes.max()

    print "Content Size Statistics:\n Avg: %s\n Min: %s\n Max: %s" % (average_content_size,min_content_size,max_content_size)
```

-----

## 第5步:计算统计返回状态码

```
    # 计算各个返回状态码的个数
    response_code_to_count_pair_rdd = access_logs.map(lambda row: (row.responseCode, 1)).reduceByKey(lambda x, y: x + y)
    print response_code_to_count_pair_rdd.take(100)
```

-----

## 第6步:展示访问这个服务端超过N次的IP列表

```
    n=10
    ip_addresses_rdd=access_logs.map(lambda row:(row.ipAddress,1)).reduceByKey(lambda x,y:x+y)\
        .filter(lambda s:s[1]>n).map(lambda s:Row(ip_address=s[0]))
    print ip_addresses_rdd.collect()
    ip_addresses_dataframe=sqlContext.createDataFrame(ip_addresses_rdd)
    print ip_addresses_dataframe.rdd.toDebugString()
```

-----

## 第7步:研究统计endpoints信息

```
    endpoint_counts_rdd=access_logs.map(lambda row:(row.endpoint,1))\
        .reduceByKey(lambda x,y:x+y).map(lambda s:Row(endpoint=s[0],num_hits=s[1]))
    print endpoint_counts_rdd.collect()
    top_endpoints_array=endpoint_counts_rdd.takeOrdered(10,lambda row:-1*row.num_hits)
        top_endpoints_dataframe=sqlContext.createDataFrame(sc.parallelize(top_endpoints_array))
```

-----

## 附加:在Python中用SQL

+ 一个dataframe能够被注册成一个临时的SQL表
+ 然后你能够争对数据做SQL查询

