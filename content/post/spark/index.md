---
title: Some advanced python features
subtitle: list, dict, set, collections, yield, decrator

summary: Summarized some advanced python features which can make your code more efficient and pythonic

# Link this post with a project
projects: []

# Date published
date: "2020-10-11T00:00:00Z"

# Date updated
lastmod: "2020-10-11T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

authors:
- admin

tags:
- Spark

categories:
- big data
---

- overview
<p>
spark is distributed computing framework
bunch of apis to process data
high level moudules: sparksql, sparkstreaming, mllib..
well integrated with Python, Scala, Java..
use HDFS as file system
CCA175考试需要掌握Core spark 和 SparkSQL ，以及一些streaming
</p>
<p>
在Spark官方文档中，需要重点看RDD operations=》tranformations 和 actions
https://spark.apache.org/docs/2.4.0/rdd-programming-guide.html#transformations
https://spark.apache.org/docs/2.4.0/rdd-programming-guide.html#actions
</p>

- 启动spark
练习的时候不建议用local模式，不利于熟悉考试环境，而是



启动的时候比较重要的参数有：num_executors(默认两个), executor_memories

- RDD
in-memory, distributed, resilient<br>
rdd和scala的collection的区别就是rdd是in-memory，distributed，resilient，他们的方法都是一样的

- Create RDDs from HDFS
    + hadoop fs -ls /xxxx 查看文件
    + hadoop fs -tail /xxx 查看文件样例，结构
    + 从HDFS: val orders = sc.textFile("/public/retail_db/orders")
    + orders是RDD数据，他不像LIST可以通过(下标)来获取，而是orders.first,或者orders.take(10)
    + 从本地文件: val prodctRaw = scala.io.Source.fromFile("data/retail_db/products/part-00000").getLines.toList
        val productRDD = sc.parallize(productRaw)

- RDD
    + 一些算子是lazy evaluation 不会执行计算，只是记录血缘
    + action算子才真正执行计算，才会在spark ui界面显示job，以及对应的DAG图
    + 比如collect，count，first ，take，takeSample
    + DAG(Directed Acyclic Graph)叫做有向无环图，原始的RDD通过一系列的转换就就形成了DAG，根据RDD之间的依赖关系的不同			将DAG划分成不同的Stage，对于窄依赖，partition的转换处理在Stage中完成计算。对于宽依赖，由于有Shuffle的存在，只能在parent RDD处理完成后，才能开始接下来的计算，因此宽依赖是划分Stage的依据。
    + orders.todebugString 可以展示所有的依赖


- SparkSQL
读文件
    ```scala
val ordersDF = sqlContext.read.json("/public/retail_db_json/orders") 或 sqlContext.load("xxxx", "josn")
ordersDF.show
ordersDF.select("order_id")
    ```

