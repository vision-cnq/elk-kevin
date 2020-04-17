# ElasticSearch

## 1、ElasticSearch简介
ElasticSearch是一个基于Lucene的搜索服务器，它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful接口。
    
## 2、ElasticSearch概念

### 2.1 ElasticSearch核心概念

Near Realtime（NRT）

    Elasticsearch是一个近乎实时的搜索平台。从索引文档到可以搜索的时间只有轻微的延迟（通常是1秒）。

Cluster（集群）

    集群，集群是一个或多个节点(服务器)的集合，它们共同保存你的整个数据，并提供跨所有节点的联合索引和搜索功能。
    集群名称是唯一的，建议命名（es-dev、es-prod、es-test）
   
Node（节点）

    节点是单独的服务器，属于集群中的一部分，节点由名称标识，默认是启动是随机分配UUID，但建议自定义名称，利于管理运维。

Shard（分片）  

    分片，Elasticsearch可以将一个index中document数据切分成多个shard，分布在多台服务器上存储，每个shard都是一个lucene index，
    最多能有document一样多。 
Shard重要性：  
    1.允许水平分割/扩展内容卷
    2.允许跨分片（多个节点）分布和并行操作，从而提高性能与吞吐量。

Replica（副本）  

    副本，replica主要用来保证高可用（故障转移）、数据备份、增强高吞吐的并行搜索。
Replica重要性：  
    1.提供了高可用，副本永远都是分配到非主分片相同节点上。
    2.允许扩展搜索量/吞吐量，搜索可以在所有副本上并行执行。   

> 注：每个索引都可以分成多个分片，索引也可以被复制0到多次，一旦被复制，每个索引都具有主分片和副本分片。<br/>
    在创建索引时，可以为每个索引定义分配和副本的数量，创建索引后可以修改副本数量，但不要能修改分片的数量。<br/>
    默认情况下，每个索引都分配5个主分片和1个副本分片，如果集群存在两个以上节点，则索引会有5个主分片与5个副本分片，每个索引总计有10个分片。

### 2.2 ElasticSearch的架构

* Gateway层  
ES用来存储索引文件的文件系统，主要是用来对数据进行长持久化以及整个集群重启之后可以通过gateway重新恢复数据。
底层文件系统例如：本地磁盘、共享存储（做snapshot的时候需要用到）、hadoop的HDFS分布式存储、亚马逊的S3。





### 2.3 ElasticSearch数据结构

Index（索引） 

    索引，是一堆document的集合，类似与mysql数据库中的database。

Type（类型）

    类型，一个index只能有一个type，类似mysql的table表，es可以在Index中建立type（table），通过mapping进行映射

Document（文档）
    
    文档，es中的最小数据单元，es存储的数据是文档型的（json格式），一条数据对应一篇文档即相当mysql中的一行数据，
    一个文档中可以有多个字段（等同mysql一行有多列）。    

Field

    es中一个文档对应多个列（等同mysql多个列）

Mapping

    es中mapping有动态识别功能（等同于mysql的schema）
    
indexed
    
    建立索引，mysql中一般对经常使用的列增加相应的索引用于提高查询速度，es默认会加上索引，
    除非你特殊指定不建立索引只是进行存储用于展示。
    
Query DSL
    
    类似mysql的sql语句，只是es中使用的是json格式的查询语句。
    
GET/PUT/POST/DELETE

    分别类似mysql的select/update/delete

### 2.4 CURL命令操作

在linux命令行中执行HTTP协议的请求

#### 2.4.1 CURL命令

访问网页  

    curl www.baidu.com
    
显示响应的头信息  

    curl -i www.baidu.com
    
显示一次HTTP请求的通信过程  

    curl -v www.baidu.com

执行GET/POST/PUT/DELETE操作

    curl -X GET/POST/PUT/DELETE url

#### 2.4.2 CURL操作ES案例

