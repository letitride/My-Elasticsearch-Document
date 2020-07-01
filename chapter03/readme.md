#### Elasticsearchを構成する概念

##### ドキュメント

Elasticsearchに格納するデータの単位。RDBでいうところのレコードの概念。

primary keyとなるIDで管理され、ドキュメント作成時に指定可能。指定しなければ自動で採番される。

##### フィールド

* text型 格納する際、単語ごとに分割され、単語ごとの検索が可能
* keyword型 格納した文字列を「完全一致」で検索する用途で使用する。
* 数値型 (long, short, integer, float)
* date型
* boolean型
* object, array JSONフォーマット

##### インデックス

データを格納する場所を指す。RDBでいうところのテーブルに近い？

##### ドキュメントタイプ

インデックスのフィールド構造に名称をつけたものを指す。

こちらもテーブルに近い概念だが1インデックスにつき1ドキュメントタイプなので、両者を複合して考えると楽そう。

Elasticsearch7 からタイプレスとなった。

https://www.elastic.co/jp/blog/moving-from-types-to-typeless-apis-in-elasticsearch-7-0

##### マッピング

ドキュメントタイプに対する構造定義を記載したもの。

記述をしない場合、Elasticsearchが推測してマッピングを作成する。

##### シャード

インデックスを分割する単位。インデックス作成時に指定。後から分割できない。

分割したインデックスは異なるノードで分散保持される。

デフォルトのシャード数は5

目安として、20〜30GBのインデックス容量を1シャード賄えるらしい。

##### レプリカ

ノード障害時の可用性、及び検索分散としてシャードのレプリケーションを持つことができる。

こちらは後からでもレプリカ数を変更できる.


#### Elasticsearchの基本操作

##### ドキュメントの作成

```
$ curl -XPUT 'http://localhost:9200/my_index/_doc/1' -H 'content-Type: application/json' -d'
{
  "user_name": "John Smith",
  "date": "2017-10-15T15:09:45",
  "message": "Hello Elasticsearch world."
}
'
```

##### ドキュメントの取得

```
$ curl -XGET http://localhost:9200/my_index/_doc/1
```
```
{"_index":"my_index","_type":"_doc","_id":"1","_version":1,"_seq_no":0,"_primary_term":1,"found":true,"_source":
{
  "user_name": "John Smith",
  "date": "2017-10-15T15:09:45",
  "message": "Hello Elasticsearch world."
}
}
```

```
$ curl -XGET http://localhost:9200/my_index/_doc/1/_source
```
```
{
  "user_name": "John Smith",
  "date": "2017-10-15T15:09:45",
  "message": "Hello Elasticsearch world."
}
```

##### ドキュメントの検索

```
$ curl -XGET http://localhost:9200/my_index/_search?pretty -H 'Content-Type: application/json' -d '
{
  "query": {
    "match": {
      "message": "Elasticsearch"
    }
  }
}
'
```
```
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 0.2876821,
        "_source" : {
          "user_name" : "John Smith",
          "date" : "2017-10-15T15:09:45",
          "message" : "Hello Elasticsearch world."
        }
      }
    ]
  }
}
```

##### ドキュメントの更新

```
$ curl -XPUT 'http://localhost:9200/my_index/_doc/1' -H 'Content-Type: application/json' -d '
{
  "user_name": "Mike Stuart",
  "date": "2017-11-04T16:36:12",
  "message": "This message was updated."
}
'
```

##### ドキュメントの一部更新

```
$ curl -XPOST 'http://localhost:9200/my_index/_doc/1/_update' -H 'Content-Type: application/json' -d '
{
  "doc": {
    "message": "Only message was updated."
  }
}
'
```

##### ドキュメントの削除

```
$ curl -XDELETE 'http://localhost:9200/my_index/_doc/1'
```

##### インデックスの作成

```
$ curl -XPUT 'http://localhost:9200/my_index' -H 'Content-Type: application/json' -d '
{
  "settings": {
    "number_of_shards": "3",
    "number_of_replicas": "2"
  }
}
'
```

##### インデックスの確認

```
$ curl -XGET 'http://localhost:9200/my_index'
```

##### レプリカ数の変更

```
$ curl -XPUT 'http://localhost:9200/my_index/_settings' -H 'Content-Type: application/json' -d '
{
  "index": {
    "number_of_replicas": "1"
  }
}
'
```

