# ElasticSearch

## Elasticsearch简介及安装部署

 - ElasticSearch是一个实时的分布式的搜索和分析引擎。它是对lucene进行封装。能够达到实时搜索、稳定可靠、快速等特点。基于REST接口（API）

    - 普通接口请求是...get?a=1
    - rest接口请求是...get/a/1

- ElasticSearch的用户

  - GitHUb、WiKIpedia、eBay等。

- 全文检索工具

  - Lucene
    - Java的一个开源搜索引擎，在java世界中是标准的全文检索程序，提供了完整的查询引擎和搜索引擎。http://lucene.apache.org/
  - solr（solr.4.x solrcloud）
    - solr是一个用java开发的独立的企业级的搜索应用服务器，提供了类时于Web-service的API接口，它是基于Lucene的全文检索服务器。http://lucene.apache.org/solr/ 
  - ElasticSearch
    - ElasticSearch是一个采用java语言开发的基于Lucene构造的开源的、分布式的搜索引擎，能够实现实时搜索。http://lucene.apache.org/solr/ 

- MySQL和ES区别

  | MySQL            | elasticsearch  |
  | ---------------- | -------------- |
  | database(数据库) | index(索引库)  |
  | table（表）      | type(类型)     |
  | row(行)          | document(文档) |
  | column(列)       | field(字段)    |

  

## Elasticsearch的基本操作

- curl 命令

  - -x 指定http请求的方法 （GET POST PUT DELETE）
  - -d 指定要传递的参数
  - -H header信息

- curl 命令简单使用

  - curl 创建索引库

    - `curl -XPUT 'http://hadoop128:9200/test/'`
    - PUT/POST都可以

  - curl 创建索引

    - ```shell
      curl -H "Content-Type:application/json" 
      -XPOST http://hadoop128:9200/test/emp/1
      -d '{
          "name":"tom",
          "age":25
      }'
      ```

    - 注意

      1. 索引库名称必须小写，不能以[_ - +]开头，也不能包含逗号

      2. 如果没有明确指定索引数据id，es会自动生成一个随机的ID(仅能使用POST参数) 

         ```shell
         curl -H "Content-Type:application/json" -XPOST http://hadoop128:9200/test/emp/ -d '{ "name":"tom", "age":25 }'
         ```

      3. 如果想要确定我们创建的都是全新的数据

         1. 使用随机ID（POST）方式

         2. 在url后面添加参数

            ​	`curl -H "Content-Type:application/json" -XPUT http://hadoop128:9200/test/emp/2/_create -d '{ "name":"tom", "age":25 }'`

      

  - curl 查询索引——GET

    - 根据id进行查询
      - `curl -XGET http://hadoop128:9200/test/emp/1?pretty`
      - 引号最好加上有时候可以不加，在任意的查询字符串添加pretty（pretty=true的缩写）参数，es可以得到易于识别的json结果
    - 检索文档中的一部分，显示指定字段
      - `curl -XGET 'http://hadoop128:9200/test/emp/1?_source=name&pretty'`
    - 查询指定索引库的所有数据
      - ` curl -XGET 'http://hadoop128:9200/test/emp/_search?pretty'` 

  - curl 更新

    - es可以使用put和post对文档进行更新（全部更新），如果指定文档已经存在，则执行更新操作

      - `curl -XPOST -H "Content-Type:application/json" -XPOST http://hadoop128:9200/test/emp/1/ -d '{ "stu":"ac", "score":20 }'`
      - 执行更新操作的时候

      1. es首先将旧的文档标记为删除状态
      2. 添加新文档，此时旧的文档不会立即消失，但是你也无法访问
      3. ES会在你继续添加更多数据的时候，后台字都清理已经标记为删除状态的文档

    - 局部更新，可以添加新字段或更新已经存在的字段（必须使用POST）

      - `curl -XPOST -H "Content-Type:application/json" -XPOST http://hadoop128:9200/test/emp/1/_update -d '{"doc":{ "stu":"alice"}}'`
      - doc 为文档

  - ucrl 删除

    - `curl -XDELETE http://hadoop128:9200/test/emp/1`
    - 如果文档存在，es会返回200 ok状态码，found属性值为true，_version属性值+1
    - 如果文档不存在，es会返回404 not found，found属性为false,_version属性值+1，这是内部管理的一部分，保证了我们在多个节点间的不同操作顺序都被正确标记了
    - 删除一个文档也不会立即生效，它只是被标记成已删除状态。ES会在之后添加大量数据的时候，在后台清理标记为删除状态的内容。