1.查看集群健康
```
    curl -X GET master:9200/_cat/health?v
或者
    curl -X GET http://master:9200/_cat/health?v
```
2.查看集群节点列表
```
    curl -X GET master:9200/_cat/nodes?v
```
3.查看全部索引
```
    curl -X GET master:9200/_cat/indices?v
```
4.添加索引
```
    curl -X PUT http://master:9200/demo
```
5.查看索引配置信息
```
    curl -X GET master:9200/demo/_settings
```
6.删除索引
```
    curl -X DELETE master:9200/demo
```
7.批量查询
```
    curl master:9200/_mget -d  {
        "docs":[
            {
                "index":"demo",
                "_type":"user",
                "_id":1
            },
            {
                "index":"demo",
                "_type":"user",
                "_id":2
            }
        ]
    }
```

### 2.5 ElasticSearch API操作之索引（在Kibana的Dev Tools中操作）

1.添加索引时设置分片与副本

    PUT /demo1
    {
      "settings":{
        "index":{
          "number_of_shards":3,
          "number_of_replicas":0
        }
      }
    }
2.查看索引配置

    GET /demo1/_settings
3.查看所有索引配置

    GET /_all/_settings
4.修改索引副本数量（分片创建之后不能再修改，但副本可以）

    PUT /demo1/_settings
    {
      "number_of_replicas":3
    }  
5.添加索引
   
    PUT /demo   
6.删除指定索引

    DELETE /demo
7.删除指定的多个索引

    DELETE /demo,demo1
8.删除匹配符索引

    DELETE /demo_*
9.删除所有索引

    DELETE /_all

### 2.6 ElasticSearch API操作之文档（在Kibana的Dev Tools中操作）

首次新增文档时会自动创建type类型，一个index在6.x版本之后只能有一个type。

不能单独在index之后创建type，错误示范：

    PUT /demo/user
    
####2.6.1 文档参数介绍

GET /demo/user/_search   
返回数据：
```
{
  "took" : 0,   // 查询花费时长（毫秒 ）
  "timed_out" : false,  // 是否超时
  "_shards" : { // 分片信息
    "total" : 5,    // 分片数量
    "successful" : 5,   // 成功的分片数量
    "skipped" : 0,  // 跳过的分片数量
    "failed" : 0    // 失败的分片数量
  },
  "hits" : {
    "total" : 2,    // 查询结果数量
    "max_score" : 1.0,  // 最大评分数，与search条件越相关，分数越高
    "hits" : [  // 查询结果的数据集合
      {
        "_index" : "demo",  // 索引名称
        "_type" : "user",   // 类型
        "_id" : "2",    // 文档id
        "_score" : 1.0, // 评分
        "_source" : {   // 存储字段数据集
          "name" : "Mrcao", // 字段
          "age" : 24,   // 字段
          "sex" : "男",  // 字段
          "hobby" : [   // 字段
            "java"
          ]
        }
      },
      {
        "_index" : "demo",
        "_type" : "user",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "kevin",
          "age" : 24,
          "sex" : "男",
          "hobby" : [
            "java",
            "scala"
          ]
        }
      }
    ]
  }
}
```

#### 2.6.2 添加操作
1.添加文档

    PUT /demo/user/1
    {
      "name":"kevin",
      "age":24,
      "sex":"男",
      "hobby":["java"]
    }
2.添加文档，使用POST，不指定ID，会默认生成一个20字符串的ID

    POST /demo/user
    {
      "name":"kevin",
      "age":24,
      "sex":"男",
      "hobby":["java","scala"]
    }

#### 2.6.3 查询操作
1.查询指定索引的所有数据（如果需要指定多个索引，以逗号","分隔）

    GET /demo/user/_search
2.查询指定索引的所有数据并设置查询超时

    GET /demo/user/_search?timeout=5s
3.查询索引指定ID获取所有元数据（列）

    GET /demo/user/1
