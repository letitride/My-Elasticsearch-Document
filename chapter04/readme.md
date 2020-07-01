#### Analyzer

Analyzerは以下の処理を行うことを目的とした機能。

* 全文検索で利用する為にインデックス、クエリ文字列を単語分割する処理。
* 言語固有の表記揺れの吸収

表記揺れとは？

* ステミング 語形の変化 making -> make, dogs -> dog, 食べる|食べた -> 食べ
* 正規化 大文字小文字, カタカナひらがな, 全角半角
* ストップワード the of など

#### Analyzerの定義

```
$ curl -XPUT 'http://localhost:9200/my_index' -H 'Content-Type: application/json' -d '
{
  "mappings": {
    "properties": {
      "blog_message": {
        "type": "text",
        "analyzer": "standard"
      }
    }
  }
}
'
```

##### カスタムAnalyzer

Analyzerは

* char_filter 入力文字列を別の文字列に置換 ```<br>``` -> ```¥n```
* tokenizer ルールに従って単語を分割
* filter tokenizerで分割した単語をルールに従ってステミングや変換を行う

で構成される。各要素の処理を定義することができる。

```
$ curl -XPUT 'http://localhost:9200/my_index' -H 'Content-Type: application/json' -d '
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": ["lowercase", "stop"]
        }
      }
    }
  }
}
```

カスタムfilterは以下のように定義できる。
```
$ curl -XPUT 'http://localhost:9200/my_index' -H 'Content-Type: application/json' -d '
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": ["lowercase", "my_stop"]
        }
      },
      "filter": {
        "my_stop": {
          "type": "stop",
          "stopwords": ["the", "a", "not", "is"]
        }
      }
    }
  }
}
'
```

##### analyzerの確認

```
$ curl -XGET 'localhost:9200/my_index/_analyze?pretty' -H 'Content-Type: application/json' -d '{"analyzer": "my_analyzer", "text": "this is a pen"}'
```

#### 日本語を扱うAnalyzerの導入

kuromojiというpluginを利用します。

```
$ cd /usr/share/elasticseaech
$ sudo  bin/elasticsearch-plugin install analysis-kuromoji
```

##### アップデートされた辞書のplugin 

少し古いelasticsearchまでしか対応してないのでversionに注意

```
$ sudo  bin/elasticsearch-plugin install org.codelibs:elasticsearch-analysis-kuromoji-ipadic-neologd:7.2.0
```
https://github.com/codelibs/elasticsearch-analysis-kuromoji-ipadic-neologd


#### kuromoji analyzerの利用

```
$ curl -XPUT 'http://localhost:9200/my_index' -H 'Content-Type: application/json' -d '
{
  "mappings": {
    "properties": {
      "user_name": {
        "type": "text",
        "analyzer": "kuromoji"
      }
    }
  }
}
'
```

```
$ curl -XGET 'localhost:9200/my_index/_analyze?pretty' -H 'Content-Type: application/json' -d '{"analyzer": "kuromoji", "text": "私は日々貯金したお金で、近々、関西国際空港からアメリカ　テキサス州へ旅行に行きます。常々夢見ていたので、とても楽しみで、このコンピューターで予約しました。料金は九万円ほどでした"}'
```

```
{
  "tokens" : [
    {
      "token" : "私",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "日々",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "貯金",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "お金",
      "start_offset" : 8,
      "end_offset" : 10,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "近々",
      "start_offset" : 12,
      "end_offset" : 14,
      "type" : "word",
      "position" : 8
    },
    {
      "token" : "関西",
      "start_offset" : 15,
      "end_offset" : 17,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "関西国際空港",
      "start_offset" : 15,
      "end_offset" : 21,
      "type" : "word",
      "position" : 9,
      "positionLength" : 3
    },
    {
      "token" : "国際",
      "start_offset" : 17,
      "end_offset" : 19,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "空港",
      "start_offset" : 19,
      "end_offset" : 21,
      "type" : "word",
      "position" : 11
    },
    {
      "token" : "アメリカ",
      "start_offset" : 23,
      "end_offset" : 27,
      "type" : "word",
      "position" : 13
    },
    {
      "token" : "テキサス",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "word",
      "position" : 14
    },
    {
      "token" : "州",
      "start_offset" : 32,
      "end_offset" : 33,
      "type" : "word",
      "position" : 15
    },
    {
      "token" : "旅行",
      "start_offset" : 34,
      "end_offset" : 36,
      "type" : "word",
      "position" : 17
    },
    {
      "token" : "行く",
      "start_offset" : 37,
      "end_offset" : 39,
      "type" : "word",
      "position" : 19
    },
    {
      "token" : "常々",
      "start_offset" : 42,
      "end_offset" : 44,
      "type" : "word",
      "position" : 21
    },
    {
      "token" : "夢見る",
      "start_offset" : 44,
      "end_offset" : 46,
      "type" : "word",
      "position" : 22
    },
    {
      "token" : "とても",
      "start_offset" : 52,
      "end_offset" : 55,
      "type" : "word",
      "position" : 27
    },
    {
      "token" : "楽しみ",
      "start_offset" : 55,
      "end_offset" : 58,
      "type" : "word",
      "position" : 28
    },
    {
      "token" : "コンピュータ",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "word",
      "position" : 31
    },
    {
      "token" : "予約",
      "start_offset" : 70,
      "end_offset" : 72,
      "type" : "word",
      "position" : 33
    },
    {
      "token" : "料金",
      "start_offset" : 77,
      "end_offset" : 79,
      "type" : "word",
      "position" : 37
    },
    {
      "token" : "九",
      "start_offset" : 80,
      "end_offset" : 81,
      "type" : "word",
      "position" : 39
    },
    {
      "token" : "万",
      "start_offset" : 81,
      "end_offset" : 82,
      "type" : "word",
      "position" : 40
    },
    {
      "token" : "円",
      "start_offset" : 82,
      "end_offset" : 83,
      "type" : "word",
      "position" : 41
    }
  ]
}
```