- 批量操作-bulk

  - bulk API可以同时执行多个请求

  - 格式

    - ```javascript
      action:index/create/update/delete
      metadata:_index,_type,_id
      request body:_source（删除操作不需要）
      {action:{metadata}}
      {request body }
      ......
      ```

    - create和index的区别

      - 如果数据存在，使用create操作失败，会提示文档已经存在，使用index则可以成功执行。

  - 使用文件的方式

    - `vi requests`

    - ```shell
      { "index" : { "_index" : "test1", "_type" : "type1", "_id" : "1" } }
      { "field1" : "value1" }
      
      { "index" : { "_index" : "test1", "_type" : "type1", "_id" : "2" } }
      { "field1" : "value1" }
      
      { "delete" : { "_index" : "test1", "_type" : "type1", "_id" : "2" } }
      
      { "create" : { "_index" : "test1", "_type" : "type1", "_id" : "3" } }
      { "field1" : "value3" }
      
      { "update" : {"_index" : "test1", "_type" : "type1","_id" : "1" } }
      { "doc" : {"field2" : "value2"} }
      ```

    - `curl -XPOST -H "Content-Type:application/json" -XPOST hadoop128:9200/test1/emp/_bulk --data-binary @requests`

    - bulk 请求可以在URL中声明/_index或者 _index/ _type

    - bulk最大处理多少数据

      - bulk会把将要处理的数据载入到内存中，所有数据量是有限制的
      - 最佳的数据量不是一个确定的数值，它取决于你的硬件、你的文档大小以及复杂性，你的索引以及ES的负载
      - 一般建议是1000-5000个文档，如果文档很大，可以适当减少队列，大小建议是5-15MB，默认不能超过100M，可以在es的配置文件中修改这个值http.max.content_length:100mb【不建议修改，太大的话bulk也会慢】
      - https://www.elastic.co/guide/en/elasticsearch/reference/6.4/modules-http.html

      



## Elasticsearch插件介绍

 - ElasticSearch Head Plugin
    - 很方便对es进行各种操作的客户端
- 按照步骤参考某度和某歌

## Elasticsearch配置参数详解

 - elasticsearch.yml

    - es已经为大多数参数设置合理的默认值

    - 这个文件是yml格式文件

      1. 属性顶格写，不能有空格

      2. 缩放一定不能使用tab制表符，只能使用空格

      3. 属性和值之间的：后面需要空格

         network.host: 192.168.1.4

- 部分参数介绍

| 参数: 值                                             | 介绍                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| cluster.name: my-application                         | 集群名称，默认是elasticsearch                                |
| node.name: node-1                                    | 节点名称，如不指定 默认字段生成                              |
| path.data: /path/to/data                             | es数据存储目录，默认存储在es_home/data目录下                 |
| path.logs: /path/to/logs                             | es日志存储目录， 默认存储在es_home/logs目录下                |
| bootstrap.memory_lock: true                          | 锁定物理内存地址，防止elasticsearch内存被交换出去，也就是避免es使用swap分区 |
| network.host: 192.168.0.1                            | 本机ip，es1.x绑定的是0.0.0.0不需要自己配置，es2.x起默认绑定127.0.0.1，需要修改 |
| http.port: 9200                                      | es 服务端口                                                  |
| discovery.zen.ping.unicast.hosts: ["host1", "host2"] | 启动新节点，通过这个ip列表发现组件集群                       |
| discovery.zen.minimum_master_nodes:                  | 防止集群脑裂现象 （九群总节点数量/2）+1                      |
| gateway.recover_after_nodes: 3                       | 一个集群n个基点启动后，才允许进行数据恢复处理，默认是1       |
| action.destructive_requires_name: true               | 设置是否可以通过正则或_all删除或者关闭索引库，默认true表示必须要显式指定索引库名称，生产环境建议设置为true，删除索引库时必须显式指定，否则可能会误删索引库中的索引库 |

  

