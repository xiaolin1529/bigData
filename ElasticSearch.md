# ElasticSearch

## Elasticsearch简介及安装部署

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

    - 查询类型searchType

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

## Elasticsearch 查询详解(置顶查询,高亮,聚合等...)

## Elasticsearch的settings和mappings详解

## Elasticsearch的分片查询方式

## Elasticsearch的脑裂问题分析

## Elasticsearch扩展之索引模板和索引别名

## Elasticsearch参数优化

## Elasticsearch源码分析