##### kuromoji カスタム

```
$ curl -XPUT 'http://localhost:9200/my_index' -H 'Content-Type: application/json' -d '
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_japanese": {
          "type": "custom",
          "tokenizer": "kuromoji_tokenizer",
          "char_filter": ["kuromoji_iteration_mark"],
          "filter": [
            "kuromoji_baseform",
            "kuromoji_part_of_speech",
            "ja_stop",
            "kuromoji_number",
            "kuromoji_stemmer"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "user_name": {
        "type": "text",
        "analyzer": "my_japanese"
      }
    }
  }
}
'
```

```
$ curl -XGET 'localhost:9200/my_index/_analyze?pretty' -H 'Content-Type: application/json' -d '{"analyzer": "my_japanese", "text": "私は日々貯金したお金で、近々、関西国際空港からアメリカ　テキサス州へ旅行に行きます。常々夢見ていたので、とても楽しみで、このコンピューターで予約しました。料金は九万円ほどでした"}'
```

```
{
  "tokens" : [
    {
      "token" : "私",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "日日",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "word",
      "position" : 2
    },
    {
      "token" : "貯金",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "word",
      "position" : 3
    },
    {
      "token" : "お金",
      "start_offset" : 8,
      "end_offset" : 10,
      "type" : "word",
      "position" : 6
    },
    {
      "token" : "近近",
      "start_offset" : 12,
      "end_offset" : 14,
      "type" : "word",
      "position" : 8
    },
    {
      "token" : "関西",
      "start_offset" : 15,
      "end_offset" : 17,
      "type" : "word",
      "position" : 9
    },
    {
      "token" : "関西国際空港",
      "start_offset" : 15,
      "end_offset" : 21,
      "type" : "word",
      "position" : 9,
      "positionLength" : 3
    },
    {
      "token" : "国際",
      "start_offset" : 17,
      "end_offset" : 19,
      "type" : "word",
      "position" : 10
    },
    {
      "token" : "空港",
      "start_offset" : 19,
      "end_offset" : 21,
      "type" : "word",
      "position" : 11
    },
    {
      "token" : "アメリカ",
      "start_offset" : 23,
      "end_offset" : 27,
      "type" : "word",
      "position" : 13
    },
    {
      "token" : "テキサス",
      "start_offset" : 28,
      "end_offset" : 32,
      "type" : "word",
      "position" : 14
    },
    {
      "token" : "州",
      "start_offset" : 32,
      "end_offset" : 33,
      "type" : "word",
      "position" : 15
    },
    {
      "token" : "旅行",
      "start_offset" : 34,
      "end_offset" : 36,
      "type" : "word",
      "position" : 17
    },
    {
      "token" : "行く",
      "start_offset" : 37,
      "end_offset" : 39,
      "type" : "word",
      "position" : 19
    },
    {
      "token" : "常常",
      "start_offset" : 42,
      "end_offset" : 44,
      "type" : "word",
      "position" : 21
    },
    {
      "token" : "夢見る",
      "start_offset" : 44,
      "end_offset" : 46,
      "type" : "word",
      "position" : 22
    },
    {
      "token" : "とても",
      "start_offset" : 52,
      "end_offset" : 55,
      "type" : "word",
      "position" : 27
    },
    {
      "token" : "楽しみ",
      "start_offset" : 55,
      "end_offset" : 58,
      "type" : "word",
      "position" : 28
    },
    {
      "token" : "コンピュータ",
      "start_offset" : 62,
      "end_offset" : 69,
      "type" : "word",
      "position" : 31
    },
    {
      "token" : "予約",
      "start_offset" : 70,
      "end_offset" : 72,
      "type" : "word",
      "position" : 33
    },
    {
      "token" : "料金",
      "start_offset" : 77,
      "end_offset" : 79,
      "type" : "word",
      "position" : 37
    },
    {
      "token" : "90000",
      "start_offset" : 80,
      "end_offset" : 82,
      "type" : "word",
      "position" : 38
    },
    {
      "token" : "円",
      "start_offset" : 82,
      "end_offset" : 83,
      "type" : "word",
      "position" : 39
    }
  ]
}
```