## Elasticsearch核心概念

 - cluster
    - 代表一个集群，集群中有多个节点，其中一个为主节点，这个主节点是可以通过选举产生的，主从节点是对于集群内部来说的，从外部看es集群，在逻辑上是个整体，与任何一个节点和通信和与整个集群通信是等价的。
    - 主节点的职责是负责管理集群状态，包括管理分片的状态和副本的状态，以及节点的发现和删除。
    - 注意：主节点不负责对数据的增删改查请求进行处理，只负责维护集群相关状态信息。
- 集群配置（修改conf/elasticsearch,yml）
    - discovery.zen.ping.unicast.hosts: ["host1", "host2:9300"]
- 集群状态查看
    - http://hadoop128:9200/_cluster/health?pretty
- shards
    - 代表索引分片，es可以把一个完整的索引分成多个分片，这样的好处是可以把一个大的索引水平拆分成多个，分布到不同的节点上。构成分布式搜索，提高性能和吞吐量。
    - 分片的数量只能在创建索引库时指定，索引库创建后不能更改。
        - `curl -H "Content-Type:application/json" -XPUT 'hadoop128:9200/test2/' -d '{"settings":{"number_of_shards":3}}'`(此处只能用PUT)
- replicas
    - 代表分片的副本，es可以给索引分片设置副本，副本的作用一是提高系统的容错性，当某个节点某个分片损坏或丢失时可以从副本中恢复。二是提高es的查询效率，es会自动对搜索请求进行负责均衡(副本数量可以随时修改)
    - 可以在创建索引库的时候指定
        - `curl -XPUT 'hadoop128:9200/test/' -d'{"settings":{"number_of_replicas":2}}'`
    - 默认一个分片有一个副本
        - index.number_of_replicas:1
    - 注意：主分片和副本不会存放在一个节点中。
    - recovery
        - 代表数据恢复或者叫数据重新分布，es在有节点加入或退出时会根据机器的负载对索引分片进行重新分配，挂掉的节点重新启动时也会进行数据恢复。

## Elasticsearch javaapi操作

 - 添加maven依赖

    - ```java
      <dependency>
          <groupId>org.elasticsearch.client</groupId>
          <artifactId>transport</artifactId>
          <version>6.4.3</version>
      </dependency>
      ```

