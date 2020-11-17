## 4ES SearchAPI

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









