##### インデックスの削除

```
$ curl -XDELETE 'http://localhost:9200/my_index'
```

##### マッピングの作成

```
$ curl -XPUT 'http://localhost:9200/my_index' -H 'Content-Type: application/json' -d '
{
  "mappings": {
    "properties": {
      "user_name": { "type": "keyword" },
      "date": {"type": "date"},
      "message": {"type": "text"}
    }
  }
}
'
```

##### マッピング定義の確認

```
$ curl -XGET 'http://localhost:9200/my_index/_mapping?pretty'
```

#### クエリ操作

基本型
```
$ curl -XGET 'http://localhost:9200//<インデックス名>/_search'
```

パラメータ指定 (Query String)
```
$ curl -XGET 'http://localhost:9200//<インデックス名>/_search?q=<フィールド名>:<検索ワード>'
```

クエリDSL
```
$ curl -XGET 'http://localhost:9200/<インデックス名>/_search'  -H 'Content-Type: application/json' -d '<クエリDSL>'
```

DSLシンタックス
```
{
  "query": {<クエリ内容>},
  "from": 0,
  "size": 10,
  "sort": [<検索結果のソート指定>],
  "_source": [<検索結果として返すフィールド名>]
}
```

* query クエリ内容とするJSONオブジェクト
* from / size 所謂、offset/limit。 default 0/10

#### 全文検索クエリ

##### match_all

総scan
```
{
  "query": {
    "match_all": {}
  }
}
```
##### match

```フィールド名:検索ワード```で指定。検索ワードは空白で区切ることで「OR」検索となる。
```
{
  "query": {
    "match": {
      "message": "Elasticsearch world"
    }
  }
}
```
また、operatorを指定することで「AND」で検索することもできる。
```
{
  "query": {
    "match": {
      "message": {
        "query": "Elasticsearch world",
        "operator": "and"
      } 
    }
  }
}
```
N個以上のキーワードが含まれる。という指定
```
{
  "query": {
    "match": {
      "message": {
        "query": "Elasticsearch world",
        "minimum_should_match": 2
      } 
    }
  }
}
```

##### match_phrase

matchに指定した語順の並び順までをチェックする

```
{
  "query": {
    "match_phrase": {
      "message": "Elasticsearch world"
    }
  }
}
```

#### Termクエリ (完全一致)

keywordフィールドに対して使用する。

```
{
  "query": {
    "term": {
      "user_name": "John Smith"
    }
  }
}
```

##### Termsクエリ いずれかが完全一致するか？の検索

```
{
  "query": {
    "terms": {
      "user_name": ["John Smith", "Terry Man"]
    }
  }
}
```

#### Rangeクエリ

日付型や数値型の範囲検索を行う。

```
{
  "query": {
    "range": {
      "date": {
        "gte": "2017-10-14T15:09:45",
        "lte": "2017-10-16T15:09:45"
      }
    }
  }
}
```

date型はシステム日付と日付計算式が使用できる。

* now
* y 年
* M 月
* w 週
* d 日
* h 時
* m 分
* s 秒

一週間前までのデータを検索
```
{
  "query": {
    "range": {
      "date": {
        "gte": "now-1w"
      }
    }
  }
}
```

#### boolクエリ

* must すべての式を満たす必要がある
* should いずれかの式、またはminimum_should_match数のを満たす必要がある。
* must_not この式を満たすドキュメントは除外する
* filter スコアに関係なく式に一致したドキュメントを返す　検索範囲を限定したり、スコアが必要ないクエリであれば高速に動作する

```
{
  "query": {
    "bool": {
      "must":[
        {"match": {"message": "elasticsearch"} },
        {"term": {"user_name": "John Smith"} }
      ],
      "should":[
        {"match": {"message": "world"} },
        {"term": {"user_name": "Terry man"} }
      ],
      "must_not": [
        {"match": {"message": "Haskell"} }
      ]
    }
  }
}
```

#### sort

```
{
  "query": {
    "match_all": {}
  },
  "sort": [
    { "date": {"order": "desc"} },
    "_score"
  ]
}
```

配列は集約結果でsortすることができる
```
{
  "query": {
    "match_all": {}
  },
  "sort": [
    { "dayoff": {"order": "desc", "mode": "avg"} }
  ]
}
```

* avg
* min
* max
* sum
* median