- 连接es集群

  - 1 通过TransportClient这个类，指定es集群中其中一台或者多台机器的ip地址和端口

    ```java
    TransportClient client = new PreBuiltTransportClient(Settings.EMPTY).addTransportAddress(new TransportAddress(InetAddress.getByName("host1"), 9300)).addTransportAddress(new TransportAddress(InetAddress.getByName("host2"), 9300));
    ```

  - 2 添加其他配置

    ```java
    Settings settings = Settings.builder()
        .put("cluster.name", "myClusterName") //集群名称
        .put("client.transport.sniff", true)  //打开嗅探，es会自动把集群其他机器ip地址加到客户端
        .build();
    TransportClient client = new PreBuiltTransportClient(settings).addTransportAddress(new TransportAddress(InetAddress.getByName("host1"), 9300)).addTransportAddress(new TransportAddress(InetAddress.getByName("host2"), 9300));
    
    ```

  - 3 es-java客户端操作

    - 索引Index（json，map，bean，es helper）

      - ` IndexResponse response = client.prepareIndex("test", "emp", "1").setSource().get()` //建立索引

    - 查询 Get

      - ` GetResponse response = client.prepareGet("test", "emp", "1").get();`

    - 更新update

      - ```java
        UpdateResponse response4 = client.prepareUpdate(index, type, "101")
                .setDoc("{\"age\":180}", XContentType.JSON)
                .get();
        ```

    - 删除delete

      - `DeleteResponse response5 = client.prepareDelete(index, type, "101").get();`

    - 批量操作

      - ```java
         BulkRequestBuilder bulkRequestBuilder = client.prepareBulk();
          
         IndexRequest indexRequest = new IndexRequest(index, type);
         indexRequest.source("{\"name\":\"zs8888\"}",XContentType.JSON);
          
         //  执行批量操作
                BulkResponse bulkItemResponses = bulkRequestBuilder.get();
                //  因为这一批数据在执行的时候，有可能会有一部分数据执行失败，所以在执行之后可以获取一些失败信息
                if(bulkItemResponses.hasFailures()){
                    //  如果有失败进行，则为true
                    //  获取失败信息
                    BulkItemResponse[] items = bulkItemResponses.getItems();
                    for (BulkItemResponse item: items) {
                        //  打印具体的失败消息
                        System.out.println(item.getFailureMessage());
                    }
                }else{
                    System.out.println("全部执行成功!!!");
                }
          
        ```

    - 查询类型searchType

      | 元素                 | 含义                                                         |
      | -------------------- | ------------------------------------------------------------ |
      | query_then_fetch     | 查询是针对所有的块执行的，但是返回的是足够的信息，而不是文档内容（Documnet）。结果会被排序和分级，基于此，只有相关的块的文档对象会被返回。由于被取到的仅仅是这些，故返回的hit的大小正好等于指定的size。这对于有许多块的index来说是很便利的（返回结果不会有重复的。因为块被分组了）。 |
      | query_and_fetch      | 最原始(也可能是最快的)实现就是简单的在所有相关的shard上执行检索并返回结果。每个shard返回一定尺寸的结果。由于shard已经返回一定尺寸的hit，这种类型实际上是返回多个shard的一定尺寸的结果给调用者。 |
      | dfs_query_then_fetch | 与query_then_fench相同，预期一个初始散射相伴用来更为准确的score计算分配了的term频率 |
      | dfs_query_and_fetch  | 与query_and_fetch相同，预期一个初始散射相伴用来更为准确的score计算分配了的term频率 |

      

    - es搜索有四种

      - query and fetch（速度最快）（返回N倍数据量）
      - query then fetch (默认的搜索方式)
      - DFS query and fetch
      - DFS query then fetch(更精确的控制搜索打分和排名)

    - 总结

      - 从性能考虑来说，Query_AND_FETCH 是最快的，DFS_QUERY_THEN_FETCH是最慢的。
      - 从搜索的准确度来说，DFS要比非DFS的准确度要高。

    - 注

      - DFS ，Distributed frequency Scatter，分布式词频率和分布式文档散发的缩写。
      - 初始化过程：初始化散发就是在进行真正的查询之前，先把各个分片的词频率和文档频率收集一下，然后进行词搜索的时候，各分片依据全局的词频率和文档频率进行搜索和排名。如果使用DFS_QUERY_THEN_FETCH这种查询方式，效率是最低的，因为一个搜索可能要请求3次分片。但是用DFS方法搜索精度应该是最高的。

## Elasticsearch 分词详解(整合ik分词器,自定义词库,热更新词库)

 - es搜索系统的索引结构是倒排索引
 - 结构（
 - 单词ID：记录每个单词的单词编号
 - 单词： 对应单词
 - 文档频率： 代表文档集合中有多少文档包含某单词
 - 倒排列表： 包含单词ID以及其他必要信息
    - Docld： 单词出现的文档id
    - TF：单词在某个文档中出现的次数
    - POS: 单词在文档中出现的位置
 - ）