4.查询索引指定ID获取指定元数据（列），例如获取name，如果需要指定多个列，以逗号"5.查询索引指定ID获取指定元数据（列），例如获取name，如果需要指定多个列，以逗号","分隔

    GET /demo/user/1?_source=name
5.查询多个索引，多个类型（如果需要查询所有索引，多个类型，将索引换成_all）
    
    GET /demo,demo2/user2,user/_search
6.分页查询（from：第几条数据，size：数据长度）

    GET /demo/user/_search?from=0&size=2
7.深度分页查询（scroll：缓存时间，size：查询数量。普通分页查询大分页非常耗费资源，可以用scroll，用于一批批的处理数据）    

    GET /demo/user/_search?scroll=2m&size=2
8.深度分页查询会返回一个scroll_id，在缓存时间内可以直接使用scroll_id查询，不用加其它条件

    GET /_search/scroll
    {
      "scroll_id" :"DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAm9FmhhQXd2ZzlsVEhHamVsTXpCazl2NWcAAAAAAAAJwRZoYUF3dmc5bFRIR2plbE16Qms5djVnAAAAAAAACb4WaGFBd3ZnOWxUSEdqZWxNekJrOXY1ZwAAAAAAAAm_FmhhQXd2ZzlsVEhHamVsTXpCazl2NWcAAAAAAAAJwBZoYUF3dmc5bFRIR2plbE16Qms5djVn"
    }
    
#### 2.6.4 修改操作    
1.更新文档（将之前存在的文档覆盖掉）

    PUT /demo/user/1
    {
      "name":"kevin",
      "age":24,
      "sex":"男",
      "hobby":["java","scala"]
    }
> 该方式为软删除，将旧版本标记为deleted，实际没有物理删除
  该条数据的_version会+1，再PUT，_version还继续+1，之后es的数据越来越多，才会物理删除这这些标记的数据。

2.更新文档（修改指定的文档某列），例如：修改age的值

    POST /demo/user/1/_update
    {
      "doc":{
        "age":23
      }
    }

#### 2.6.5 删除操作
1.删除指定文档

    DELETE /demo/user/1
> 通过id删除文档，其实是标记为deleted，等数据越来越大的时候ES才会进行物理删除。

2.删除指定索引

    DELETE /demo   
    
#### 2.6.6 批量查询文档操作
使用es提供过的Multi Get API可以通过索引名、类型名、文档id一次得到一个文档集合，文档可以来自一个索引库也可以是不同的索引库。
>一次需要查询多条数据时使用mget提高性能

1.批量查询，index与type相同，id不同  
方式一：   
```
GET /_mget
{
  "docs":[
    {
      "_index":"demo",
      "_type":"user", 
      "_id":1
    },
    {
      "_index":"demo",
      "_type":"user", 
      "_id":2
    }
    ]
}
```
方式二：
```
GET /demo/user/_mget
{
  "docs":[
    {
      "_id":1
    },
    {
      "_id":2
    }
    ]
}
```
方式三 ：
```
GET /demo/user/_mget
{
  "ids":[1,2]
}
```
2.批量查询，index与type相同，id不同，且指定字段  
方式一：
```
GET /_mget
{
  "docs":[
    {
      "_index":"demo",
      "_type":"user", 
      "_id":1,
      "_source":"name"
    },
    {
      "_index":"demo",
      "_type":"user", 
      "_id":2,
      "_source":["age","sex"]
    }
    ]
}
```
方式二：
```
GET /demo/user/_mget
{
  "docs":[
    {
      "_id":1,
      "_source":"name"
    },
    {
      "_id":2,
      "_source":["age","sex"]
    }
    ]
}
```
> 批量查询可以简化语句，同索引下的不用在docs中写索引名，直接在_mget前写入即可

#### 2.6.7 使用Bulk API 实现批量操作






### 3. Elasticsearch DSL常用语法







