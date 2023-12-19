# ES7为什么废弃type类型

>lxw1844912514
>
>2023-04-07
>
>https://blog.csdn.net/lxw1844912514/article/details/129980490

### 一、介绍

ES7之前是有type的，属于index下，一个index可以有不同的type，ES7开始就把type这个显示概念给删除了，统一换成了_doc来表示type。也就是ES7开始一个index只能有一个type，而且这个type还是默认的_doc。

### 二、type的底层存储

#### 1、概念讲解

##### 什么是类型(type)？

从Elasticsearch的第一个发布版本以来，每一个文档都被存储在一个单独的索引里，并被赋予了一个type，一个映射类型代表着一个被索引的文档或实体的类型。

document中field的value在底层的lucene中建立索引的时候全部是二进制类型的，因为Lucene是没有type概念的，所以在document中，实际上将type作为一个document的field来存储（_type字段），然后ES通过_type来进行type的过滤和筛选。

也就是说一个index中的多个type实际上是放到一起存储的，所以一个index下不能有多个重名的type。

#### 2、代码讲解

比如有如下两个结构的type：

```json
{
   "test_index": {
      "mappings": {
         "mobile": {
            "properties": {
               "name": {
                  "type": "string",
               },
               "price": {
                  "type": "double"
               },
                 "xxx": {
                      "type": "string"
                 }            

            }
         },
         "computer": {
            "properties": {
               "name": {
                  "type": "string",
               },
               "price": {
                  "type": "double"
               },
                 "yyy": {
                      "type": "string"
                 }
            }
         }
      }

   }
}
```

type为mobile的下面有一条document如下：

```json
{
  "name": "xiaomi",
  "price": 3999.99,
  "xxx": "abcd"
}
type为computer的下面有一条document如下：

{
  "name": "apple",
  "price": 9999.99,
  "yyy": "apple good"
}
```


上面那两条数据在底层存储mapping是下面这个样子的：

```json
{
   "test_index": {
      "mappings": {
        "_type": {
          "type": "string",
          "index": "not_analyzed"
        },
        "name": {
          "type": "string"
        }
        "price": {
          "type": "double"
        }
        "xxx": {
          "type": "string"
        }
        "yyy": {
          "type": "string"
        }
      }
   }
}
```

可以看到type其实就是一个_type字段，字符串类型，不分词。

上面那两条数据在底层存储document是下面这个样子的：

```json
{
  "_type": "mobile",
  "name": "xiaomi",
  "price": 3999.99,
  "xxx": "abcd",
  "yyy": ""
}
{
  "_type": "computer",
  "name": "apple",
  "price": 9999.99,
  "xxx": "",
  "yyy": "apple good"
}
```

#### 3、总结

1. type是document中的一个字段_type，且类型是string，不进行分词。
2. 底层会将当前index下所有type下的所有document的field都统一存放到了一个地方，对多个type的多个document进行了merge合并。
3. document1中没有yyy字段，但是也会将他放入其内，只是value为空字符串，document2同理。进一步证明了进行了merge。
4. 同一个index下的不同type下的document字段名若相同，则类型一样要相同，否则会冲突报错，因为他在merge的时候不知道你这个字段是什么类型。

### 三、ES7.x为什么删除了type？

假设在一个index中建立很多type，并且这些type下的document都没有相同的字段，那么会导致数据极其稀疏（因为会合并，没有的字段是空字符串），影响ES的存储、检索效率。

在一个index中建立很多type，其中有两个document属于不同的type，但是这两个document的field名称一样，数据类型不一样。这时候就报错了。

可以试一下，在同一个index中，不同的type，创建一个同名的字段，但是类型不要弄成一样的，看能否成功创建。答案是不可以，它会提示你，不可以将这个字段的类型更改为这个类型。

```
illegla_argument_exception

mapper [create_time_] cannot be changed from type [date] to [keyword]
```


所以，结论就是，es确实把不同type中的同名字段，当成了一个字段。

在设计索引库的时候，同名问题一定要注意，最简单的方法就是一个index，一个type，想要其他类型，另外创建index，当然你可以用别的字段名。

注意：ES7废弃，但还在用，ES8才真正的去掉了type。

### 四、type在Elastic 8.x中

8.0是一个大版本，ES历经3年终于从7.x来到了8.x时代。

8.x最主要的一个变化是，终于将type从ES代码中去掉了。

##### 大概的思路如下：

- 在6.x，ES加上了index.mapping.single_type: true的默认设置，强制用户只能使用一个type字段。如果用户还需要有多个type的需求，那么需要显式把index.mapping.single_type设置为false。

- 在6.x，type建议用户设置为_doc，这是为接下来_doc作为一个常量准备的，ES的思路是API从PUT {index}/{type}/{id}，这种改成{index}/_doc/{id}。

- 7.x去掉了index.mapping.single_type配置，type只能设置一个，且为_doc。

- 7.x 在mapping中把type这层去掉，这时候如果用户有兼容性问题，支持加上include_type_name为true，增加type这层，但是名称只能为_doc。

- 8.x去掉include_type_name参数，且代码中不在依赖多 type能力。