### 分词器

 - 分词器是把一段文本中的词按一定规则进行切分。对应的是Analyzer类，这是一个抽象类，具体规则是由子类实现的，所以对于不同的语言，要用不同的分词器，不同语言的分词器是不同的。
 - 在创建索引时会用到分词器，在搜索时也会用到分词器，两个地方要是使用同一个分词器，否则可能会搜索不出结果。
 - 工作流程
    - 1切分关键词
    - 2去除停用词(a,an,the of，的了 ) 
       - 英文停用词 http://www.ranks.nl/stopwords
       - 中文停用词 http://www.ranks.nl/stopwords/chinese-stopwords
    - 3 对于英文，全部转小写（搜索时不区分大小写）
- 中文分词有 单词分词，二分分词，词库分词
  - 几个重要分词器
    - StandardAnalyzer 单字分词
    - ChinessAnalyzer 单字分词
    - CJKAnzlyzer 二分分词
    - IKAnalyzer 词库分词

### ES中文分词插件 es-ik

 - es官方默认分词器 对中文分词效果不理想
 - 集成ik分词工具
    - 下载es的ik插件 https://github.com/medcl/elasticsearch-analysis-ik/releases
    - 解压到es_home/plugins/ik目录下 unzip
    - 重启es
    - 测试分词效果 curl -H "Content-Type: application/json" 'http://localhost:9200/test/_analyze?pretty=true' -d '{"text":"我们是中国人","tokenizer":"ik_max_word"}'
       - 分词模式：
          - ik_max_word:会将文本做最细颗粒度的拆分，会穷尽各种可能的组合
          - ik_smart: 会做最粗粒度的拆分。
   - es-ik 自定义词库
     - 自定义词库
       - 在ik目录下面创建一个custom目录，创建一个my.dic文件,放入不想被切分的词
       - 编辑config/IKAnalyzer.cfg.xml文件
       - `<entry key="ext_stopwords">custom/my.dic</entry>`
       - 重启es
     - 热更新ik词库
       - 编辑onfig/IKAnalyzer.cfg.xml文件
       - `<entry key="remote_ext_dict">网络位置</entry>`
       - 可以从网络中添加一个文件作为词典如http://192.168.80.100:8080/hot.dic
     - 词典文件必须是utf8 无bom的。

## Elasticsearch 查询详解(置顶查询,高亮,聚合等...)

