---
# Documentation: https://wowchemy.com/docs/managing-content/

title: "Big Data Realtime Project"
summary: "GULI mall realtime user behavior analysis"
authors: [Xin LI]
tags: [Kafka, SparkStreaming, HBase, ElasticSearch, Kibana, SpringBoot]
categories: [Big Data]
date: 2020-11-26T15:48:39+01:00

# Optional external URL for project (replaces project detail page).
external_link: ""

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "project structure"
  focal_point: ""
  preview_only: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---
### 1. demand analysis
Big data processing contains offline analysis architecture and real-time processing architecture.
 - Offline demand
Generally, reports and other data are generated based on the data of the previous day. Although there are many statistical indicators and reports, they are not sensitive to timeliness.
{{< figure src="Picture1.png" title="offline architecture" >}}

 - Real-time demand
Mainly focus on the real-time monitoring of the data of the day, usually the business logic is simpler than offline requirements, and there are fewer statistical indicators, but more attention is paid to the timeliness of the data and the interactivity of users.
{{< figure src="Picture2.png" title="real-time architecture" >}}

### 2. mock data
#### 2.1 created mock module and several mock classes to randomly generate logs.
<p>this module simulated users' every click event: add favor, add comment, add cart, click item, place an order etc; simulated users' geolocation, application versions, phone models etc; and also the time.</p>

#### 2.2 created logUploader class to send the generated log to the server via http.

### 3 start the data collection server
#### 3.1 spring boot
Spring Boot is a new framework provided by the Pivotal team. Its design purpose is to simplify the initial setup and development process of new Spring applications.
The framework uses a specific way to configure, so that developers no longer need to define boilerplate configurations.
Benefits of using Spring boot:
1. Embedded Tomcat, no need for external Tomcat
2. No longer need those cookie-cutter, cumbersome xml files.
3. It is more convenient to integrate with various third-party tools (mysql, redis, elasticsearch, dubbo, etc.), and only need to maintain a configuration file.

#### 3.2 create controller
Controller is used to process http requests.
Peusocode is like below:
```java
// @Controller
@RestController   // equals to: @Controller + @ResponseBody
public class LoggerController {
    //    @RequestMapping(value = "/log", method = RequestMethod.POST)
    //    @ResponseBody  //表示返回值是一个 字符串, 而不是 页面名
    @PostMapping("/log")  // 等价于: @RequestMapping(value = "/log", method = RequestMethod.POST)
    public String doLog(@RequestParam("log") String log) {
        System.out.println(log);
        return "success";
    }

    /**
     * 业务:
     *
     * 1. 给日志添加时间戳 (客户端的时间有可能不准, 所以使用服务器端的时间)
     *
     * 2. 日志落盘
     *
     * 3. 日志发送 kafka
     */
}
```

#### 3.3 send logs to Kafka
add Kafka configuration in Application.properties file.

    spring.kafka.bootstrap-servers=hadoop201:9092,hadoop202:9092,hadoop203:9092
    spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
    spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer
    
#### 3.4 deploy data collection server in Linux
    
    start bash :
    java -jar gmall-logger-1.0-SNAPSHOT.jar >/dev/null 2>&1 &
    
### 4. nginx Load Balancer
<p>Nginx, a high-performance HTTP and reverse proxy server, is characterized by less memory and strong concurrency. In fact, the concurrency of nginx does perform better in the same type of web server. The users of the nginx website in mainland China include: Baidu, JD, Sina, NetEase, Tencent, Taobao, etc.</p>

### 5. Use Spark Streaming to build a real-time processing module
<p>What is DAU:
1. Generally: The user who opens the application is the active user, regardless of the user's usage. A device opened multiple times a day will be counted as an active user. That is, you only need to open statistics for the first time
2. Game users: The number of users who open/log in to the game every day (definition for game DAU)
We adopt the first definition of daily active user, DAU statistical ideas:
1. Read user startup log from Kafka
2. Only keep the user's first startup record on the same day, filter out other startup records: with the help of Redis
3. Then save the first startup record in hbase for other applications to query</p>

#### 5.1 read data from Kafka

```scala
object DauApp {
    def main(args: Array[String]): Unit = {
        val conf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("DauApp")
        val ssc = new StreamingContext(conf, Seconds(5))
        val sourceStream: InputDStream[(String, String)] =
            MyKafkaUtil.getKafkaStream(ssc, GmallConstant.TOPIC_STARTUP)
        
        // 1. modify data structure
        val starupLogDSteam = sourceStream.map {
            case (_, log) => JSON.parseObject(log, classOf[StartupLog])
        }
        // 2. save to redis
        starupLogDSteam.foreachRDD(rdd => {
            rdd.foreachPartition(it => {
                val client: Jedis = RedisUtil.getJedisClient
                it.foreach(startupLog => {
                    // save to Redis value type set, store uid
                    val key = "dau:" + startupLog.logDate
                    client.sadd(key, startupLog.uid)
                })
                client.close()
            })
        })
        ssc.start()
        ssc.awaitTermination()
    }
}

```

