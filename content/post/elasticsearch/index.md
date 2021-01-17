---
title: Why use Elasticsearch?

summary: Elasticsearch is a distributed, high-scalable, high-real-time search and data analysis engine

# Link this post with a project
projects: []

# Date published
date: "2020-10-10T00:00:00Z"

# Date updated
lastmod: "2020-10-10T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

authors:
- admin

tags:
- Elasticsearch

categories:
- big data
---


- why use es？
|| redis  | mysql  | es（只有es支持全文检索）  | hbase  | hadoop/hive  |
|---|---|---|---|---|---|
|  容量 | 低  |  中 | 较大  | 海量  |海量|
| 查询时效性  |  极高 | 中等  |  较高（因为es会建很多索引，所以数据量会增多，拿空间换时间的办法） | 中等写比读快 |低（写磁盘）  |
| 查询灵活性  |  较差kv结构数据库 | 非常好支持sql  | 较好关联查询较弱，但是可以全文检索DSL语言可以处理过滤，匹配，排序但是不能join  | 较差kv结构数据库主要靠rowkeyscan的话性能不行  |非常好支持sql|
|写入速度|极快|中等|较快不直接往磁盘写，先写入内存，之后再往磁盘写放弃了一致性|较快|慢｜
|一致性/事务弱 |强 |弱| 弱| 弱|
<p>
总结  mysql扩展性差，几乎没有超过100台业务还是往mysql放 设计明细操作，都可以放es单日数据量TB以下可以  
综上，在实际环境中，需要一种能够容纳较大规模数据且交互性好的数据库
mysql交互性好，但是容量有限
hbase支持海量数据，但是查询的灵活度不足，hbase近实时查询
</p>


- ES

支持PB级数据的搜索，wiki和百度都用es作为搜索引擎<br>
特点：全文检索（自动切词），模糊查询（打错也能帮你纠正），数据分析<br>

es天然集群，一个机器也是一个集群，非常容易扩展，<br>
只需要在三个节点的配置文件中指定node几和ip地址，还有报道中心的ip（一个就好）<br>
天然分片，不管数据大小，都把数据分成多个shard，每个shard在不同的节点<br>
天然索引，<br>
不同于mysql，mysql主键是索引或者主动建索引<br>
es所有field都有索引，而且对text的field还会对每个字做索引<br>
不要索引需要说明<br>

启动：bin/elasticsearch >/dev/null 2>&1 &

<p>es根据你发给他的第一个数据的格式推断数据格式</p>

mysql和es的各自对应：
|mysql| es 5.x| es 6.x |es 7.x|
database |index |index |index|
table |type |只有一个type| 废弃|
row| document |document(java bean对象)| docuement|
column| field |field| field|

- 索引
    + mysql	B+tree（正排索引，通过表的id去找对应数据）
    + hbase	LSM tree（预写日志，排序，合并）
    + ES		倒排索引（对每个字建索引，可以通过每个字对应到整首诗）

- es的字符串有两种类型：
    + text 分词 全文匹配	占空间大
    + keyword 不分词 精确匹配 和作为聚合字段	占空间小
你的字符串，es会存两份，text的和keyword的。
所以尽量自己定义好，不要让它自己推断。
es和mysql不一样，只要不特殊说明，必定建索引，说明“index”: false


es语法：
# nodes of cluster
GET /_cat/nodes

#index of cluster
GET /_cat/indices?v

# create index
PUT /movie_index

#add document in index
   # every document should have an id
PUT movie_index/movie/1         movie是type，只能有一个；1相当于行id
{
  "id": 1,
  "name": "operation red sea",
  "doubannScore": 8.5,
  "actorList": [
    {"id":1, "name":"zhangyi"},
    {"id":2, "name": "haiqing"}
    ]
}
# query
GET movie_index/movie/_search
# query a specific id
GET movie_index/movie/1

#modify document，complete overwrite
put /movie_index/movie/1
{
  "id":100
}

#modify specific field
post /movie_index/movie/2/_update
{
  "doc":{
    "id":200
  }
}
# delete a specific document
delete movie_index/movie/100
全文索引，name里面有sea的都会找出来
GET movie_index/movie/_search
{
  "query":{
    "match":{
      "name": "sea"
    }
  }
}
查询演员名字有zhang或有yi的
GET movie_index/movie/_search
{
  "query":{
    "match":{
      "actorList.name":"zhang yi" //会分词，然后找有zhang和有yi的
    }
  }
}
ps: match换成match_phrase就不会分词了
模糊查询
GET movie_index/movie/_search
{
  "query":{
    "fuzzy":{
      "name": "saa"
    }
  }
}
查询后过滤
GET /movie_index/movie/_search
{
  "query": {"match": {"name": "red" }},
  "post_filter": {"term": { "actorList.id": "3"}
  }
}
查询前过滤
GET movie_index/movie/_search
{
  "query": {
    "bool": {
      "filter": [
        {"term": {"actorList.id": 3}},
        {"term": {"actorList.id": 1}}], //演员又有1又有3
      "must": {"match": { "name": "zhang"}} 
    }
  }
}
范围查询
GET movie_index/movie/_search
{
  "query": {
    "bool": {
      "filter": {
        "range": {
          "doubanScore": {
            "gt": 5,
            "lt": 9
          }
        }
      }
    }
  }
}
排序
GET movie_index/movie/_search
{
  "query":{"match": {"name":"red operation"}},
   "sort": [ {"doubanScore": { "order": "desc"}} ]
}
分页
GET movie_index/movie/_search
{
  "query": { "match_all": {} },
  "from": 1,
  "size": 1
}
返回指定字段
GET movie_index/movie/_search
{
  "query": { "match_all": {} },
  "_source": ["name", "doubanScore"]
}
聚合 
默认几count
GET movie_index/movie/_search
{
  "aggs": {
    "groupby_actor": {
      "terms": {
        "field": "actorList.id"
      }
    }
  }
}
指定聚合条件
GET movie_index/movie/_search
{
  "aggs": {
    "groupby_actor_id": {
      "terms": {
        "field": "actorList.id" ,
        "order": {
          "avg_score": "desc"
          }
      },
      "aggs": {
        "avg_score":{
          "avg": {
            "field": "doubanScore"
          }
        }
       }
    }
  }
}