```java
 SearchResponse searchResponse = client.prepareSearch(index)

                .setQuery(QueryBuilders.matchAllQuery()) // 查询所有数据
                //.setQuery(QueryBuilders.matchQuery("name","tom")) // 根据某一列进行模糊查询，不支持通配符
               // .setQuery(QueryBuilders.multiMatchQuery("tom","name","dada"))// 根据多列进行模糊查询
               // .setQuery(QueryBuilders.queryStringQuery("name:to*")) //lucene语法，支持*？通配符
             /*   .setQuery(QueryBuilders.boolQuery()
                .should(QueryBuilders.matchQuery("name","tom")).boost(5.0f)
                        .should(QueryBuilders.matchQuery("age",45)).boost(1.0f)
                )*/
                // .setQuery(QueryBuilders.termQuery("name","tom"))//默认会分词
                // 默认索引库分词，termQuery查不到数据
                //.setQuery(QueryBuilders.queryStringQuery("name:\"tom train\""))
                .setSize(5) //一次获取多少条数据，默认10
                .setFrom(0) // 表示从哪一条数据开始，默认角标为0
                .addSort("age",SortOrder.DESC) // 指定字段进行排序
                .setPostFilter(QueryBuilders.rangeQuery("age").from(15).to(20))//对数据进行过滤
                .setExplain(true) // 根据数据匹配度返回数据
                .get();


        SearchHits hits = searchResponse.getHits();
        // 获取查询数据的总条数
        long totalHits = hits.getTotalHits();
        System.out.println("数据总条数"+totalHits);

        for(SearchHit item:hits){
            System.out.println(item.getSourceAsString());
        }
```

 - 统计 使用aggregations 

    - 根据字段进行分组统计
    - 根据字段分组，统计其他字段的值
    - size设置为0，会获取所有数据，否则，默认返回前10个分组的数据。

   ```java
   /**
        * 聚合，aggregation 分组求sum
        * @param client
        */
       private static void testAggSum(TransportClient client) {
   
           SearchResponse searchResponse = client.prepareSearch("test")//   指定索引库信息。可以指定多个，中间用逗号隔开
                   .setQuery(QueryBuilders.matchAllQuery())//  指定查询规则
                   .addAggregation(AggregationBuilders.terms("name_term").field("name.keyword")//默认name是Text类型，这个类型不支持排序，所以需要使用这个字段的keyword类型
                           .subAggregation(AggregationBuilders.sum("score_sum").field("score"))
                   )// 指定分组字段和聚合字段信息
                   .get();
           //  获取分组信息
           Terms name_term = searchResponse.getAggregations().get("name_term");
           List<? extends Terms.Bucket> buckets = name_term.getBuckets();
           for (Terms.Bucket bk: buckets) {
               //  获取分值的和
               Sum score_sum = bk.getAggregations().get("score_sum");
               System.out.println(bk.getKey()+"----"+score_sum.getValue());
           }
   
       }
   
   /**
        *  统计相同年龄学员的个数
        * @param client
        */
       private static void testAggcount(TransportClient client) {
   
           SearchResponse searchResponse = client.prepareSearch(index)
                   .setQuery(QueryBuilders.matchAllQuery())
                   .addAggregation(AggregationBuilders.terms("age_term").field("age"))
                   .get();
           Terms age_term = searchResponse.getAggregations().get("age_term");
           List<? extends Terms.Bucket> buckets = age_term.getBuckets();
           for (Terms.Bucket bk:
                buckets) {
               System.out.println(bk.getKey()+"__"+bk.getDocCount());
           }
   
       }
   ```

### ES 删除索引库

   - curl -XDELETE 'http://localhost:9200/test'
   - transportClient.admin().indices().prepareDelete("test").get();
   - 注意：这样会把索引库及索引库中的所有数据都删掉，慎用。

### ES 分页

 - 与SQL使用LIMIT来控制单”页“数量类似，Elasticsearch使用的是form以及size两个参数：
    - from：从哪条结果开始，默认值为0
    - size：每次返回多少个结果，默认为10
- 假设每页显示5条结果，1页至3页的请求就是：
  - GET /_search?size=5
  - GET /_search?size=5&from=5
  - GET /_search?size=5&from=10
- 注意： 不要一次请求过多或者页码过大的结果，这么做会对服务器造成很大压力。因为它们会在返回前排序。一个请求会经过多个分片。每个分片都会生成自己的排序结果。然后再集中进行整理，以确保最终结果的正确性。

## Elasticsearch的settings和mappings详解

- settings修改索引库默认配置
  - 如分片数量、副本数量。
  - 查看：` curl -XGET http://localhost:9200/test/_settings?pretty`
  - 操作不存在索引：` curl -H "Content-Type: application/json" -XPUT 'localhost:9200/test1/' -d'{"settings":{"number_of_shards":3,"number_of_replicas":0}}'`
  - 操作已经存在索引：` curl -H "Content-Type: application/json" -XPUT 'localhost:9200/test1/_settings' -d'{"index":{"number_of_replicas":1}}'`
- Mapping,就是对索引库中索引的字段名称以及数据类型进行定义，类似于mysql的表结构信息。不过es的mapping比数据库灵活很多，它可以动态识别字段。一般不需要指定mapping都可以。因为es会自动根据数据格式识别它的类型，如果你需要对某些字段添加特殊属性（定义使用其它分词器、是否分词、是否存储），就必须手动添加mapping。
  - 查询索引库的mapping信息
    - `curl -XGET http://localhost:9200/test/emp/_mapping?pretty`
  - 指定分词器
    - `curl -H "Content-Type: application/json" -XPUT 'localhost:9200/test2' -d'{"mappings":{"emp":{"properties":{"name":{"type":"text","analyzer": "ik_max_word"}}}}}'`
  - 

