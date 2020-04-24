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
// 没有指定_id时，ES会自动生成一个随机id
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
// 创建一个文档
PUT /demo/user/1
{
  "name":"kevin",
  "sex":"男",
  "hobby":["java"]
}
// 此时_version=1
```
内部版本控制（案例）：
```
// 更新版本并指定为version=1
PUT /demo/user/1?version=1
{
  "name":"kevin",
  "age":24,
  "sex":"男"
}
// 更新成功，此时_version=2

// 再次更新版本并还是指定version=1
PUT /demo/user/1?version=1
{
  "name":"kevin",
  "age":24,
  "sex":"男"
}
// 更新失败，因为版本不一致，抛出异常：reason": "[user][1]: version conflict, current version [2] is different than the one provided [1]"

```
外部版本控制（案例）：
```
// 更新时，判断_version是否比指定的数值小，如果是则保存外部版本号到version中
PUT /demo/user/1?version=4&version_type=external
{
  "name":"kevin",
  "age":24,
  "sex":"男"
}
// 更新前，version=2，指定的数值为4，判定通过，更新数据成功，此时version=4
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
> 设置搜索时的分词器，默认跟ananlyzer是一致的，比如index时用standard+ngram，搜索时用standard用来完成自动提示功能

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




## 3、 Elasticsearch DSL常用语法







