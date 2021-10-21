## ES SearchAPI

### 指定查询的索引

| 语法                   | 范围              |
| ---------------------- | ----------------- |
| /_search               | 集群上所有的索引  |
| /index1/_search        | index1            |
| /index1,index2/_search | index1,index2     |
| /index*/_search        | 以index开头的索引 |



### URL 查询

- 使用 ”q“，指定查询字符串
- ”name:zhangsan“,KV 健值对

```
curl -XGET

"http://localhost:9200/index/_search?q=name:zhangsan"
```

q:表示查询内容

```
GET index/_search?q=2020&df=title&sort=year:desc&from=0&size=10&timeout=1s
{
	"profile":true
}
```

- q:指定查询语句，使用 Query String Syntax 
- df:默认查询字段，不指定，会查询所有字段
- sort:排序 from 和 size 用于分页
- profile:查询执行计划

### Query String Syntax 

#### 	指定字段 v.s 泛查询

​		q=title:2020  / q=2020

#### 	Term v.s Phrase

​	  （Beautiful Mind ）等于 Beautiful OR Mind

​		"Beautiful Mind"      等于 Beautiful AND Mind 。Phrase查询，还要求顺序一致

```
// Term 查询
GET index/_search?q=title:Beautiful Mind
{
	"profile":true
}

// Phrase 查询
GET index/_search?q=title:"Beautiful Mind"
{
	"profil":true
}

```

#### 布尔查询

​	AND / OR / NOT / 或者 && / || / !

- 必须大写

- title:(魔戒 NOT 霍比特)


```
// bool 查询 结果必须包含Beautiful，Mind
GET index/_search?q=title:(Beautiful AND Mind)
{
	"profile":true
}
```

#### 分组

+ +表示must
+ -表示must_not
+ title:(+魔戒  -霍比特)

#### 范围查询

- 区间表示：[] 闭区间，{} 开区间
  - year:{2019 TO 2020]
  - year:[* TO 2019]

#### 算数符号

- year:>2010
- year:(>2010 && <=2020)
- year:(+>2010 +<=2020)

```
GET index/_search?q=year:>2010
{
	"profile":true
}
```

#### 通配符查询（效率低，占用内存大）

- ？表示一个字符，*表示0或者多个字符
  - title:he?lo
  - title:w*

#### 正则

- title:[bt]oy

#### 模糊匹配与近似查询

- title:beautifl~1
- title:"lord rings"~2

```
title:beautifl~1         可以查询出  beautiful  
title:"lord rings"~2     可以查询出  lord of the rings
```

### Request Body Search

- 将查询语句通过http request body 发给elasticsearch
- Query DSL

```
POST index/_search
{
	"profile":true,
	"query":{
		"match_all":{}
	}
}
```

#### 排序

```
POST index/_search
{
	"sort":[{"create_date":"desc"}],
	"from":1,
	"size":10,
	"query":{
		"match_all":{}
	}
}
```

#### 过滤返回结果

```
POST index/_search
{
	"_source":["order_date","categroy.keyword"],
	"from":1,
	"size":10,
	"query":{
		"match_all":{}
	}
}
```

#### 脚本字段

```
GET index/_search
{
	"script_fields":{
		"new_field":{
			"script":{
				"lang":"painless",
				"source":"doc['order_date'].value+'_hello'"
			}
		}
	},
	"from":1,
	"size":10,
	"query":{
		"match_all":{}
	}
}

生成一个new_field:20200202_hello 字符拼接
```

#### 使用查询表达式 - Match

```
POST index/_search
{
	"query":{
		"match":{
			"comment":"Last Date"
		}
	}
}

POST index/_search
{
	"query":{
		"match":{
                "comment":{
				"query":"Last Date",
				"operator":"AND"
			}
		}
	}
}
```

#### 短语搜索 - Match Phrase

```
POST index/_doc/_search
{
	"query":{
		"match_phrase":{
			"comment":{
				"query":"Last Date",
				"slop":1
			}
		}
	}
}
```

- Last Date 必须顺序出现 
- "slop":1 表示中间可以有一个字符

#### Query String Query

类似URLquery 		

```
POST index/_search
{
	"query":{
		"query_string":{
			"default_field":"name",
			"query":"jim AND tom"
		}
	}
}

POST index/_search
{
	"query":{
		"query_string":{
			"default_field":["name","about"],
			"query":"(jim AND tom) OR (java AND ES)"
		}
	}
}

```

#### Simple Query String Query

- 不支持 AND OR NOT 
- Term 之间默认是OR,可以指定Operator
- 支持部分逻辑
  - +代替 AND
  - |代替 OR
  - -代替NOT

```
POST index/_search
{
	"query":{
		"simple_query_string":{
			"query":"jim yangyang",
			"fields":["name"],
			"default_operator":"AND"
		}
	}
}


```

### DynamicMapping

#### 字段的数据类型

- 简单类型
  - Text/Keyword
  - Date
  - Integer/Floating
  - Boolean
  - IPv4 & IPv6

- 复杂类型 - 对象和嵌套对象
- 特殊类型
  - geo_point & geo_shape / percolator

