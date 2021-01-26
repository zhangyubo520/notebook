# elasticsearch

### 1、概念

```
ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎

特点：
1、天然分片、天然集群
2、天然索引：
传统索引：文档 ---> 单词
倒排索引：单词 ---> 文档

```

### 2、数据库比较

|               | redis        | mysql           | elasticsearch                                                | hbase                                                   | hadoop/hive     |
| ------------- | ------------ | --------------- | ------------------------------------------------------------ | ------------------------------------------------------- | --------------- |
| 容量/容量扩展 | 低           | 中              | 较大                                                         | 海量                                                    | 海量            |
| 查询时效性    | 极高         | 中等            | 较高                                                         | 中等                                                    | 低              |
| 查询灵活性    | 较差 k-v模式 | 非常好，支持sql | 较好，关联查询较弱，但是可以全文检索，DSL语言可以处理过滤、匹配、排序、聚合等各种操作 | 较差，主要靠rowkey,  scan的话性能不行，或者建立二级索引 | 非常好，支持sql |
| 写入速度      | 极快         | 中等            | 较快                                                         | 较快                                                    | 慢              |
| 一致性、事务  | 弱           | 强              | 弱                                                           | 弱                                                      | 弱              |

### 3、安装elasticsearch

```
1、tar -zxvf elasticsearch-6.6.0.tar.gz -C /opt/module/

2、修改linux环境(3台都做，所以建议改一台然后分发)
vi /etc/security/limits.conf
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 65536

vim /etc/sysctl.conf 
vm.max_map_count=262144

3、重启linux(reboot)

4、vim /opt/module/elasticsearch-6.6.0/config/elasticsearch.yml
cluster.name: my_es
node.name: node1
network.host: hadoop102
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
discovery.zen.ping.unicast.hosts: ["hadoop102","hadoop103"]
discovery.zen.minimum_master_nodes: 2

5、vim  /opt/module/elasticsearch-6.6.0/config/jvm.options
-Xms256m
-Xmx256m

6、分发并修改
node.name: node1
network.host: hadoop102

7、集群脚本：
#!/bin/bash 
es_home=/opt/module/elasticsearch-6.6.0
kibana_home=/opt/module/kibana-6.6.0-linux-x86_64
case $1  in
 "start") {
  for i in hadoop102 hadoop103 hadoop104
  do
    ssh $i  "source /etc/profile;${es_home}/bin/elasticsearch >/dev/null 2>&1 &"
   done
};;
"stop") {
  for i in hadoop102 hadoop103 hadoop104
  do
      ssh $i "ps -ef|grep $es_home |grep -v grep|awk '{print \$2}'|xargs kill" >/dev/null 2>&1
  done
  
};;
esac


单点启动(三台机器)：/opt/module/elasticsearch-6.6.0/bin/elasticsearch

集群启动：使用脚本elasticsearch.sh start

8、查看：http://hadoop102:9200/_cat/nodes?v
```

### 4、安装kibana

```
1、解压：tar -zxvf kibana-6.6.0-linux-x86_64 -C /opt/module/

2、vim /opt/module/kibana-6.6.0-linux-x86_64/config/kibana.yml
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://hadoop102:9200"]

3、启动（等elasticsearch启动一会儿，然后再启动，不然可能失败）:
nohup /opt/module/kibana-6.6.0-linux-x86_64/bin/kibana >/opt/module/kibana-6.6.0-linux-x86_64/kibana.log 2>&1 &

4、查看：http://hadoop102:5601/

5、关闭：ps -ef|grep /opt/module/kibana-6.6.0-linux-x86_64/bin/kibana |grep -v grep|awk '{print $2}'|xargs kill
```

### 5、简单操作

```
1、查询各个索引状态：GET /_cat/indices?v 

2、服务整体状态查询：GET /_cat/health?v

3、查询各个节点状态：GET /_cat/nodes?v

4、查询某个索引的分片情况：GET /_cat/shards/xxxx

```

### 6、基本概念介绍

| cluster  | 整个elasticsearch 默认就是集群状态，整个集群是一份完整、互备的数据。 |
| -------- | ------------------------------------------------------------ |
| node     | 集群中的一个节点，一般一个进程就是一个node                   |
| shard    | 分片，即使是一个节点中的数据也会通过hash算法，分成多个片存放，默认是5片。（7.0默认改为1片） |
| index    | index相当于table                                             |
| type     | 对表的数据再进行划分，比如“区”的概念。但是实际上对数据查询优化的作用有限，比较鸡肋的设计。（6.x只允许建一个，7.0之后被废弃） |
| document | 类似于rdbms的 row、面向对象里的object                        |
| field    | 相当于字段、属性                                             |