#### 5.3 data cleaning
data deduplication using Redis, because for DAU we only need the the first login message for each user.
```
var filteredStartupLogDStream: DStream[StartupLog] = startupLogDStream.transform(rdd => {
            
            val client: Jedis = RedisUtil.getJedisClient
            val uidSet: util.Set[String] = client.smembers(GmallConstant.REDIS_DAU_KEY + ":" + new SimpleDateFormat("yyyy-MM-dd").format(new Date()))
            val uidSetBC: Broadcast[util.Set[String]] = ssc.sparkContext.broadcast(uidSet)
            client.close()
            
            rdd.filter(startupLog => {
                val uids: util.Set[String] = uidSetBC.value
                // 返回没有写过的
                !uids.contains(startupLog.uid)
            })
        })
        
        // 2.3 批次内去重:  如果一个批次内, 一个设备多次启动(对这个设备来说是第一个批次), 则前面的没有完成去重
        filteredStartupLogDStream = filteredStartupLogDStream
            .map(log => (log.uid, log))
            .groupByKey
            .flatMap {
                case (_, logIt) => logIt.toList.sortBy(_.ts).take(1)
            }
```    


### 6. write to Elasticsearch
why choose ES?
{{< figure src="Picture3.png" title="comparision between databases" >}}

common operations
```    
# 查询健康状态
GET /_cat/health?v
# 查看所有的索引
GET /_cat/indices?v
# 查看索引分片情况

# 增加一个索引
PUT /movie_index  

# 删除一个索引
DELETE /movie_index

# 新增文档
PUT /movie_index/movie/1
{ "id":1,
  "name":"operation red sea",
  "doubanScore":8.5,
  "actorList":[  
      {"id":1,"name":"zhang yi"},
      {"id":2,"name":"hai qing"},
      {"id":3,"name":"zhang han yu"}
  ]
}
PUT /movie_index/movie/2
{
  "id":2,
  "name":"operation meigong river",
  "doubanScore":8.0,
  "actorList":[  
      {"id":3,"name":"zhang han yu"}
  ]
}

PUT /movie_index/movie/3
{
  "id":3,
  "name":"incident red sea",
  "doubanScore":5.0,
  "actorList":[  
      {"id":4,"name":"zhang chen"}
  ]
}

# 根据 _id 查找
GET /movie_index/movie/1

# 修改某一个 field 的值
POST /movie_index/movie/1/_update
{
  "doc": {
      "doubanScore": 10
  }
}

# 搜索  匹配  过滤

# 分词匹配
GET /movie_index/movie/_search
{
  "query": {
    "match": {
      "name": "sea"
    }
  }
}
# 子查询匹配, 层级结构
GET /movie_index/movie/_search
{
  "query": {
    "match": {
      "actorList.name": "zhang"
    }
  }
}
# 短语匹配: 不拆词  比如查询人名
GET /movie_index/movie/_search
{
  "query": {
    "match_phrase": {
      "name": "operation meigong"
    }
  }
}

# 过滤

## 查询后过滤
GET /movie_index/movie/_search
{
  "query": {
    "match": {
      "name": "red sea"
    }
  }
  , "post_filter": {
    "term": {
      "actorList.id": 4
    }
  }
}
## 查询前过滤
GET /movie_index/movie/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "actorList.name": "zhang"
        }
      }
      , "must": [
        {"match": {
          "actorList.id": 3
        }},{
          "match": {
            "actorList.name": "hai"
          }
        }
      ]
    }
  }
}

# 排序
GET /movie_index/movie/_search
{
  "query": {
    "match": {
      "actorList.name": "zhang"
    }
  }
  , "sort": [
    {
      "doubanScore": {
        "order": "asc"
      }
    }
  ]
}

# 分页
GET /movie_index/movie/_search
{
  "from": 1
  , "size": 2
}

# 高亮
GET /movie_index/movie/_search
{
  "query": {
    "match": {
      "name": "sea"
    }
  }
  , "highlight": {
    "fields": {
      "name": {}
    }
  }
}

# 聚合
GET /movie_index/movie/_search
{
  "aggs": {
    "groupby_actor": {
      "terms": {
        "field": "actorList.name.keyword",
        "size": 10
      }
      , "aggs": {
        "avg_douban": {
          "avg": {
            "field": "doubanScore"
          }
        }
      }
    }
  }
}
```    
#### 6.2 considerations about designing ES index
- Type of each field
- Index type:
    + Need index, also need word segmentation (type=text):
        For example, title, product name, category name
    + Need index, but no word segmentation (type=keyword)
        Type id, date, quantity, age, various id
    + Neither index nor word segmentation is required (index=false)
        Will not be used for conditional filtering, desensitized fields, 138****0101
- Be sure to define the mapping before saving the data

#### 7 write data to HBase
use phoenix, phoenix is the sql plugin of HBase
code can be like below:
```    
    rdd.saveToPhoenix(
        "GMALL_DAU",
        Seq("MID", "UID", "APPID", "AREA", "OS", "CHANNEL", "LOGTYPE", "VERSION", "TS", "LOGDATE", "LOGHOUR"),
        zkUrl = Some("hadoop201,hadoop202,hadoop203:2181"))
})
```    

#### 8. use Kibana to query DAU and hourly details

query like below:
```  
GET /gmall_dau/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "logDate": "2019-05-15"
        }
      }
    }
  }
}
GET /gmall_dau/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "logDate": "2019-05-15"
        }
      }
    }
  }
  , "aggs": {
    "groupby_hour": {
      "terms": {
        "field": "logHour",
        "size": 24
      }
    }
  }
}
```  

### 9. visualization
{{< figure src="Picture4.png" title="final interface" >}}