##### その他

カタカナ読みに変換。
```
filter: [ "kuromoji_readingform" ]
```

#### kuromoji analyzerを用いたインデックスの例

```
$ curl -XPUT 'http://localhost:9200/my_index' -H 'Content-Type: application/json' -d '
{
  "mappings": {
    "properties": {
      "user_name": {
        "type": "text",
        "analyzer": "kuromoji"
      },
      "date": {
        "type": "date"
      },
      "message": {
        "type": "text",
        "analyzer": "kuromoji"
      }
    }
  }
}
'
```

テストデータ

```
$ curl  -XPOST 'http://localhost:9200/my_index/_doc/' -H 'Content-Type: application/json' -d '
{
  "user_name": "山本　太郎",
  "date": "2017-10-15T15:09:45",
  "message": "秋は京都で紅葉狩りをします。"
}
'

$ curl  -XPOST 'http://localhost:9200/my_index/_doc/' -H 'Content-Type: application/json' -d '
{
  "user_name": "佐藤　洋子",
  "date": "2017-10-15T15:09:45",
  "message": "冬は北海道でスキーをします。"
}
'
```

##### analyzerの確認

```
$ curl -XPOST 'http://localhost:9200/my_index/_analyze' -H 'Content-Type: application/json' -d '
{
  "analyzer": "kuromoji",
  "text": "秋は京都で紅葉狩りをします。"
}
'
```
```
{"tokens":[{"token":"秋","start_offset":0,"end_offset":1,"type":"word","position":0},{"token":"京都","start_offset":2,"end_offset":4,"type":"word","position":2},{"token":"紅葉狩り","start_offset":5,"end_offset":9,"type":"word","position":4}]}
```

##### 検索クエリの例

```
$ curl -XGET 'http://localhost:9200/my_index/_doc/_search' -H 'Content-Type: application/json' -d '
{
  "query": {
    "match": {
      "message": "京都の秋のおすすめ"
    }
  }
}
'
```

```
{"took":186,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":{"value":1,"relation":"eq"},"max_score":1.3862942,"hits":[{"_index":"my_index","_type":"_doc","_id":"Uic8CnMBjvhLNEhN50wI","_score":1.3862942,"_source":
{
  "user_name": "山本　太郎",
  "date": "2017-10-15T15:09:45",
  "message": "秋は京都で紅葉狩りをします。"
}
}]}}
```

インデックスは"紅葉狩り"で登録されているので、"紅葉"では検索できない
```
$ curl -XGET 'http://localhost:9200/my_index/_doc/_search' -H 'Content-Type: application/json' -d '
{
  "query": {
    "match": {
      "message": "紅葉の写真"
    }
  }
}
'
```

#### Aggregation

RDBでいうところのGroup byと集約関数

office:Otemachiを含むドキュメントのsalaryフィールドの平均値をavg_salaryとして出力する。
```
$ curl -XGET 'http://localhost:9200/employee/_search' -H 'Content-Type: application/json' -d '
{
  "size": 0,
  "query": {
    "match": {
      "office": "Otemachi"
    }
  },
  "aggs": {
    "avg_salary": {
      "avg": {
        "field": "salary"
      }
    }
  }
}
'
```

avg, sum, min, max, cardinality(count(distinct field)), stats(count, min, max, avg, sumの統計値がまとめて返される)

##### bucket

bucketはフィールドの値ごとの集約を求めることができる。つまりgroup byのような動作をする。

genreフィールドでグルーピング。件数の多い上位3件を返す。
```
{
  "aggs": {
    "genre_buckets": {
      "terms": {
        "field": "genre.keyword",
        "size": 3
      }
    }
  }
}
```

##### range, histgram

bucketの集約はrangeごと、増分(interval)ごとに集約することができる。