```
//写入文档
PUT index/_doc/1
{
	"firstName":"xiaogang",
	"lastName":"xiaoli",
	"loginDate":"2018-01-18T11:11:11.103Z"
}

//查看 Mapping文件
GET index/_mapping

//删除index
DELETE index
```

#### 能否更改Mapping的字段类型

- 新增字段
  - Dynamic设为true时，一旦有新增字段的文档写入，Mapping也同时会被更行
  - Dynamic设为false时，Mapping不会被更新，新增字段数据无法被索引，但信息回出现在_source中
  - Dynamic设为Strict时，文档写入失败

- 修改已有字段，一旦有数据写入，就无法修改字段定义
- 如果要修改字段类型，需要Reindex API,重建索引

|               | true | false | strict |
| ------------- | ---- | ----- | ------ |
| 文档可索引    | YES  | YES   | NO     |
| 字段可索引    | YES  | NO    | NO     |
| Mapping被更新 | YES  | NO    | NO     |

新增Mapping

```
PUT movies
{
	"mappings":{
		"_doc":{
			"dynamic":"false"
		}
	}
}
```

修改Mapping

```
PUT movies/_mapping
{
	"dynamic":"false"
}
```

#### Index Options

```json
PUT users
{
	"mappings":{
		"properties": {
          	"name":{
                "type":"text"
            },
            "mobile":{
              	"type":"text",
                "index":false
            },
            "bio":{
                "type":"text",
                "index_options":"offsets"
            }
	}
}
```

- 四种级别 Index Option 配置
  - docs - 记录doc id
  - freqs - 记录doc id 和 trem frequencies
  - positions - 记录doc id / trem frequencies / term position
  - offsets - 记录doc id / trem frequencies / term position / character offsets 

- Text 默认记录类型为positions,其他默认为docs

#### null_value

```
PUT users
{
	"mappings":{
		"properties": {
          	"name":{
                "type":"text"
            },
            "mobile":{
              	"type":"keyword",
                "null_value":"NULL"
            }
	}
}
```

- 需要对null搜索
- 只有keyword类型支持设定null_value

#### copy_to

```json
PUT users
{
	"mappings":{
		"properties": {
          	"first_name":{
                "type":"text",
                "copy_to":"fullName"
            },
            "last_name":{
              	"type":"text",
                "copy_to":"fullName"
            }
	}
}

GET users/_search?q=fullName:(li xiaogang)
```

- _all 在 7 中被copy_to 替代
- copy_to 字段的值会被拷贝到目标字段
- copy_to 的目标字段不会出现在_source中

#### 数组类型

```
PUT users/_doc/1
{
	"name":"xiaogang",
	"interests":["music","reading"]
}
```

- ES 没有专门的数组类型，但是任何字段都可以包含多个相同类型的值

#### 什么是聚合（Aggregation）

- Bucket Aggregatioin -- 满足特定条件的文档集合
- Metric Aggregation -- 数字运算，对文档进行统计分析
- Pipeline Aggregation -- 对其他的聚合结果进行二次聚合
- Matric Aggregation -- 支持多个字段的操作，并提供一个结果矩阵

### Update/Create

#### 部分字段修改操作

```
POST es_member_activity_goods_for_search/type/1/_update
{
  "doc": {
    "saleStatus":1,
    "isAppShow":1,
    "storeStatus":2
  }
}

```

#### 覆盖原来文档内容，及映射

```
PUT school_alias/_doc/1
{
  "id":1,
  "name":"西河一中",
  "address":"西河"
}

```



### Bulk

#### 批量修改数据

| 行为     | 解释                                                   |
| -------- | ------------------------------------------------------ |
| `create` | 当文档不存在时创建之。详见《创建文档》                 |
| `index`  | 创建新文档或替换已有文档。见《索引文档》和《更新文档》 |
| `update` | 局部更新文档。见《局部更新》                           |
| `delete` | 删除一个文档。见《删除文档》                           |

```
POST middle_school/_doc/_bulk
{"create":{"_id":4}}
{"id":"4","name":"陵南中学","address":"神奈川"}
{"create":{"_id":5}}
{"id":"5","name":"海南附中","address":"神奈川"}
```



### Delete

#### 删除部分文档数据

```
POST es_member_activity_goods_for_search/content/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}

```



### Alias

#### 创建别名

```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "school_index", ----元索引
        "alias": "school_alias"  ----别名
      }
    }
  ]
}
```

#### 删除别名（只删除元索引关联的别名）

```
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "school_index", ----元索引
        "alias": "school_alias"  ----别名
      }
    }
  ]
}                                                                           
```

```
DELETE /{index}/_alias/{name}  
```

#### 修改别名

```
POST /_aliases
{
  "actions": [
    {
      "remove": {
        "index": "school_index",
        "alias": "school_alias"   ----删除之前索引别名
      }
    },
    {
      "add": {
        "index": "school_index",
        "alias": "school_Alias"   ----添加新的索引别名
      }
    }
  ]
}

```

#### 查询索引与别名的映射关系