## Elasticsearch的分片查询方式

- 默认randomize across shards
  - 随机读取，表示随机从分片中读取数据。
- _local:指查询操作会优先在本地节点有的分片进行查询，没有的话再去其他节点查询。
- _only_local:指查询只会在本地节点有的分片中查询。
- _primary: 指查询只在主分片中查询。
- _replica:指查询只在副本中查询
- _primary_first: 指查询会先在主分片中查询，如果主分片找不到(挂了),再去副本中查询。
- _replica_first: 指查询会先在副本中查询，如果副本找不到了(挂了),就会在主分片中查询。
- _only_node: 指在指定id的节点里面进行查询，如果该节点只要有查询索引的部分分片，就会在这部分分片中查找，所以查询结果可能不完整。
- _only_nodes: 指定多个节点id，查询多个节点的数据。
- _prefer_node:nodeid 优先在指定的节点上查询
- _shards:0,1,2,3：查询指定分片的数据。

## Elasticsearch的脑裂问题分析

 - 同一个集群中的不同节点对集群的状态有了不一样的理解。

 - discovery.zen.minimum_master_nodes

    - 用于控制选举行为发生的最小集群节点数量。推荐设为大于1的数值，因为只有在2个以上节点的集群中，主节点才是有意义的。

   

## Elasticsearch扩展之索引模板和索引别名

- 在实际工作中针对一批大量数据存储的时候需要使用多个索引库，如果手工指定每个索引库的配置信息(settings和mappings)的话就很麻烦了。

  - 创建模板 https://www.elastic.co/guide/en/elasticsearch/reference/6.4/indices-templates.html

    - ```shell
      curl -H "Content-Type: application/json" -XPUT localhost:9200/_template/template_1 -d '
      {
          "template" : "*",
          "order" : 0,
          "settings" : {
              "number_of_shards" : 1
          },
          "mappings" : {
              "type1" : {
                  "_source" : { "enabled" : false }
              }
          }
      }
      '
      curl -XPUT localhost:9200/_template/template_2 -d '
      {
          "template" : "te*",
          "order" : 1,
          "settings" : {
              "number_of_shards" : 1
          },
          "mappings" : {
              "type1" : {
                  "_source" : { "enabled" : true }
              }
          }
      }
      '
      
      ```

    - 注意 order值大的模板内容会覆盖order值小的。

  - 查看模板： ` curl -XGET localhost:9200/_template/temp*?pretty`

  - 删除模板： `curl -XDELETE localhost:9200/_template/temp_1`



### ES扩展之index alias

 - 索引别名的应用场景：
 - 公司使用es收集应用的运行日志，每个星期创建一个索引库，这样时间长了就会创建很多的索引库，操作和管理的时候很不方便。
 - 由于新增索引数据只会操作当前这个星期的索引库，所以就创建了两个别名
   - curr_week：这个别名指向这个星期的索引库，新增数据操作这个索引
   - last_3_month：这个别名指向最近三个月的所有索引库，因为我们的需求是查询最近三个月的日志信息。后期只需要修改这两个别名和索引库之间的指向关系即可。应用层代码不需要任何改动。
     还要把三个月以前的索引库close掉，留存最近一年的日志数据，一年以前的数据删除掉。
 - ES默认对查询的索引的分片总数量有限制，默认是1000个，使用通配符查询多个索引库的时候会这个问题，通过别名可以解决这个问题

## Elasticsearch参数优化