```
参考理解：
 　　index ==》索引 ==》Mysql中的一个库，库里面可以建立很多表，存储不同类型的数据，而表在ES中就是type。

　　 type ==》类型 ==》相当于Mysql中的一张表，存储json类型的数据

　　 document  ==》文档 ==》一个文档相当于Mysql一行的数据

　　 field ==》列 ==》相当于mysql中的列，也就是一个属性
```



### 7、数据操作

```
1、新增一个index：PUT /movie_index

2、删除一个index：DELETE /movie_index

3、新建一个type并添加document，增加field(id)
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

说明：之前没建过index或者type，es 会自动创建

4、非幂等新增
POST /movie_index/movie 
{
  "id":3,
  "name":"incident red sea",
  "doubanScore":5.0,
  "actorList":[  
{"id":4,"name":"zhang chen"}
]
}
说明：重复执行会新增重复数据，_id会随机生成

5、修改文档
PUT /movie_index/movie/3
{
  "id":"3",
  "name":"incident red sea",
  "doubanScore":"5.0",
  "actorList":[  
{"id":"1","name":"zhang chen"}
]
}
说明：和新增没有什么区别，要求：必须包括全部字段

6、post修改文档中某一个字段：
POST movie_index/movie/3/_update
{ 
  "doc": {
    "doubanScore":"7.0"
  } 
}

7、删除文档
DELETE movie_index/movie/3

8、使用id查找：
GET movie_index/movie/1

9、查找所有数据：
GET movie_index/movie/_search

10、match分词查询(对应text类型的)：
GET movie_index/movie/_search
{
  "query":{
    "match": {"name":"red"}
  }
}

11、分词子属性查询：
GET movie_index/movie/_search
{
  "query":{
    "match": {"actorList.name":"zhang"}
  }
}

12、term精准查询：
GET movie_index/movie/_search
{
  "query":{
    "term": {"actorList.name.keyword":"zhang"}
  }
}

13、match_phrase短语查询：
GET movie_index/movie/_search
{
    "query":{
      "match_phrase": {"name":"operation red"}
    }
}

14、fuzzy查询(不支持中文)
GET movie_index/movie/_search
{
    "query":{
      "fuzzy": {"name":"rad"}
    }
}
说明：校正匹配分词，当一个单词都无法准确匹配，es通过一种算法对非常接近的单词也给与一定的评分，能够查询出来，但是消耗更多的性能
例子中要查的是red,但是拼错了rad,系统仍然能够帮忙找到相似的

15、post_filter查询后过滤
GET movie_index/movie/_search
{
    "query":{
      "match": {"name":"red"}
    },
    "post_filter":{
      "term": {
        "actorList.id": 3
      }
    }
}
说明：term类似于直等，match类似于%like%,match_not,should

16、bool过滤后查询
GET movie_index/movie/_search
{ 
    "query":{
        "bool":{
          "filter":[ {"term": {  "actorList.id": "1"  }},
                     {"term": {  "actorList.id": "3"  }}], 
           "must":{"match":{"name":"red"}}
         }
    }
}

17、范围过滤
GET movie_index/movie/_search
{
   "query": {
     "bool": {
       "filter": {
         "range": {
            "doubanScore": {"gte": 8, "lte": 10}
         }
       }
     }
   }
}
说明：gt大于，lt小于，gte大于等于，lte小于等于

18、根据条件删除(很少用，危险)
POST movie_index0/movie/_delete_by_query 
{
  "query":{
    "term": {
      "actorList.name.keyword": "zhang chen"
    }
  }
}

19、根据条件修改
POST movie_index/movie/_update_by_query
{
  "script":"ctx._source['actorList.name']='zhang san feng'",
  "query":{
    "term": {
      "actorList.name.keyword": "zhang chen"
    }
  }
}

20、排序
GET movie_index/movie/_search
{
  "query":{
    "match": {"name":"red sea"}
  },
  "sort": [
    {
      "doubanScore": {
        "order": "desc"
      }
    }
  ]
}

21、分页查询(即可以显示一部分，不全部显示)
GET movie_index/movie/_search
{
  "query": { "match_all": {} },
  "from": 1,
  "size": 1
}

22、指定显示的字段：
GET movie_index/movie/_search
{
  "query": { "match_all": {} },
  "_source": ["name", "doubanScore"]
}

23、高亮某字段：
GET movie_index/movie/_search
{
    "query":{
      "match": {"name":"red sea"}
    },
    "highlight": {
      "fields": {"name":{} }
    }
    
}

24、自定义高亮字段
GET movie_index/movie/_search
{
    "query":{
      "match": {"name":"red sea"}
    },
    "highlight": {
      "fields": {"name":{"pre_tags": "<span color='red'>","post_tags":"</span>"}}
    }
    
}

25、aggs聚合
#定义聚合字段名，类似起别名：groupby_actor
size是取前n个
每个演员共参演了多少部电影
GET movie_index/movie/_search
{ 
  "aggs": {
    "groupby_actor": {
      "terms": {
        "field": "actorList.name.keyword",
        "size": 2
      }
    }
  }
}

GET movie_index/movie/_search
{ 
  "aggs": {
    "groupby_actor_id": {
      "terms": {
        "field": "actorList.name.keyword" ,
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

```

