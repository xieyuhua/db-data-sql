ElasticSQL
-----------
[![Build Status](https://travis-ci.org/farmerx/elasticsql.svg?branch=master)](https://travis-ci.org/farmerx/elasticsql)
[![Go Documentation](http://img.shields.io/badge/go-documentation-blue.svg?style=flat-square)](https://godoc.org/github.com/farmerx/elasticsql)
[![Coverage Status](https://coveralls.io/repos/github/farmerx/elasticsql/badge.svg?branch=master)](https://coveralls.io/github/farmerx/elasticsql?branch=master)
[![Go Report Card](https://goreportcard.com/badge/github.com/farmerx/elasticsql)](https://goreportcard.com/report/github.com/farmerx/elasticsql)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg?maxAge=2592000)](https://github.com/farmerx/elasticsql/blob/master/LICENSE)

> ElasticSQL package converts SQL to ElasticSearch DSL

## SQL Features Support:

- [x] SQL Select
- [x] SQL Where
- [x] SQL Order By
- [x] SQL Group By
- [x] SQL AND & OR
- [x] SQL Like & NOT Like
- [x] SQL COUNT distinct   count(distinct(mid))
- [x] SQL In & Not In
- [x] SQL Between
- [x] SQL avg(field)、count(*), count(field), min(field), max(field)

## Beyond SQL Features Support：
- [x] ES TopHits
- [x] ES date_histogram    |||   `date_histogram(field="changeTime", _interval="1h", format="yyyy-MM-dd HH:mm:ss")`
- [x] ES histogram      ||| `histogram(field="grade", _interval="10")`
- [x] ES STATS           ||| `stats(field="grade")`
- [x] ES RANGE           ||| `range(field="age", range="20,25,30,35,40")`
- [x] ES DATE_RANGE      |||  `date_range(field="insert_time", format="yyyy-MM-dd" ,range="2014-08-18, 2014-08-17, now-6d,now")`



*Improvement : now the query DSL is much more flat*


## SQL Usage
Query
```
select * from test where a=1 and b="c" and create_time between '2015-01-01T00:00:00+0800' and '2016-01-01T00:00:00+0800' and process_id > 1 order by id desc limit 100,10
```
Aggregation
```
select avg(age),min(age),max(age), count(student), count(distinct student) from test group by grade,class limit 10
```
Beyond SQL
 * range age group 20-25,25-30,30-35,35-40
	```
	SELECT COUNT(age) FROM bank GROUP BY range(field="age", range="20,25,30,35,40")
	```
 * range date group by your config
 	```
	SELECT online FROM online GROUP BY date_range(field="insert_time",format="yyyy-MM-dd" ,range="2014-08-18,2014-08-17,now-8d,now-7d,now-6d,now")
	```
 * range date group by day

	```
	select * from test group by date_histogram(field="changeTime", _interval="1h", format="yyyy-MM-dd HH:mm:ss")
	```
 * stats
 	```
	 SELECT online FROM online group by stats(field="grade")
	```
 * topHits
 	```
	  select top_hits(field="class", hitssort="age:desc", taglimit = "10", hitslimit = "1", _source="name,age,class,gender") from school
	```


## PKG Usage
-------------

> go get github.com/farmerx/elasticsql

Demo :
```go
package main

import (
    "fmt"
    "github.com/farmerx/elasticsql"
)

var sql = `
select * from test where a=1 and b="c" and create_time between '2015-01-01T00:00:00+0800' and '2016-01-01T00:00:00+0800' and process_id > 1 order by id desc limit 100,10
`

var sql2= `
  select avg(age),min(age),max(age),count(student) from test group by class limit 10
`
var sql3= `
  select * from test group by class,student limit 10
`
var sql4 = `
  select * from test group by date_histogram(field="changeTime",interval="1h",format="yyyy-MM-dd HH:mm:ss")
`


func main() {
    esql := elasticsql.NewElasticSQL(eslasticsql.InitOptions{})
    table, dsl, err := esql.SQLConvert(sql)
	fmt.Println(table, dsl, err)
}

```

## OUTPUT
```
date_historgram
{
    "query": {
        "bool": {
            "must": [
                {
                    "match_all": {}
                }
            ]
        }
    },
    "from": 0,
    "size": 0,
    "aggregations": {
        "date_histogram": {
            "date_histogram": {
                "field": "changeTime",
                "format": "yyyy-MM-dd HH:mm:ss",
                "interval": "1h"
            }
        }
    }
}

date_range
{
    "query": {
        "bool": {
            "must": [
                {
                    "match_all": {}
                }
            ]
        }
    },
    "from": 0,
    "size": 0,
    "aggregations": {
        "date_range": {
            "range": {
                "field": "insert_time",
                "ranges": [
                    {
                        "format": "yyyy-MM-dd",
                        "from": "2014-08-18",
                        "to": "2014-08-17"
                    },
                    {
                        "format": "yyyy-MM-dd",
                        "from": "2014-08-17",
                        "to": "now-8d"
                    },
                    {
                        "format": "yyyy-MM-dd",
                        "from": "now-8d",
                        "to": "now-7d"
                    },
                    {
                        "format": "yyyy-MM-dd",
                        "from": "now-7d",
                        "to": "now-6d"
                    },
                    {
                        "format": "yyyy-MM-dd",
                        "from": "now-6d",
                        "to": "now"
                    },
                    {
                        "format": "yyyy-MM-dd",
                        "from": "now"
                    }
                ]
            }
        }
    }
}
...
```

## Licenses

This program is under the terms of the MIT License. See [LICENSE](https://github.com/farmerx/elasticsql/blob/master/LICENSE) for the full license text.