### 优化1

 1. 解决启动的警告信息

     	1. max file descriptors [4096] for elasticsearch process likely too low, consider increasing to at least [65536]
          	2. vi /etc/security/limits.conf 添加下面两行(使用root用户)
         - \* soft nofile 65536
         - \* hard nofile 131071

 2. 修改配置文件调整ES的JVM内存大小

     1. 修改bin/elasticsearch.in.sh中ES_MIN_MEM和ES_MAX_MEM的大小，建议设置一样大，避免频繁分配内存，根据服务器内存大小，一般分配60%左右（默认256M）

     2. 内存最大不要超过32G

        一旦你越过这个神奇的32 GB边界，指针会切换回普通对象指针.。每个指针的大小增加，使用更多的CPU内存带宽。事实上，你使用40~50G的内存和使用32G的内存效果是一样的。

 3. 设置memory_lock来锁定进程的物理内存地址

     	1. 避免交换(swapped)，来提高性能
          	2. 修改文件conf/elasticsearch.yml
               	3. bootstrap.memory_lock:true
                    	4. 需要根据es启动日志修改/etc/security/limits.conf文件(重启系统)



### 优化2

 1. 分片多的话，可以提升索引的能力，5-20个比较合适

     	1. 如果分片数过少或过多，都会导致检索比较慢。
          	2. 分片数过多会导致检索时打开比较多的文件，另外也会导致多台服务器之间大量通讯。
               	3. 而分片数量过少导致单个分片索引过大，所以检索速度也会慢。
                    	4. 建议单个分片存储20G左右的索引数据【最高也不要超过50G,否则性能会很差】，所以分片数量=数据总量/20G

	2. 副本过多的话，可以提升搜索的能力，但是如果设置很多副本的话，也会对服务器造成额外的压力，因为主分片需要给所有副本同步数据。所以建议最多设置1-2个即可。

    查看索引库某个分片占用磁盘空间大小

    ​	`curl -XGET  "localhost:9200/_cat/segments/test?v&h=shard,segment,size"`

### ES 优化3

 - 要定时对索引进行合并优化，不然segment越多，占用的segment memory越多，查询的性能也越差
    - 索引量不是很大的情况下可以将segment设置为1
    - es2.1.0以前调用\_optimize接口，后期改为\_forcemerge接口
    - `curl -XPOST 'http://localhost:9200/test/_forcemerge?max_num_segments=1'`
    - `client.admin().indices().prepareForceMerge("test").setMaxNumSegments(1).get();`
    - 索引合并是针对分片的。segment设置为1，则每个分片都有一个索引片段。
- 针对不使用的index，建议close，减少内存占用。因为之一索引处于open状态，索引库中的segement就会占用内存，close之后就只会占用磁盘空间了。
  - `curl -XPOST 'localhost:9200/test/_close'`



### ES 优化4

- 删除文档： 在es中删除文档，数据不会马上在硬盘中除去，而是在es索引中产生一个.del的文件，而在检索中这部分数据也会参与检索，es在检索过程会判断是否删除了，如果删除了再过滤掉。这样会降低检索效率。所以可以执行清除删除文档
- `curl -XPOST 'http://localhost:9200/test/_forcemerge?only_expunge_deletes=true'`
- `client.admin().indices().prepareForceMerge("test").setOnlyExpungeDeletes(true).get();`

### ES 优化5

- 如果在项目开始的时候需要批量入库大量数据的话，建议将副本数设置为0
  - 因为es在索引数据的时候，如果有副本存在，数据包也会马上同步到副本中，这样会对es增加压力。可以等索引完成后将副本按需要改回来，这样可以提高索引效率。

### ES需要注意的问题

使用java操作es集群的时候要保证本地使用的es的版本和集群上es的版本保持一致。

​	es集群的jdk版本不一致可能会导致 org.elasticsearch.transport.RemoteTransportException: Failed to deserialize exception response from stream




## Elasticsearch源码分析

- elasticsearch在建立索引时，根据id或(id，类型)进行hash，得到hash值之后再与该索引的分片数量取模，取模的值即为存入的分片编号。
- 可以指定把数据存储到某一个分片中，通过routing参数
  - `curl -XPOST 'localhost:9200/yehua/emp?routing=rout_param' -d '{"name":"zs","age":20}'`
  - routing（路由参数）
- 注意： 显著提高查询性能，routing，(急速查询)