### 8、索引结构

```
term index : 树形索引，用于快速定位分词在term dirctionary的位置

term dirctionary : 列表，列出所有该索引涉及的分词

posting list : 每个分词对应一个列表，该分词对应的所有文档id
```

### 9、使用SQL

```
GET _xpack/sql?format=txt
{
  "query":"select actorList.name.keyword, avg(doubanScore) from movie_index  where match(name,'red') group by actorList.name.keyword limit 1 " 
}

```

### 10、中文分词器ik的使用

```
1、解压到unzip -d /opt/module/elasticsearch-6.6.0/plugins/ik/ elasticsearch-analysis-ik-6.6.0.zip

2、分发

3、使用
```

### 11、字段的数据类型

```

第一种是系统自动推断：
1、true/false → boolean
2、1020  →  long
3、20.1 → float
4、“2018-02-01” → date
5、“hello world” → text +keyword

第二种是自定义字段类型（不能变）
PUT movie_chn
{
  "mappings": {
    "movie":{
      "properties": {
        "id":{
          "type": "long"
        },
        "name":{
          "type": "text"
          , "analyzer": "ik_smart"
        },
        "doubanScore":{
          "type": "double"
        },
        "actorList":{
          "properties": {
            "id":{
              "type":"long"
            },
            "name":{
              "type":"keyword"
            }
          }
        }
      }
    }
  }
}

```

### 12、分割索引

```
概念：类似于分区

优点：
1	查询范围优化： 因为一般情况并不会查询全部时间周期的数据，那么通过切分索引，物理上减少了扫描数据的范围，也是对性能的优化。
2	结构变化的灵活性：因为elasticsearch不允许对数据结构进行修改。但是实际使用中索引的结构和配置难免变化，那么只要对下一个间隔的索引进行修改，原来的索引位置原状。这样就有了一定的灵活性。

使用：把order_info  变成 order_info_20200101,order_info_20200102
```

### 13、自定义词库

```
1、vim /usr/share/elasticsearch/plugins/ik/config/IKAnalyzer.cfg.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict"></entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
        <!--用户可以在这里配置远程扩展字典 -->
         <entry key="remote_ext_dict">http://192.168.238.1/dict</entry>
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!--<entry key="remote_ext_stopwords">words_location</entry>-->
</properties>

2、分词发布服务器
@RestController
public class DictController {

    @GetMapping("dict")
    public  String dict(HttpServletResponse response){
        response.addHeader("Last-Modified",new Date().toString());
        //  查询数据库  得到最新的词条列表

        return "蓝瘦香菇";
    }
}
```

### 14、别名

```
1、新增(filter是条件)：
POST  _aliases
{
    "actions": [
        { "add":    { "index": "movie_index", "alias": "movie_index20200721","filter": {"term": {  "actorList.id": "3"}}}}
    ]
}

2、删除：
POST  _aliases
{
    "actions": [
        { "remove":    { "index": "movie_chn_xxxx", "alias": "movie_chn_2020-query" }}
    ]
}

3、替换：
POST /_aliases
{
    "actions": [
        { "remove": { "index": "movie_chn_xxxx", "alias": "movie_chn_2020-query" }},
        { "add":    { "index": "movie_chn_yyyy", "alias": "movie_chn_2020-query","filter":{"term": {  "actorList.id": "1"  }}}}
    ]
}

4、查看别名：GET  _cat/aliases?v

说明：别名的用途
（1）灵活的扩容：推荐每个人为他们的Elasticsearch所以使用别名，因为在未来重建索引的时候，别名会赋予你更多的灵活性。假设一开始创建索引只有一个主分片，之后你又决定为索引扩容。如果为原索引使用的是别名，现在你可以修改别名让其指向额外创建的新索引，而无须修改被搜索的索引之名称（假设一开始你就为搜索使用了别名）。

（2）动态的滚动查询：在实际应用中，我们也不应该向单个索引持续写入数据，知道它的分片巨大无比。巨大的索引会在数据老化后难以删除，以——id为单位删除文档不会立即释放空间，删除doc只在lucene分段合并时才会真正从磁盘中删除。即使手工触发分段合并，仍会引起较高的I/O压力，并且可能因为分段巨大导致合并过程中磁盘空间不足（分段大小大于此片可用空间的一半）

因此，另外一个有用的特性是：在不同的索引创建窗口。比如，如果为数据创建了每日索引，你可能期望一个滑动窗口覆盖过去一周的数据，别名就称为last-7-days.然后，每天创建新的每日索引时，将其加入别名，同时删除第8天前的旧索引。

这样，对于业务方来说，读取时使用的别名不变，当需要删除数据的时候，可以直接删除整个索引

（3）进行索引分组

（4）使用别名过滤器来屏蔽文档，他们可以对正在执行的查询自动地实施过滤

（5）结合别名和路由，在查询或索引得时候自动地使用路由值。

```

