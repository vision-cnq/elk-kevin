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

* ES分布式搜索引擎架构图

![img](https://note.youdao.com/yws/api/personal/file/18641221DE5742C6A165F6DF0BED04E3?method=download&shareKey=e5cc7a2b68b7957d171e3bb8addb09fb)

* Gateway层  
ES用来存储索引文件的文件系统，主要是用来对数据进行长持久化以及整个集群重启之后可以通过gateway重新恢复数据。
底层文件系统例如：本地磁盘、共享存储（做snapshot的时候需要用到）、hadoop的HDFS分布式存储、亚马逊的S3。

* Lucene Directory层  
Lucene的分布式框架，lucene是单价搜索引擎，为了满足ES分布式搜索引擎系统，需要做成一个分布式的运行框架来满足在每个节点上都运行Lucene进行相应的索引，查询以及更新。

* ES四大模块组件层  
    * Index Module：索引模块，对数据建立索引（倒排索引等）。
    * Search Module：搜索模块，对数据进行查询搜索。
    * Mapping：数据映射与解析模块，数据中每个字段会根据建立的表结构通过mapping进行映射解析。若没有建立表结构则默认根据数据类型推测数据结构后自动生成mapping，再根据该mapping解析数据。
    * River：第三方插件模块，可以通过自定义脚本将mysql等数据源通过格式化转换后同步到es集群中。

* 两大模块组件层
    * Discovery模块：ES是一个集群包含很多节点，相互发现对方节点（服务发现），组成集群进行选主（Master选举）。默认Zen，也能是EC2。
    * Script模块：ES支持多种脚本语言。（mvel、js、python、Etc.等）
    
* Transport层  
ES的通信接口Transport层，支持：Thrift、Memcached以及Http，默认为http，JMX是Java的一个远程监控管理框架，ES是通过Java实现的。

* RESTful接口层  
ES的访问接口，推荐Restful接口，直接发送http请求，方便后续使用nginx做代理、分发，包括可能会做权限的管理，http容易做这方面的管理。
使用Java客户端是直接调用API，在负载均衡与权限管理不太好做。


### 2.3 ElasticSearch数据结构

与Mysql对比理解

|          | 数据库   |    表    |   记录   |    列    | 
| :------: | :------: | :------: | :------: | :------: | 
|  MySQL   |    DB    |   Table  |   Row    |  Column  | 
|    ES    |  Index   |   Type   | Document |   Field  | 

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
    
#### 2.6.1 文档参数介绍

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
    
### 2.7 mget批量查询文档操作
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

### 2.8 使用Bulk API 实现批量操作

Bulk会将要处理的数据载入内存中，所以数据量是有限制的，最佳的数据量不是一个固定数值，取决于硬件、文档大小、复杂性、索引以及搜索的负载。  
Bulk一次最大处理数据量：一般建议是1000-5000个文档，大小建议为5-15MB，默认不能超过100M，可以在ES的配置文件（config下的elasticsearch.yml）中。

#### 2.8.1 Bulk格式
```
{action:{metadata}}\n
{requstbody}\n
```

> Bulk每一行就是一个操作。删除操作没有请求体。

action：(行为)  
 * create：文档不存在时创建。
 * update：更新文档。
 * index：创建新文档或替换已有文档。
 * delete：删除一个文档。
 > create与index的区别：如果数据存在。  
    使用create操作失败，且提示文档存在。  
    使用index操作则可以成功执行。  
    
metadata：_index，_type，_id。

requstbody：非必选。

#### 2.8.2 批量添加  
新增索引demo1，类型user，以及文档1,2,3，index那一行是文档索引，下面一行是文档中的数据内容
```
POST /demo1/user/_bulk
{"index":{"_id":1}}
{"name":"Kevin","age":24,"sex":"男"}
{"index":{"_id":2}}
{"name":"MrCao","age":24,"sex":"男"}
{"index":{"_id":3}}
{"name":"cao","age":23,"sex":"男"}
```

#### 2.8.3 批量获取
使用_mget进行批量获取
```
GET /demo1/user/_mget
{
  "ids":["1","2","3"]
}
```

#### 2.8.4 批量修改
修改demo1索引user类型下1,2文档内的数据
```
POST /demo1/user/_bulk
{"update":{"_index":"demo1","_type":"user","_id":1}}
{"doc":{"age":21}}
{"update":{"_index":"demo1","_type":"user","_id":2}}
{"doc":{"name":"xiaocao","age":22}}
```

#### 2.8.5 批量删除
删除demo1索引user类型下的1,2,3文档
```
POST /demo1/user/_bulk
{"delete":{"_index":"demo1","_type":"user","_id":1}}
{"delete":{"_index":"demo1","_type":"user","_id":2}}
{"delete":{"_index":"demo1","_type":"user","_id":3}}
或者
POST /_bulk
{"delete":{"_index":"demo1","_type":"user","_id":1}}
{"delete":{"_index":"demo1","_type":"user","_id":2}}
{"delete":{"_index":"demo1","_type":"user","_id":3}}
```

#### 2.8.6 批量混合操作
可以批量将各种增删改混合在一起同时执行，例如新增后修改然后删除别的再新增一个
```
POST /demo1/user/_bulk
{"create":{"_index":"demo1","_type":"user","_id":4}}
{"name":"小爱同学","age":5,"sex":"女"}
{"update":{"_index":"demo1","_type":"user","_id":4}}
{"doc":{"name":"小爱"}}
{"delete":{"_index":"demo1","_type":"user","_id":3}}
{"index":{"_index":"demo1","_type":"user"}}
{"name":"siri","age":7,"sex":"女"}
# 没有指定_id时，ES会自动生成一个随机id
```

### 2.9 ElasticSearch的版本控制

ElasticSearch采用了乐观锁来保证数据的一致性，当用户对Document（文档）操作时，并不需要对该Document（文档）作加锁和解锁的操作，只需要指定操作的版本即可。
当版本号一致时，ES允许该操作顺利执行。
当版本号存在冲突时，ES会提示冲突并抛出异常（VersionConflictEngineException异常）。

#### 2.9.1 ES版本号的取值范围
ElasticSearch的版本号的取值范围为1到2^63-1。

#### 2.9.2 ES使用版本控制
* 内部版本控制：使用的是_version。
* 外部版本控制：ES在处理外部版本号时会与对内部版本号的处理有些不同。
它不再是检查_version是否与请求中指定的数值是否相同,而是检查当前的_version是否比指定的数值小。
如果请求成功，那么外部的版本号就会被存储到文档中的_version中。

版本控制案例：
```
# 创建一个文档
PUT /demo/user/1
{
  "name":"kevin",
  "sex":"男",
  "hobby":["java"]
}
# 此时_version=1
```
内部版本控制（案例）：
```
# 更新版本并指定为version=1
PUT /demo/user/1?version=1
{
  "name":"kevin",
  "age":24,
  "sex":"男"
}
# 更新成功，此时_version=2

# 再次更新版本并还是指定version=1
PUT /demo/user/1?version=1
{
  "name":"kevin",
  "age":24,
  "sex":"男"
}
# 更新失败，因为版本不一致，抛出异常：reason": "[user][1]: version conflict, current version [2] is different than the one provided [1]"

```
外部版本控制（案例）：
```
# 更新时，判断_version是否比指定的数值小，如果是则保存外部版本号到version中
PUT /demo/user/1?version=4&version_type=external
{
  "name":"kevin",
  "age":24,
  "sex":"男"
}
# 更新前，version=2，指定的数值为4，判定通过，更新数据成功，此时version=4
```
  
#### 2.9.3 ES保证内、外部版本控制数据一致
为了保持_version与外部版本控制的数据一致
```
使用：version_type=external
```

### 2.10 ElasticSearch的mapping（映射）

ES自动创建了index，type，以及type对应的mapping。  

什么是映射：mapping定义了type中每个字段的数据类型以及这些字段如何分词等相关属性。

创建索引时，可以预先定义字段的类型以及相关属性，这样就能够把日期字段处理成日期，数字字段处理成数字，字符串字段处理成字符串值。

> 检索时用到的分析策略需要与建立索引时的分析策略相同，否则将导致数据不准确。

#### 2.10.1 支持的数据类型

1.核心数据类型  
  
    字符型（string）：
        1.text类型：用来索引长文本，建立索引之前会将这些文本进行分词，转化为词的组合，建立索引。允许ES检索这些词语，但不能用来排序与聚合。
        2.keyword类型：不需要进行分词，但可以被用来检索过滤、排序和聚合。keyword类型字段只能用本身来进行检索。 
    数字型：long，integer，short，byte，double，float
    日期型：date
    布尔型：boolean
    二进制型：binary
2.复杂数据类型

    数组类型(Array)：数组类型不需要专门指定数组元素的type
        字符型数组：["one","two"]
        整型数组：[1,2]
        数组型数组：[1,[2,3]] 等价于 [1,2,3]
        对象数组：[{"name":"kevin","age":24},{"name":"mrcao","age":24}]
    对象类型(Object)：_object_用于单个JSON对象。
    嵌套类型(Nested)：_nested_用于JSON数组。
3.地理位置类型
    
    地理坐标类型(Geo-point)：_geo-point_用于经纬度坐标。
    地理形状类型(Geo-Shape)：_geo-shape_用于类型多边形的复杂形状。
4.特定类型
    
    IPv4类型(IPv4)：_ip_用于IPv4地址。
    Completion类型(Completion)：_completion_提供自动补全建议。
    Token Count类型(Token count)：_token_count_用于统计做了标记的字段的index数目，该值会一直增加，不会因为过滤条件而减少。
    mapper-murmur3类型：通过插件，可以通过_murmur3_来计算index的hash值。
    附加类型(Attachment)：采用mapper-attcahments插件，可支持_attachments_索引。（例如Microsoft Office格式，Open Document格式，ePub，HTML等）。

#### 2.10.2 支持的属性    

"store":false   
> 是否单独设置此字段的是否存储而从_source字段中分离，默认是false，只能搜索，不能获取值

"index": true   
> 分词，不分词是：false，设置成false，字段将不会被索引
   
"analyzer":"ik" 
> 指定分词器，默认分词器为standard analyzer

"boost":1.23    
> 字段级别的分数加权，默认值是1.0

"doc_values":false  
> 对not_analyzed字段，默认都是开启，分词字段不能使用，对排序和聚合能提升较大性能，节约内存

"fielddata":{"format":"disabled"}   
> 针对分词字段，参与排序或聚合时能提高性能，不分词字段统一建议使用doc_value

"fields":{"raw":{"type":"string","index":"not_analyzed"}} 
> 可以对一个字段提供多种索引模式，同一个字段的值，一个分词，一个不分词
            
"ignore_above":100 
> 超过100个字符的文本，将会被忽略，不被索引

"include_in_all":true   
> 设置是否此字段包含在_all字段中，默认是true，除非index设置成no选项

"index_options":"docs"  
> 4个可选参数docs（索引文档号） ,freqs（文档号+词频），positions（文档号+词频+位置，通常用来距离查询），offsets（文档号+词频+位置+偏移量，通常被使用在高亮字段）分词字段默认是position，其他的默认是docs

"norms":{"enable":true,"loading":"lazy"}   
> 分词字段默认配置，不分词字段：默认{"enable":false}，存储长度因子和索引时boost，建议对需要参与评分字段使用 ，会额外增加内存消耗量

"null_value":"NULL"
> 设置一些缺失字段的初始化值，只有string可以使用，分词字段的null值也会被分词

"position_increament_gap":0 
> 影响距离查询或近似查询，可以设置在多值字段的数据上火分词字段上，查询时可指定slop间隔，默认值是100

"search_analyzer":"ik"  
> 设置搜索时的分词器，默认跟analyzer是一致的，比如index时用standard+ngram，搜索时用standard用来完成自动提示功能

"similarity":"BM25" 
> 默认是TF/IDF算法，指定一个字段评分策略，仅仅对字符串型和分词类型有效

"term_vector":"no"  
> 默认不存储向量信息，支持参数yes（term存储），with_positions（term+位置）,with_offsets（term+偏移量），with_positions_offsets(term+位置+偏移量) 对快速高亮fast vector highlighter能提升性能，但开启又会加大索引体积，不适合大数据量用

#### 2.10.3 映射的分类

动态映射：ES在文档中出现不存在的字段时，会利用动态映射来决定该字段的类型，并自动对该字段添加映射。

dynamic设置可以控制该行为，能接受以下选项：  
* true：默认值，动态添加字段
* false：忽略新字段
* strict：陌生字段，抛出异常。   

dynamic设置可以使用在根对象上或者object类型的任意字段上。

#### 2.10.4 创建mapping

创建mapping时，可以指定每个field是否需要mapping，默认为"index":"true"，mapping一旦创建，将不允许修改。
不能更新mapping，只能在创建index的时手动配置mapping，或者在添加新字段的时候新增mapping。

```
# 对demo2索引的user类型创建mapping
PUT /demo2
{
  "mappings":{
    "users":{
      "properties":{
        "name":{
          "type":"text",
          "index":true
        },  
        "age":{
          "type":"integer"
        },
        "sex":{
          "type":"keyword"
        },
        "createtime":{
          "type":"date",
          "format":"yyyy-MM-dd HH:mm:ss"
        },
        "mail":{
          "type":"text",
          "index":false
        }
      } 
    }
  }
}
# mappings：映射，properties：类型中的属性，"index":true设置为让其分词，如果是false则不分词
# "type":"text", "type":"integer"，"type":"keyword"这些是设置映射的类型，"type":"date",并且指定时间格式

```

新增字段并且设置mapping
```
PUT /demo2/_mapping/users
{
  "properties":{
    "new_field":{
      "type":"text",
      "index":false
    }
  } 
}
```

#### 2.10.5 查看mapping

```
GET /demo2/_mapping
```

## 3、 Elasticsearch DSL常用语法

数据准备
```
# 创建索引，设置分片副本，配置映射
PUT /demo
{
  "settings":{
    "number_of_replicas": 1, 
    "number_of_shards": 3
  },
  "mappings":{
    "user":{
      "properties":{
        "name":{"type":"text"},
        "age":{"type":"integer"},
        "sex":{"type":"text"},
        "remark":{"type":"text"},
        "create_time":{"type":"date","format":"yyyy-MM-dd||yyyy/MM/dd||yyyy-MM-dd HH:mm:ss||epoch_millis"}
      }
    }
  }
}

# 查看 mapping
GET /demo/user/_mapping

# 批量新增数据
POST /demo/user/_bulk
{"index":{"_index":"demo","_type":"user","_id":1}}
{"name":"kevin","age":24,"sex":"男","remark":"我是kevin","create_time":"2020-05-04"}
{"index":{"_index":"demo","_type":"user","_id":2}}
{"name":"cao","age":23,"sex":"男","remark":"我是cao","create_time":"2020-05-03"}
{"index":{"_index":"demo","_type":"user","_id":3}}
{"name":"mr.cao","age":22,"sex":"男","remark":"我是mr.cao","create_time":"2020-05-02"}
{"index":{"_index":"demo","_type":"user","_id":4}}
{"name":"kevin","age":21,"sex":"男","remark":"我是kevin","create_time":"2020-05-02"}
{"index":{"_index":"demo","_type":"user","_id":5}}
{"name":"coco","age":21,"sex":"女","remark":"我是coco","create_time":"2020-05-04"}
{"index":{"_index":"demo","_type":"user","_id":6}}
{"name":"co","age":20,"sex":"女","remark":"我是co","create_time":"2020-05-03"}
{"index":{"_index":"demo","_type":"user","_id":7}}
{"name":"coco","age":19,"sex":"女","remark":"我是coco","create_time":"2020-05-02"}

# 查询测试
# 查询name为kevin
GET /demo/user/_search?q=name:kevin

# 查询name为kevin且以age倒序
GET /demo/user/_search?q=name:kevin&sort=age:desc
```

### 3.1 DSL语句校验查询是否正确
```
# 查询前先校验语句是否正确
GET /demo/user/_validate/query?explain
{
  "query":{
    "match":{
      "name":"kevin"
    }
  }
}
```

### 3.2 DSL基本用法

模糊查询时使用match，精准查询时使用term。

term query：直接对关键词准确查找，该查询只适合keyword、numeric、date。

    term：查询某个字段中含有某个关键词的文档。
    terms：查询某个字段中含有多个关键词的文档。
    
match query：对所查找的关键词进行分词，在根据分词匹配查找。

    match_all：查询所有文档。
    multi_match：指定多个字段。
    match_phrase：短语匹配查询。

ES检索引擎会先分析查询字符串，从分析后的文本中构建短语查询，表示必须匹配短语中所有的分词，并保证各个分词的相对位置不变。
    
from：从第几条数据开始查询。
    
size：需要查询的个数。

sort：实现排序，desc降序，asc升序。

range：实现范围查询。（参数：from，to，include_lower，include_upper，boost）

    include_lower：是否包含范围的左边界，默认true。
    include_upper：是否包含范围的右边界，默认true。

wildcard：允许使用通配符*和?进行查询

    *：表示0或者多个字符
    ?：表示任意一个字符  

fuzzy：实现模糊查询

    value：查询的关键字
    boost：查询的权值，默认值1.0
    min_similarity：设置匹配的最小相似度，默认值0.5，对于字符串，取值0-1（包括0和1），对于数值，取值可能大于1，对于日期，取值为1d,1m等，1d为1天。
    prefix_length：指明区分词项的共同前缀长度，默认0.
    max_expansions：查询中的词项可以扩展的数目，默认可以无限大。

filter是不计算相关性的，同时可以cache，所以filter效率高于query。

范围过滤：

    gte：>=（大于等于）
    gt：>（大于）
    lte：<=（小于等于）
    lt：<（小于）
    
bool过滤

    must：必须满足的条件（and）
    should：可以满足也可以不满足的条件（or）
    must_not：不需要满足的条件（not）

### 3.3 term与terms查询

term query：直接对关键词准确查找，该查询只适合keyword、numeric、date。

    term：查询某个字段中含有某个关键词的文档。
    terms：查询某个字段中含有多个关键词的文档。

#### 3.3.1 指定单个关键词查询
term查询
```
# term指定查询name为kevin的数据
GET /demo/user/_search
{
  "query":{
    "term":{"name":"kevin"}
  }
}
```

#### 3.3.2 指定多个关键词查询
terms查询
```
# terms指定查询name为kevin与cao的数据
GET /demo/user/_search
{
  "query":{
    "terms":{
      "name":["kevin","cao"]
    }
  }
}
```

#### 3.3.3 分页查询

from：从第几条数据开始查询，size：需要查询几条数据

```
# 分页查询
GET /demo/user/_search
{
  "query":{
    "terms":{
      "name":["kevin","coco"]
    }
  },
  "from":0,
  "size":3
}
```

#### 3.3.4 查询时返回版本号
```
# 查询时返回版本号，默认version为false
GET /demo/user/_search
{
  "version":"true",
  "query": {
    "terms": {
      "name": ["kevin"]
    }
  }
}
```

### 3.4 match查询

match query：对所查找的关键词进行分词，在根据分词匹配查找。

    match_all：查询所有文档。
    multi_match：指定多个字段。
    match_phrase：短语匹配查询。

#### 3.4.1 查询全部（match_all）

match_all：查询所有文档。

```
# 查询所有用户
GET /demo/user/_search
{
  "query":{
    "match_all":{}
  }
}
```

#### 3.4.2 指定单个字段查询
```
# 指定查询，name为kevin且以age倒序
GET /demo/user/_search
{
  "query":{
    "match":{
      "name":"kevin"
    }
  },
  "sort":[
    {
      "age":"desc"
    }
  ]
}
```

#### 3.4.3 指定多个字段查询（multi_match）

multi_match：指定多个字段。

```
# 在多个字段中查询包含谁，kevin，coco的数据
GET /demo/user/_search
{
  "query":{
    "multi_match":{
      "query":"谁 kevin coco",
      "fields":["name","remark"]
    }
  }
}
# "query":"谁 kevin coco",查询的语句，fields查询的字段
```

#### 3.4.4 短语完全匹配查询（match_phrase）

match_phrase：短语完全匹配查询。  

ES检索引擎会先分析查询字符串，从分析后的文本中构建短语查询，表示必须匹配短语中所有的分词，并保证各个分词的相对位置不变。

```
# 精准匹配所有包含（完全匹配），必须包含
GET /demo/user/_search
{
  "query":{
    "match_phrase":{
      "remark":{
        "query":"我",
        "slop":1
      }
    }
  }
}
# "slop"是可调节因子，"slop":1则表示往后移一个单词也能匹配
```

#### 3.4.5 前缀匹配查询

match_phrase_prefix：前缀匹配查询

```
# 匹配name的开头是co
GET /demo/user/_search
{
  "query":{
    "match_phrase_prefix":{
      "name":{
        "query":"co"
      }
    }
  }
}
```

#### 3.4.6 分页查询

from：从第几条数据开始查询，size：需要查询几条数据

```
# 分页查询
GET /demo/user/_search
{
  "query":{
    "match":{
      "name":"kevin"
    }
  },
  "from":0,
  "size":2
}
```

#### 3.4.7 查询返回指定字段
```
# 查询name为kevin的name，age，create_time字段
GET /demo/user/_search
{
  "query":{
    "match":{
        "name":"kevin"
    }
  },
  "_source":[
    "name",
    "age",
    "create_time"
  ]
}
```

#### 3.4.8 控制加载的字段（选择性展示的字段）

includes：包含的字段展示  
excludes：不包含的字段展示
```
# 包含的字段展示
GET /demo/user/_search
{
  "query":{
    "match_all":{}
  },
  "_source":{
    "includes":["name","sex"]
  }
}

# 不包含的字段展示
GET /demo/user/_search
{
  "query":{
    "match_all":{}
  },
  "_source":{
    "excludes":["age","create_time"]
  }
}
```

#### 3.4.9 使用通配符*

```
# 展示字段名包含m的字段
GET /demo/user/_search
{
  "_source":{
    "includes":"*m*"
  },
  "query":{
    "match_all":{}
  }
}

# 展示字段名包含m的字段，并不展示name与最后为e的字段
GET /demo/user/_search
{
  "_source":{
    "includes":"*m*",
    "excludes":["name","*e"]
  },
  "query":{
    "match_all":{}
  }
}
```

#### 3.4.10 排序

sort：实现排序，desc降序，asc升序。

```
# 根据age做升序排序
GET /demo/user/_search
{
  "query":{
    "match_all":{}
  },
  "sort":{
    "age":{
      "order":"asc"
    }
  }
}
```

#### 3.4.11 查询范围性数据

range：实现范围查询。（参数：from，to，include_lower，include_upper，boost）

    include_lower：是否包含范围的左边界，默认true。
    include_upper：是否包含范围的右边界，默认true。
    
```
# 查询create_time为2020-05-01到05的数据
GET /demo/user/_search
{
  "query":{
    "range":{
      "create_time":{
        "from":"2020-05-01",
        "to":"2020-05-05"
      }
    }
  }
}

# 查询age为19到21的数据，不包含21
GET /demo/user/_search
{
  "query":{
    "range":{
      "age":{
        "from":19,
        "to":21,
        "include_lower":true,
        "include_upper":false
      }
    }
  }
}
```    

#### 3.4.12 wildcard模糊查询

wildcard：允许使用通配符*和?进行查询

    *：表示0或者多个字符
    ?：表示任意一个字符  
    
```
# 模糊匹配name中最右面为co的数据
GET /demo/user/_search
{
  "query":{
    "wildcard":{
      "name":"*co"
    }
  }
}

# 模糊匹配name中c与o之间还有一个值的数据
GET /demo/user/_search
{
  "query":{
    "wildcard":{
      "name":"c?o"
    }
  }
}
```

#### 3.4.13 fuzzy实现模糊查询

fuzzy：实现模糊查询

    value：查询的关键字
    boost：查询的权值，默认值1.0
    min_similarity：设置匹配的最小相似度，默认值0.5，对于字符串，取值0-1（包括0和1），对于数值，取值可能大于1，对于日期，取值为1d,1m等，1d为1天。
    prefix_length：指明区分词项的共同前缀长度，默认0.
    max_expansions：查询中的词项可以扩展的数目，默认可以无限大。
    
```
# 模糊匹配分词后remark有kevin的数据
GET /demo/user/_search
{
  "query":{
    "fuzzy":{
      "remark":{
        "value":"kevin"
      }
    }
  }
}
# 比如remark的数据为我是keivn，会分词为dos[0]:我[0] 是[1] kevin[2]
``` 

#### 3.4.13 高亮搜索结果

```
# 高亮显示搜索结果
GET /demo/user/_search
{
  "query":{
    "match":{
      "name":"coco"
    }
  },
  "highlight":{
    "fields": {
      "name":{}
    }
  }
}
```
    
### 3.5 Filter查询

filter是不计算相关性的，同时可以cache，所以filter效率高于query。

<b>搜索结果</b>：  
* filter：只查询出搜索条件的数据，不计算相关度分数
* query：查询出搜索条件的数据，并计算相关度分数，按照分数进行倒序排序

<b>性能</b>：  
* filter：不计算相关度分数，也不需要排序，内置的自动缓存最常使用查询结果的数据（缓存bitset不是缓存文档内容）
* query：要计算相关度分数，按照分数进行倒序排序，没有缓存结果的功能

当query与filter同时使用时filter会先执行，并且同时兼顾两者的特性

#### 3.5.1 简单的过滤查询

```
# 过滤只取指定的结果
GET /demo/user/_search
{
  "post_filter":{
    "term":{
      "age":21
    }
  }
}

# 过滤只取指定的多个结果
GET /demo/user/_search
{
  "post_filter":{
    "terms":{
      "age":[21,23]
    }
  }
}
```

新增一个用户id字段设置映射不分词
```
PUT /demo/_mapping/user
{
  "properties":{
    "user_id":{
      "type":"text",
      "index":"false"
    }
  }
}
```

#### 3.5.2 bool过滤查询

bool过滤

    must：必须满足的条件（and）
    should：可以满足也可以不满足的条件（or）
    must_not：不需要满足的条件（not）
格式
```
    {
        "bool":{
            "must":[],
            "should":[],
            "must_not":[]
        }
    }
```
普通bool案例
```
#获取name=keivn or age = 20 且 age <> 24
GET /demo/user/_search
{
  "post_filter":{
    "bool":{
      "should":[
          {"term":{"name":"kevin"}},
          {"term":{"age":20}}
        ],
      "must_not":{
        "term":{"age":24}
      }
    }
  }
}

```
嵌套bool
```
# 嵌套bool,name=kevin or (name=coco and age=21)
GET /demo/user/_search
{
  "post_filter":{
    "bool":{
      "should":[
        {"term":{"name":"kevin"}},
        {
          "bool":{
            "must":[
              {"term":{"name":"coco"}},
              {"term":{"age":21}}
              ]
          }
        }
        ]
    }
  }
}
```

#### 3.5.3 范围过滤

范围过滤：

    gte：>=（大于等于）
    gt：>（大于）
    lte：<=（小于等于）
    lt：<（小于）
    
```
# 范围过滤,取age>20 and age<23的数据
GET /demo/user/_search
{
  "post_filter":{
    "range":{
      "age":{
        "gt":20,
        "lt":23
      }
    }
  }
}
```

#### 3.5.4 过滤非空

```
# 过滤非空,取name is not null 的数据
GET /demo/user/_search
{
  "query":{
    "bool":{
      "filter":{
        "exists":{
          "field":"name"
        }
      }
    }
  }
}
```

检索词频率TF(Term Frequency)对搜索结果排序的影响。  
检索词频率：检索词在该字段出现的频率越高相关性也越高，字段中出现过多次要比一次相关性高。  

constant_score：将query与filter包装，消除TF对搜索结果排序的影响。  
```
GET /demo/user/_search
{
  "query":{
    "constant_score":{
      "filter":{
        "exists":{
          "field":"name"
        }
      }
    }
  }
}
```

#### 3.5.5 过滤器缓存

ElasticSearch提供了一种特殊的缓存，即过滤器缓存(filter cache)，用来存储过滤器的结果，被缓存的过滤器并不需要过多的内存（因为它们只存储了哪些文档能与过滤器相匹配的相关信息），
而且可以供后续所有与之相关的查询重复使用，从而极大的提高了查询的性能。

ElasticSearch默认缓存的过滤器：  
    exists,missing,range,term,terms。

ElasticSearch默认不缓存的过滤器：  
    numeric_range，script，geo_bbox，geo_distance，geo_distance_range
    geo_polygon，geo_shape，and，or，not

开启方式：在filter查询语句后加上"_catch":true

### 3.6 聚合查询

sum：求总数值，等同sum()
```
# 聚合name=kevin的age,等同：name=kevin,sum(age)
GET /demo/user/_search
{
  "size":0,
  "aggs":{
    "age_of_sum":{
      "sum":{
        "field":"age"
      }
    }
  },
  "query":{
    "term":{
      "name":"kevin"
    }
  }
}
# "size":0，不展示原始数据，数值设为多少展示几条，不设置默认全部展示
```
min：求最小值，等同min()
```
#聚合查询，获取最小的年龄，等同min(age)
GET /demo/user/_search
{
  "size":0,
  "aggs":{
    "age_of_min":{
      "min":{
        "field":"age"
      }
    }
  }
}
```
max：求最大值，等同max()
```
#聚合查询，获取最大的年龄，等同max(age)
GET /demo/user/_search
{
  "size":0,
  "aggs":{
    "age_of_max":{
      "max":{
        "field":"age"
      }
    }
  }
}
```
avg：求平均值，等同avg()
```
#聚合查询，获取平均的年龄值，等同avg(age)
GET /demo/user/_search
{
  "size":0,
  "aggs":{
    "age_of_avg":{
      "avg":{
        "field":"age"
      }
    }
  }
}
```
cardinality：求基数，等同count()
```
# 求基数，求age有多少个数据，等同count(age)
GET /demo/user/_search
{
  "size":0,
  "aggs":{
    "age_of_cardinality":{
      "cardinality":{
        "field":"age"
      }
    }
  }
}
```
terms：分组，等同group by 
```
# 分组，根据age进行分组，等同group by age
GET /demo/user/_search
{
  "size":0,
  "aggs":{
    "age_group_by":{
     "terms":{
       "field":"age"
     } 
    }
  }
}
```
嵌套聚合查询案例
```
# 嵌套聚合，获取name=kevin，根据age做分组，再根据已平均的年龄做排序，等同：name=kevin,group by age,order by avg(age)
GET /demo/user/_search
{
  "query":{
    "match":{
      "name":"kevin"
    }
  },
  "size":0,
  "aggs":{
    "age_group_by":{
      "terms":{
        "field":"age",
        "order":{
          "avg_of_age":"desc"
        }
      },
      "aggs":{
        "avg_of_age":{
          "avg":{
            "field":"age"
          }
        }
      }
    }
  }
}
```

### 3.7 复合查询

将多个基本查询组合成单一查询的查询

#### 3.7.1 使用bool查询

参数：

    must：文档必须匹配这些条件才能被包含进来。
    must_not：文档必须不匹配这些条件才能被包含进来。
    should：如果满足这些语句的任意语句，将增加_score，否则无任何影响，主要用于修正每个文档的相关性得分。
    filter：必须匹配，但它以不评分、过滤模式来进行，这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

相关性得分是如何组合的，每一个子查询都独自地计算文档的相关性得分，一旦得分被计算出来，bool查询就将这些得分进行合并并返回一个代表整个布尔操作的得分。

案例：
```
# 标识查找remark包含我是kevin的词，并且age<>21，
# should，sex=男或create_time>=2020-05-03的可以被匹配也可以不被匹配，如果匹配排名更高
GET /demo/user/_search
{
  "query":{
    "bool":{
      "must":{"match":{"remark":"我 是 kevin"}},
      "must_not":{"match":{"age":21}},
      "should":[
        {"match":{"sex":"男"}},
        {"range":{"create_time":{"gte":"2020-05-03"}}}
      ]
    }    
  }
}
```
改写案例：不让创建时间影响得分
```
# 标识查找remark包含我是kevin的词，并且age<>21，
# should，sex=男可以被匹配也可以不被匹配，如果匹配排名更高
# 避免create_time影响得分，将其过滤
GET /demo/user/_search
{
  "query":{
    "bool":{
      "must":{"match":{"remark":"我 是 kevin"}},
      "must_not":{"match":{"age":21}},
      "should":[
        {"match":{"sex":"男"}}
      ],
      "filter":{
        "range":{"create_time":{"gte":"2020-05-03"}}
      }
    }    
  }
}
```
通过将range查询移到filter，转成不评分的查询，将不再影响文档相关排名。
由于上一个案例是不评分的查询，可以使用各种对filter查询有效的优化来提升性能。

bool查询本身也可以被用做不评分的查询，简单地将它放置到filter语句中并在内部构建布尔逻辑。
```
# 标识查找remark包含我是kevin的词，并且age<>21，
# should，sex=男可以被匹配也可以不被匹配，如果匹配排名更高
# 过滤为false的，create_time>=2020-05-03，且age<=25，并name<>'coco'
GET /demo/user/_search
{
  "query":{
    "bool":{
      "must":{"match":{"remark":"我 是 kevin"}},
      "must_not":{"match":{"age":21}},
      "should":[
        {"match":{"sex":"男"}}
      ],
      "filter":{
        "bool":{
          "must":[
            {"range":{"create_time":{"gte":"2020-05-03"}}},
            {"range":{"age":{"lte":25}}}
          ],
          "must_not":[
            {"term":{"name":"coco"}}  
          ]
        }
      }
    }    
  }
}
```

#### 3.7.2 constant_score查询

constant_score：将一个不变的常量评分应用于所有匹配的文档。

使用场景：常用于只需要执行一个filter而没有其它查询（例如评分查询）的情况。

term查询被放置在constant_score中，转成不评分的filter，这种方式可以用来取代只有filter语句的bool查询。
```
GET /demo/user/_search
{
  "constant_score":{
    "filter":{
      "term":{
        "name":"coco"
      }
    }
  }
}
```


## 4.ElasticSearch原理

### 4.1 elasticsearch分布式架构

#### 4.1.1 分布式架构的透明隐藏特性

ElasticSearch是一个分布式系统隐藏了复杂的处理机制

分片机制：

分片副本：
    集群发现机制（cluster discovery）：当启动多个es进程时，非首个启动的进程将作为一个node自动发现并加入集群。
    shard负载均衡：es会均衡分配shard，以保持每个节点均衡的负载均衡。

#### 4.1.2 扩容机制
垂直扩容：购置新的机器，替换已有的机器。
水平扩容：直接增加机器。

#### 4.1.3 rebalance

增加或减少节点时会自动均衡

#### 4.1.4 master与slave节点

master节点：主要职责统计node节点状态信息，集群状态信息，索引创建和删除，索引分配的管理，关闭node接地那，并决定哪些分片分配给相关的节点。

salve节点：同步数据，等待机会成为Master（master宕机/重启）。
 
#### 4.1.5 节点对等

每个节点都能接收请求
每个节点接收到请求后都能把该请求路由到有相关数据的其它节点
接收原始请求的节点负责采集数据并返回给客户端

### 4.2 分片与副本机制

分片（shard）：ES是分布式搜索引擎，索引通常会分成不同的部分，这些分布在不同节点的数据就是分片。
ES自动管理与分片，并会在必要时对分片数据重新平衡分片，所以基本不需担心分片的处理细节，一个分片默认最大文档数量是20亿。

副本（replica）：ES默认为一个索引分配5个主分片，并分别为其创建一个副本分片，每个索引5个主分片与5个副本分片。

分片以及副本分分配是高可用以及快速搜索响应的核心设计，主分片与副本分片都能处理查询请求，区别是只有主分片能处理索引请求。

> 注：  
    1.每分配一个分片，都有额外成本。  
    2.每个分片本质上都是一个Lucene索引，因此会消耗相应文件句柄、内存、CPU资源。  
    3.每个搜索请求会调度索引的每个分片。  
    4.ES使用词频统计来计算相关性，这些统计会分配到各个分片上，若大量分片只维护较少数据，会导致文档相关性较差。

分片建议：假如有3个节点，建议创建分片数量最多不超过(3*3)个。

分片案例：
```
PUT /demo1
{
  "settings":{
    "index":{
      "number_of_shards":3,
      "number_of_replicas":1
    }
  }
}
```

index包含多个shard  
每个shard都是一个最小的工作单元，承载部分数据。  
每个shard都是一个lucene实例，有完整的建立索引和请求能力。  
增减节点时，shard会自动在nodes中负载均衡。  
primary shard和replica shard，每个document肯定只存在某一个primary shard以及其对应的replica shard中，不可能存在于多个primary shard。  
replica shard是primary shard的副本，负责容错，以及承担读请求负载。  
primary shard的数量在创建索引的时候就固定了，replica shard的数量可以随时修改。  
primary shard的默认数量是5，replica默认是1，默认有10个shard，5个primary shard，5个replica shard。  

### 4.3 单节点环境下创建索引分析

分片案例：
```
PUT /demo1
{
  "settings":{
    "index":{
      "number_of_shards":3,
      "number_of_replicas":1
    }
  }
}
```

单节点下会将3个primary shard分配到仅有的一个node上，另外3个replica shard是无法进行分配的（一个shard的replica是不能在同一个节点），
集群可以正常工作，但节点宕机后数据会全部丢失，而且集群不可用无法接受任何请求。

### 4.4 多节点环境下创建索引分析

例如：当前环境为2个节点

将3个primary shard分配到一个node上，另外3个replica shard分配到另外一个节点。
primary shard与replica shard保持同步，并且都可以处理客户端请求。

### 4.5 水平扩容的过程

1.扩容后primary shard和replica shard会自动负载均衡。  
2.扩容后每个节点上的shard会减少，分配给每个shard的CPU，内存，IO资源会更多，性能提高。  
3.扩容的极限，如果有6个shard，扩容的极限就是6个节点，每个节点上一个shard，如果想超出扩容的极限，可以增加replica shard的个数。   

案例：当有6个shard，3个节点时，最多能承受几个节点所在服务器宕机。（容错性）。
如果6个shard：3个primary shard和3个replica shard，则每个服务器宕机都会造成部分数据丢失。
解决方案：

    为了提高容错性可以增加shard个数。
    比如9个shard：3个primary shard和6个replica shard，每个服务器都会有1个primary shard与另外2个不同的replica shard。这时，最高能容忍2台服务器宕机。

> 扩容是为了提高系统的吞吐量，同时也需要考虑容错性，让尽可能多的服务器宕机也能保证数据不丢失。

### 3.6 ElasticSearch的容错机制

以9个shard，3个节点为案例：

如果master node宕机，此时并非所有的primary shard 都是Active status，此时集群状态是shard。

容错处理：

    1.选举一个node作为master。
    2.新选举的master会将挂掉的primary shard某个replica shard提升为primary shard，此时集群状态为yellow，
      因为缺少一个replica shard，并不是所有的replica shard都是active status。
    3.重启宕机的服务器，新master会将所有的副本都复制一份到该节点上（同步宕机后的修改），此时集群的状态green，
      因为所有的primary shard和replica shard都是active status。

### 3.7 文档的核心元数据(_index,_type,_id,_source)

1._index：表示文档存储的索引，同个索引下存放的是相似的文档（文档的field多数为相同）。  
命名：索引名必须小写，不能以下划线开头，不能有逗号。

案例：
```
# 创建名字为test的index
PUT /test
```

2._type：表示文档索引中的类型，一个索引只能有一个type。
命名：类型名可以大小写，不能以下划线开头，不能包括逗号。

案例：
```
# 在test索引创建名字为user的type
PUT /test/user/1
{
  "name":"mrcao"
}
```

3._id：表示文档的唯一标识，与索引、类型组合在一起作为唯一标识一个文档。

生成方式：可以手动指定值，也可以由ES自动生成。 

    1.手动指定值：通常将其它系统的已有数据导入es时。
    2.由ES生成id值：ES生成的id长度为20个字符，使用的是base64编码，URL安全，使用的是GUID算法，分布式下并发生成id值时不会冲突。

案例：
```
# 在test索引的user内创建id为1的数据
PUT /test/user/1
{
  "name":"mrcao"
}

# 在test索引的user内创建id由es自动生成的数据
POST /test/user
{
  "name":"kevin"
}
```

4._source：为添加文档时request的body中的内容

案例：
```
# 查询id=1的name字段
GET /demo/user/1?_source=name
```
        
### 3.8 改变文档内容原理解析

修改文档内容的两种方式

1.全部替换(PUT)：相当于添加一个新文档，将原有的文档覆盖。例如：需要修改age，但其余字段也需要指定，然后将文档标记为deleted。

```
# 更新id为2的name为kevin，version+1
PUT /demo/user/2
{
  "name":"kevin",
  "age":23,
  "sex":"男",
  "remark":"我是kevin",
  "create_time":"2020-05-03"
}
```
> 注：使用全部替换时，需要指定全部字段，如果不指定全部字段，更新后，字段会缺失。

2.修改方式(POST)：使用_update只需要指定修改的字段，ES检索出文档后更新到文档，并将之前的文档标记为deleted。
```
# 更新id为2的name为mrcao，version+1
POST /demo/user/2/_update
{
  "doc":{
    "name":"mrcao"
  }
}
```

区别：
POST方式比PUT方式网络传输次数少，因此提高了性能。
POST方式从查询文档到删除文档，再创建新的文档都是在ES内部实现，降低POST方式并发冲突可能性。

删除文档：标记为deleted，随着数据量增加，ES会在合适的时间删除掉。
```
# 删除id=2的数据，实际是将状态标记为deleted
DELETE /demo/user/2
```

### 3.9 基于groovy脚本执行partial update

es有内置的脚本支持，可以基于groovy脚本实现复杂的操作

es第一次加载一个新脚本时，会将新脚本编译并存储在缓存中，编译可能是一个繁重的过程，如果需要将变量传递给脚本，建议：将它们作为命名参数传递给脚本本身，二不是硬编码在脚本中。



#### 3.9.1 修改数据

```
# 让age在原有的数据在加1
POST /demo/user/3/_update
{
  "script": "ctx._source.age+=1"
}

# 在name字段上过追加字符串cao
POST /demo/user/3/_update
{
  "script": "ctx._source.name+='cao'"
}
```

#### 3.9.2 



































