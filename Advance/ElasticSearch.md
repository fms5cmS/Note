Elasticsearch是一个**分布式搜索服务**，提供Restful API，底层基于Lucene，采用多shard（分片）的方式保证数据安全，并且提供自动resharding的功能，github等大型的站点也是采用了ElasticSearch作为其搜索服务。

特点：分布式(故高可用)、支持多种数据类型、支持多种API、面向文档、异步写入、近实时。

开源的 [ElasticSearch](https://www.elastic.co/products/elasticsearch) 是目前全文搜索引擎的首选。可以快速的存储、搜索和分析海量数据。

- 结构化数据：具有固定格式或有限长度的数据，如数据库、元数据等；
- 非结构化数据：指不定长或无固定格式的数据，如邮件、word文档等。

非结构化数据的搜索：

- 顺序扫描法(Serial Scanning)，适用于小数据量的文件搜索
- 全文搜索(Full-text Search)，适合大数据量的文件搜索
  - 将非结构化数据的一部分信息提取出来重新组织，使其具有一定结构，然后对这个有一定结构的数据进行搜索。从非结构化数据种提取出来重新组织的信息称为索引
  - 原理：
    - 建文本库——>建立索引——>执行搜索——>过滤结果
  - 全文搜索实现技术：
    - 基于Java的开源实现：Lucene、ElasticSearch、Solr

学习文档：[Elasticsearch 权威指南](https://es.xiaoleilu.com/index.html)



# Docker安装

ElasticSearch默认进行Web通信使用9200端口，分布式的情况下各个节点之间使用9300端口通信！

下载ElasticSearch：

```shell
docker pull elasticsearch
# ElasticSearch 是使用Java编写的，启动时默认占用2G内存，故这里使用 -e 来限制它的堆内存使用
docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 --name ES01 elasticsearch
```

测试：在浏览器地址栏输入`ip地址:9200`会得到Json数据响应。



# 概念&存储

Elasticsearch 是 *面向文档* 的，意味着它存储整个对象或文档。 使用 JavaScript Object Notation 或者 [*JSON*](http://en.wikipedia.org/wiki/Json) 作为文档的序列化格式。

- ElasticSearch中的概念与数据库的概念类比：

  - 索引(名词) ———— 数据库
  - 索引(动词) ———— `insert`操作
  - 类型 ———— 表
  - 文档 ———— 表中的记录
  - 属性 ———— 字段

- 以存储员工文档为例：

  - 存储数据到ElasticSearch的行为叫做**索引(动词)**，而要想执行索引操作，需要先确定文档存储的位置：

    - 一个 ElasticSearch 集群可以包含多个**索引(名词)**；
    - 每个索引可以包含多个**类型**
    - 每个类型存储有多个文档

    ```
    请求方式：PUT 
    请求体：见下面
    路径：/索引名称/类型名称/特定员工ID
    ```

  - 一个**文档**代表一个员工数据，每个文档有多个**属性**。请求体：

    ```json
    {
        "first_name" : "John",
        "last_name" :  "Smith",
        "age" :        25,
        "about" :      "I love to go rock climbing",
        "interests": [ "sports", "music" ]
    }
    ```

    

# 检索

ElasticSearch 中存储了一些数据后，如何检索数据呢？

在 ElasticSearch 中检索很简单，执行一个 HTTP `GET`请求并指定文档的地址：索引库、类型、ID即可，如：

```http
GET /megacorp/employee/1
```

返回的数据，如：

```json
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```



使用`DELETE`命令来删除文档，删除成功，则状态码为200，否则为404，且都会有返回数据；

使用`HEAD`命令来检查文档是否存在，如果存在，则状态码为200，否则为404，都无返回数据。



## 轻量搜索

- 不再指定文档ID，而使用`_search`代替，如：

  ```http
  GET /megacorp/employee/_search
  ```

  - 会返回类型中所有的文档，放在数组hits中，默认一个搜索返回十条结果

- 高亮搜索(即条件查询)：

  - 在`_search`的基础上，添加一个查询字符串：`q=属性名:值`

  ```http
  GET /megacorp/employee/_search?q=last_name:Smith
  ```



## 复杂搜索

- 查询表达式搜索：即复杂的条件查询，如仍然查询所有的Smith搜索，使用请求体来代替查询字符串

  ```http
GET /megacorp/employee/_search
  ```
  
  ```json
{
      "query" : {
          "match" : {  //match查询，属于查询类型的一种，仅支持单个字段的查询
              "last_name" : "Smith"
          }
      }
  }
  ```



# SpringBoot整合

Spring Boot 通过整合 SpringData ElasticSearch 提供了非常便捷的检索功能支持

SpringBoot默认使用SpringData ElasticSearch模块操作。

- SpringBoot默认支持两种技术和ES交互：
  - Jest (默认不生效，需要导入jest的工具包 io.searchbox.client.JestClient)
  - SpringData ElasticSearch【ES版本可能不合适】
    - [版本适配](https://github.com/spring-projects/spring-data-elasticsearch)，如果版本不适配，有两种方式解决：
      1. 升级SpringBoot版本；
      2. 安装对应版本的ES。
    - 需要配置Client节点信息：clusterNodes、clusterName(默认为"elasticsearch")
    - 两种操作ES的方式：
      - `ElasticsearchTemplate`来操作ES
      - 编写一个`ElasticsearchRepository`的**子接口**来操作ES



## 使用Jest

[文档](https://github.com/searchbox-io/Jest/tree/master/jest)

去除spring-boot-starter-data-elasticsearch的依赖，加入：

```xml
<dependency>
    <groupId>io.searchbox</groupId>
    <artifactId>jest</artifactId>
    <version>5.3.3</version>
</dependency>
```

需要配置：username、password、uris(`List`类型，默认指向localhost的9200端口)，使用spring.elasticsearch.jest前缀指定:

```properties
spring.elasticsearch.jest.uris=http://192/168.214.132:9200
```

测试：

```java
    @Autowired
    JestClient jestClient;

    /**测试索引*/
    @Test
    public void index() {
        //给ES中索引(动词)一个文档
        Article article = new Article();
        article.setId(1);
        article.setAuthor("zhangsan");
        article.setTitle("好消息");
        article.setContent("Hello World");
        //构建一个索引功能
        //由于在Article中使用@JestId标识了id，所以这里在指定了索引(名词)、类型后不再指定id
        Index index = new Index.Builder(article).index("zzk").type("news").build();

        try {
            //执行
            jestClient.execute(index);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**测试搜索*/
    @Test
    public void search(){
        //构建查询表达式
        String json = "  {\n" +
                "      \"query\" : {\n" +
                "          \"match\" : {  //match查询，属于查询类型的一种\n" +
                "              \"content\" : \"hello\"\n" +
                "          }\n" +
                "      }\n" +
                "  }";
        //构建搜索功能
        Search search = new Search.Builder(json).addIndex("zzk").addType("news").build();
        try {
            SearchResult result = jestClient.execute(search);
            System.out.println(result.getJsonString());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```



## SpringData ElasticSearch

加入spring-boot-starter-data-elasticsearch模块。

配置Client节点信息：clusterNodes、clusterName(默认为"elasticsearch")：

```properties
# cluster-name可以通过在浏览器中输入 ip地址:端口 ，得到的Json数据中就有该属性
spring.data.elasticsearch.cluster-name=elasticsearch
# ElasticSearch 服务地址
spring.data.elasticsearch.cluster-nodes=192.168.*.*:9300
```



### 方式一

继承`ElasticsearchRepository`接口的方式。

1. 编写一个继承`ElasticsearchRepository`接口的子接口：

```java
/**
* 第一个泛型是要操作的数据类型
* 第二个泛型是第一个泛型类型的主键类型
* @author +1day
*/
public interface BookRepository extends ElasticsearchRepository<Book,Integer> {
    //不用实现方法也可以
}
```

2. 测试索引(动词)方法：

```java
@Autowired
BookRepository bookRepository;

@Test
public void test(){
    Book book = new Book();
    book.setId(1);
    book.setAuthor("zz");
    book.setBookName("aaaaaaaa");
    bookRepository.index(book);
}
```

3. 那么如何指定索引的索引和类型呢？在bean中通过注解指定：

```java
@Document(indexName = "zzk",type = "book")
public class Book implements Serializable { }
```



### 方式二

通过`ElasticsearchTemplate`操作ES。

见[文档](https://github.com/spring-projects/spring-data-elasticsearch)