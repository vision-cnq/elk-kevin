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

### 2.2 ElasticSearch数据结构

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


### 2.3 CURL命令
在linux命令行中执行HTTP协议的请求

访问网页  

    curl www.baidu.com
    
显示响应的头信息  

    curl -i www.baidu.com
    
显示一次HTTP请求的通信过程  

    curl -v www.baidu.com

执行GET/POST/PUT/DELETE操作

    curl -X GET/POST/PUT/DELETE url

CURL操作es案例
```
    curl -X PUT http://master:9200/demo
```

   























