---
layout: post
title: 优化task-log索引
categories: [elasticsearch]
tags: [elasticsearch]
fullview: true
markdown: kramdown

---

### 背景

![task log data](/assets/media/WX20200323-135441.png)

索引表数据过大，基于场景，分析该表就是统计任务写入状态使用,便于每天汇总报告.

### 解决方案

利用Index Template 格式如下：

```
PUT _template/task_log
{
	"index_patterns": ["task-log-*"],
	"settings": {
		"number_of_shards": 3
	},
	"mappings": {
		"_source": {
			"enabled": false
		},
		"dynamic": false,
		"properties": {
			"id": {
				"type": "long"
			},
			"success": {
				"type": "boolean"
			},
			"createtime": {
				"type": "date",
				"format": "yyyy-MM-dd HH:mm:ss"
			}
		}
	},
	"aliases": {
	  "task-log": {}
	},
	"version": 2
}
```

按天创建索引,而且支持动态创建（task_log Template)

```
POST task-log-20200325/_doc
{
  "id":1011221,
  "success":false,
  "createtime":"2020-12-12 13:12:12"
}
```


分析数据（每天的成功个数和失败个数)

```
GET task-log/_search
{
  "size": 0,
  "aggs": {
    "avg_day": {
      "date_histogram": {
        "field": "createtime",
        "interval": "day"
      },
      "aggs": {
        "success": {
          "terms": {
            "field": "success",
            "size": 10
          }
        }
      }

    }
  }
}
```

结果:

```
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 7,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "avg_day" : {
      "buckets" : [
        {
          "key_as_string" : "2020-12-12 00:00:00",
          "key" : 1607731200000,
          "doc_count" : 7,
          "success" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : 1,
                "key_as_string" : "true",
                "doc_count" : 5
              },
              {
                "key" : 0,
                "key_as_string" : "false",
                "doc_count" : 2
              }
            ]
          }
        }
      ]
    }
  }
}
```

基本功能实现了，原来的`_source`数据不存在了，一条省个及字节，一天几亿条,那就节省不少空间。


### 优化点

1. `_source enable false`
2. `index template` 支持`version, alias, mapping`
3. 移除副本配置，缩小不必要的数据空间
4. 一个存储日志的主分片理论值50G，三个主分片可容纳150G数据，基于现有每天的数据量按每条1KB，可容纳150亿条数据，足矣满足现在业务使用场景。