### 15、定义模板

```
1、创建：

PUT _template/template_movie2020
{
  "index_patterns": ["movie_test*"],                  
  "settings": {                                               
    "number_of_shards": 1
  },
  "aliases" : { 
    "{index}-query": {},
    "movie_test-query":{}
  },
  "mappings": {                                          
"_doc": {
      "properties": {
        "id": {
          "type": "keyword"
        },
        "movie_name": {
          "type": "text",
          "analyzer": "ik_smart"
        }
      }
    }
  }
}


2、使用
PUT movie_test_20200721/_doc/333
{
  "id":"333",
  "name":"zhang3"
}

PUT movie_test_20200722/_doc/333
{
  "id":"444",
  "name":"zhang4"
}


3、查看所有模板：GET  _cat/templates

4、查看模板详情：GET  _template/template_movie2020
```

### 16、分片shard及优化

```
6.0默认是5个分片，7.0改为1个

优化：
1、手动合并segment：
POST  movie_index/_forcemerge?max_num_segments=1

```

### 17、java程序

```
 package com.atguigu.gmall2020.realtime.util

import java.util

import io.searchbox.client.config.HttpClientConfig
import io.searchbox.client.{JestClient, JestClientFactory}
import io.searchbox.core.{Index, Search, SearchResult}
import org.apache.lucene.queryparser.xml.builders.BooleanQueryBuilder
import org.elasticsearch.index.query.{BoolQueryBuilder, MatchQueryBuilder}
import org.elasticsearch.search.builder.SearchSourceBuilder

import scala.collection.mutable.ListBuffer


object MyEsUtil {

  private    var  factory:  JestClientFactory=null

//  使用工厂类创建对象
  def getClient:JestClient ={
    if(factory!=null){
      factory.getObject
    }else{
      build()
      factory.getObject
    }
  }

//  创建对象细节
  def  build(): Unit ={
    factory=new JestClientFactory
    factory.setHttpClientConfig(new HttpClientConfig.Builder("http://hadoop102:9200" )
      .multiThreaded(true)
      .maxTotalConnection(20)
      .connTimeout(10000).readTimeout(1000).build())

  }

  def main(args: Array[String]): Unit = {
    val client: JestClient = getClient//从连接池中获取对象

    //    写操作
    //    val index = new Index.Builder(Movie("334","复仇")).index("movie_test_20200721").`type`("_doc").id("334").build()
    //    client.execute(index)
    //    client.close()

    queryFromEs()


  }
//  读操作

  def queryFromEs() = {
    val client:JestClient= getClient//获取连接池中的对象

    val builder = new SearchSourceBuilder

    val builder1 = new BoolQueryBuilder()//用来定义查询内容
    builder1.should(new MatchQueryBuilder("movie_name","复仇"))//定义查询内容
    builder.query(builder1)
    val string: String = builder.toString

    println(string)

    val search = new Search.Builder(string).addIndex("movie_test_20200721").addType("_doc").build()

    val redult: SearchResult = client.execute(search)

    val list: util.List[SearchResult#Hit[util.Map[String, Any], Void]] = redult.getHits(classOf[util.Map[String,Any]])

    val buffer = new ListBuffer[util.Map[String,Any]]

    import  scala.collection.JavaConversions._
    for ( hit <- list ) {
      val source: util.Map[String, Any] = hit.source
      buffer.add(source)
    }

    println(buffer.mkString("\n"))

    client.close()
  }
  case class Movie(id: String,movie_name: String)
}

说明：写操作的输出结果：{id=334, movie_name=复仇, es_metadata_id=334}
```