```
GET school_index/_alias   ---- 索引名查询
GET school_Alias/_alias   ---- 别名查询

返回结果：

{
  "school_index": {
    "aliases": {
      "school_Alias": {}
    }
  }
}
```

#### 别名关联多条索引

```
POST /_aliases                   ----------- 法一：分别添加
{
  "actions": [
    {
      "add": {
        "index": "school_index",
        "alias": "ss_alias"
      }
    },
    {
      "add": {
        "index": "student_index",
        "alias": "ss_alias"
      }
    }
  ]
}

```

```
POST /_aliases                    ----------- 法二：一次添加
{
  "actions": [
    {
      "add": {
        "indices":["school_index","student_index"],
        "alias": "ss_alias"
      }
    }
  ]
}
```

### RoutingKey

#### 添加路由，唯一绑定可修改索引（is_write_index）

```
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "school_index",
        "alias": "ss_alias",
        "routing": "scho",
        "is_write_index": true
      }
    },
    {
      "add": {
        "index": "student_index",
        "alias": "ss_alias"
      }
    }
  ]
}

注： "routing": "scho",  设置路由健：所有ss_alias上的CRUD都会被路由到同一个shard上
注： "is_write_index": true 当别名绑定多个索引时，被设置is_write_index的索引可以被别名修改，而且一个别名只能在一个索引上设置is_write_index；
```

#### 路由查询

```
GET ss_alias/_search?routing=scho
{
  "explain": true
}

只会返回school_index，同一分片上的数据，去掉?routing=scho可以返回别名下所有索引的数据
```

### Aggregation

#### Metrics Aggregation (指标)

##### avg,sum,max,min

```
GET middle_school_alias/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "stuSum": {
      "sum": {
        "field": "stuNum"
      }
    }
  }
}
注：stuSum:为自定义，sum:求和关键字，stuNum:聚合字段
```

##### value_count （字段统计）

```
GET middle_school_alias/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "nameCount": {
      "value_count": {
        "field": "name.keyword"
      }
    }
  }
}
注：类似sql中的select count("name") 统计字段“name”的个数
```

##### stats （统计聚合）

```
GET middle_school_alias/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "stuNumStatas": {
      "stats": {
        "field": "stuNum"
      }
    }
  }
}
注：统计stuNum的 "count","min","max","avg","sum"
```

##### cardinality （去重统计）

```
GET middle_school_alias/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "nameCount": {
      "cardinality": {
        "field": "address.keyword"
      }
    }
  }
}
注：统计address去重后的个数
```

#### Bucket Aggregation (桶)

##### terms (词聚合)

```
GET middle_school_alias/_search
{
  "query": {
    "match_all": {}
  },
  "aggs": {
    "stuNun_terms": {
      "terms": {
        "field": "address.keyword",
        "size": 10,
        "order": {
          "_count": "desc"
        },
        "min_doc_count": 2,
        "include": "神.*",
        "exclude": "U.*"
      }
    }
  }
}
注：field 参与聚合字段；
注：size 	定义返回buckets的size，默认全部返回；
注：order （_term）:key值进行排序；（_count）:doc_count进行排序；
注：min_doc_count 返回doc_count不能少于该设定值；
注：include 返回key值需包含该设定值
注：exclude 返回key值不能含有该设定值

注：条件field必填，其他看需要

返回例子：
  "aggregations": {
    "stuNun_terms": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "神奈川",
          "doc_count": 3
        }
      ]
    }
  }
```

##### filter 

```
GET middle_school_alias/_search
{
  "aggs": {
    "t_shirts": {
      "filter": {
        "term": {
          "address.keyword": "神奈川"
        }
      },
      "aggs": {
        "stuNum_avg": {
          "avg": {
            "field": "stuNum"
          }
        }
      }
    }
  }
}
注：过滤聚合。基于一个条件，来对当前的文档进行过滤的聚合

返回例子：
  "aggregations": {
    "t_shirts": {
      "doc_count": 3,
      "stuNum_avg": {
        "value": 2377.3333333333335
      }
    }
  }
```

##### range (范围聚合)

```
GET middle_school_alias/_search
{
  "aggs": {
    "stuNum_R": {
      "range": {
        "field": "stuNum",
        "ranges": [
          {
            "to": 500
          },
          {
            "from": 500
          }
        ]
      }
    }
  }
}
注：分组返回 [from,to) 前闭后开原则，分[0,500),[500,~)两组返回统计个数

返回例子：
 "aggregations": {
    "stuNum_R": {
      "buckets": [
        {
          "key": "*-500.0",
          "to": 500,
          "doc_count": 1
        },
        {
          "key": "500.0-*",
          "from": 500,
          "doc_count": 4
        }
      ]
    }
  }
```

#### Pipeline Aggregation (管道)

#### Matrix Aggregation (矩阵)



#### 角本设置 

```
PUT _cluster/settings
{
    "transient" : {
        "script.max_compilations_rate" : "100/1m"
    }
}

GET _cluster/settings

ElasticSearch 默认： 5分钟内执行脚本编译超过75个


PUT _settings
{
  "index": {
    "max_result_window": 20000
  }
}


```