下面拿一个订单的例子来介绍一些比较重要的RDD operation:

    ```scala
tranformation
string manipulation
str.split(",")
str.substring(0,10)
str.contans("a")
str.replace("/", "-")
map
val ordersPairedRDD = orders.map(order => {
val o = order.split(",")
(o(0).toInt, o(1).substring(0, 10).replace("-", "").toInt)
})
flatMap
flatMap(ele => ele.split(" "))

filter
// 从orders中过滤出2013年9月的并且状态是完成或者关闭的订单
orders.filter(order => {
val o = order.split(",")
(o(3) == "COMPLETE" || o(3) == "CLOSED") && (o(1).contains("2013-09"))
}) 
join
inner join
// 把订单map成（订单id，日期）
val ordersMap = orders.map(order => {
(order.split(",")(0).toInt, order.split(",")(1).substring(0, 10))
})
// 把订单items map成（订单id，这个item的总价）
 val orderItemsMap = orderItems.map(orderItem => {
val oi = orderItem.split(",")
(oi(1).toInt, oi(4).toFloat)
})
// join默认就是inner join操作
 val ordersJoin = ordersMap.join(orderItemsMap)

//结果如下所示：可以看出join操作把两个（k,v）的v用一个tuple组合在一起，k是共同的k
Array((41234,(2014-04-04,109.94)), 
   (65722,(2014-05-23,119.98)), //多个65722订单，因为该订单有多个items
   (65722,(2014-05-23,400.0)), 
   (65722,(2014-05-23,399.98)), 
   (65722,(2014-05-23,199.95)), 
   (65722,(2014-05-23,199.98)), 
   (28730,(2014-01-18,299.95)), 
   (28730,(2014-01-18,50.0)), 
   (68522,(2014-06-05,329.99)), 
   (23776,(2013-12-20,199.99)))
left outer join
val ordersMap = orders.map(order => {
  (order.split(",")(0).toInt, order)
})
val orderItemsMap = orderItems.map(orderItem => {
  val oi = orderItem.split(",")
  (oi(1).toInt, orderItem)
})
val ordersLeftOuterJoin = ordersMap.leftOuterJoin(orderItemsMap)
//结果
(9336,(9336,2013-09-22 00:00:00.0,11662,COMPLETE,Some(23297,9336,627,1,39.99,39.99)))
(18872,(18872,2013-11-19 00:00:00.0,597,PENDING_PAYMENT,Some(47201,18872,365,5,299.95,59.99)))
(18872,(18872,2013-11-19 00:00:00.0,597,PENDING_PAYMENT,Some(47202,18872,1014,5,249.9,49.98)))
(18872,(18872,2013-11-19 00:00:00.0,597,PENDING_PAYMENT,Some(47203,18872,403,1,129.99,129.99)))
(18872,(18872,2013-11-19 00:00:00.0,597,PENDING_PAYMENT,Some(47204,18872,1004,1,399.98,399.98)))
(18872,(18872,2013-11-19 00:00:00.0,597,PENDING_PAYMENT,Some(47205,18872,703,2,39.98,19.99)))
(53850,(53850,2014-06-29 00:00:00.0,884,PENDING,Some(134632,53850,1073,1,199.99,199.99)))
(53850,(53850,2014-06-29 00:00:00.0,884,PENDING,Some(134633,53850,1073,1,199.99,199.99)))
(53850,(53850,2014-06-29 00:00:00.0,884,PENDING,Some(134634,53850,191,2,199.98,99.99)))
(53850,(53850,2014-06-29 00:00:00.0,884,PENDING,Some(134635,53850,572,2,79.98,39.99)))
(12420,(12420,2013-10-09 00:00:00.0,449,PENDING,None))

some表示无值的可能性，也有None（表示这个订单没有关联的订单item），
我们可以用这个操作来找出所有没有订单	item关联的订单，同理rightOuterJoin也能实现。

aggregation
全局aggregation 
.count
.countbyKey
.reduce
用reduce实现sum

用reduce实现max

combiner
combiner的作用是预聚合，在shuffle之前先在分区内按key进行聚合
尽量用reducebyKey和aggregatebykey代替groupbyKey，因为后者没有用combiner，性能不高
aggregatebyKey可以在分区内和分区间使用不同的运算，reducebyKey是相同的运算

sort
    val products = sc.textFile("/public/retail_db/products")
val productsMap = products.
  	map(product => (product.split(",")(1).toInt, product))
// 按照categoryid，和 productprice的降序排序
val productsMap = products.
  filter(product => product.split(",")(4) != "").
  map(product => ((product.split(",")(1).toInt, -product.split(",")(4).toFloat), product)) //把要排序的两个值作为tuple放在key，就能按照多个值排序了，并且在值前面放-可以变成降序排序

val productsSortedByCategoryId = productsMap.sortByKey().map(rec => rec._2) // 最后通过map把排序的key去掉
global ranking, 全局取前10个价格最高的商品
一个办法是sortbykey 价格之后，take（10）
                      val productsSortedByPrice = productsMap.sortByKey(false)
productsSortedByPrice.take(10).foreach(println)
最好的办法是 直接takeordered10
products.
  filter(product => product.split(",")(4) != "").
  takeOrdered(10)(Ordering[Float].reverse.on(product => product.split(",")(4).toFloat)).
  foreach(println)
ranking， 获取每个category中价格最高的前10个商品

val products = sc.textFile("/public/retail_db/products")
val productsMap = products.
  filter(product => product.split(",")(4) != "").
  map(product => (product.split(",")(1).toInt, product))

val productMapGBK = productsMap.groupByKey

val productMapGBKSort = productMapGBK.map(res => (res._1, res._2.toList.sortBy(product => -product.split(",")(4).toFloat)))

val productMapGBKTopN = productMapGBKSort.map(res => {
  var l:List[String] = List()
  var i = 0
  while(i <10){
    l = l :+ res._2(i)
    i += 1
  }
  (res._1, l)
})

val result = productMapGBKTopN.flatMap(res => res._2)
获取在2013年8月订过，但是2013年9月没定过的客户
// Get all customers who placed orders in 2013 August but not in 2013 September
val customer_201308_minus_201309 = customers_201308.map(c => (c, 1)).
  leftOuterJoin(customers_201309.map(c => (c, 1))).
  filter(rec => rec._2._2 == None).
  map(rec => rec._1).
  distinct
保存到HDFS
rdd.saveasTextFile("xxxxx")
    ```


