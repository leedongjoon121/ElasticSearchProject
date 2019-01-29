# ElasticSearch 테스트


<hr/>

# 실습

## 데이터 생성
### id를 넣지 않는 경우에는 PUT으로는 불가능하다, 반드시 POST로 해야 한다.

```
PUT http://localhost:9200/customer/news
- 불가능
```

```
POST http://localhost:9200/customer/news
- 가능
- id를 자동으로 생성한다. (고유 값으로)
```

## 데이터 수정
### POST, PUT으로 다시 입력시 수정이 된다. 그러나 version은 순차적으로 증가한다.
- POST로 수정하면 데이터를 하나씩 세세하게  수정 가능하다.
```
{
    "doc" : {
        "author" : "이것만 수정가능"
    }
}
```

- PUT으로 수정할 경우, 데이터를 하나 수정하면 수정되지만, 다른 필드가 사라지게 된다. 

## 데이터 삭제
### 키값을 통해 삭제 한다.
```
   XDELETE PUT http://localhost:9200/customer/news/1
   
   customer 인덱스의 new 타입의 id 1 삭제 
```

### 모든 인덱스를 삭제할 경우  
- '*' 또는 'all'을 활용한다.
```
    $ curl -XDELETE http://localhost:9200/*
    $ curl -XDELETE http://localhost:9200/_all
```

- 모든 인덱스 삭제가 위험해서, 사전에 모든 삭제를 방지하는 옵션 : .yml에 추가한다.
```
  action.destructive_requires_name: true
```

## 벌크 인서트
```
  POST http://localhost:9200/shakespeare/act/_bulk
  
{"index":{"_index":"shakespeare","_type":"act","_id":0}}
{"line_id":1,"play_name":"Henry IV","speech_number":"","line_number":"","speaker":"","text_entry":"ACT I"}

{"index":{"_index":"shakespeare","_type":"scene","_id":1}}
{"line_id":2,"play_name":"Henry IV","speech_number":"","line_number":"","speaker":"","text_entry":"SCENE I. London. The palace."}

{"index":{"_index":"shakespeare","_type":"line","_id":2}}
{"line_id":3,"play_name":"Henry IV","speech_number":"","line_number":"","speaker":"","text_entry":"Enter KING HENRY, LORD JOHN OF LANCASTER, the EARL of WESTMORELAND, SIR WALTER BLUNT, and others"}

{"index":{"_index":"shakespeare","_type":"line","_id":3}}
{"line_id":4,"play_name":"Henry IV","speech_number":1,"line_number":"1.1.1","speaker":"KING HENRY IV","text_entry":"So shaken as we are, so wan with care,"}

{"index":{"_index":"shakespeare","_type":"line","_id":4}}
{"line_id":5,"play_name":"Henry IV","speech_number":1,"line_number":"1.1.2","speaker":"KING HENRY IV","text_entry":"Find we a time for frighted peace to pant,"}

{"index":{"_index":"shakespeare","_type":"line","_id":5}}
{"line_id":6,"play_name":"Henry IV","speech_number":1,"line_number":"1.1.3","speaker":"KING HENRY IV","text_entry":"And breathe short-winded accents of new broils"}

{"index":{"_index":"shakespeare","_type":"line","_id":6}}
{"line_id":7,"play_name":"Henry IV","speech_number":1,"line_number":"1.1.4","speaker":"KING HENRY IV","text_entry":"To be commenced in strands afar remote."}
  
```

## 벌크 DELETE
```
    { "delete":{"_index":"shakespeare","_type":"act","_id":0} }
    { "delete":{"_index":"shakespeare","_type":"act","_id":1} }
    { "delete":{"_index":"shakespeare","_type":"act","_id":2} }
```

## 매핑
1.ElasticSearch 별도 스키마 설정이 없이도 자동으로 자료구조를 명시적으로 제어하고 싶을 때 매핑을 이용한다.
2.데이터가 입력되면 자동으로 매핑되어 매핑이 저장된다.
3.한 인덱스에 매핑을 여러개 만들수 있으나, 필드명은 중복되면 안된다.
4.매핑은 삭제할 수 없다. 

```
  PUT http://localhost:9200/news
  
  {
      "mappings" : {
          "news" : {
          
          }
      }
  }
```

데이터를 입력하면 자동으로 매핑을 하기때문에, 위 처럼 매핑을 안해도 아래처럼 조회가 된다.
```
   GET http://localhost:9200/customer/_mapping/news
```

## 분석기 추가 매핑
```
{
   "mappings" : {
     "article" : {
        "properties" : {
           "id" : {"type":"long", "store":"yes", "precision_step":"8"}
           "title" : {"type":"string", "store":"yes", "index":"analyzed"},
           "link" : {"type":"string"},
           "content" : {"type":"string","store":"no","index":"analyzed"},
           "reg_date" : {"type" : "date"}
        }
     }
   }
}
```

## 커스텀 분석기
1. 셋팅을 하기전에 인덱스를 close 시켜야 한다.
```
    http://localhost:9200/news/_close
```
2. 작업 완료 후 다시 open한다.
```
    http://localhost:9200/news/_open
```

```
    PUT  http://localhost:9200/news/_settings
    {
       "settings" : {
          "analysis" : {
            "analyzer" : {
              "my_english_analyzer": {
                  "type" : "standard",
                  "max_token_length" : 5,
                  "stopwords" : "_english_"
                  
              }
            }
          }
       }
    }
    
    my_english_analyzer 라는 이름으로  생성하고, english라는 단어 기준 
```

## 좀 더 복잡한 분석기 셋팅
```
    PUT  http://localhost:9200/news/_settings
    {
       "settings" : {
          "analysis" : {
            "analyzer" : {
              "my_english_analyzer": {
                  "type" : "custom",
                  "tokenizer" : "standard",
                  "char_filter":[
                    "html_strip"
                  ],
                  "filter" : [
                    "lowercase",
                    "asciifolding"
                  ]
              }
            }
          }
       }
    }
    
```

## 셋팅 내역 확인
    GET  http://localhost:9200/news/_settings
    
## 분석기 테스트
```
POST  http://localhost:9200/news/_analyze
{
   "analyzer" : "my_custom_analyzer",
   "text" : "Hi my name is Joon. "
}
```

```
POST  http://localhost:9200/news/_analyze
{
   "tokenizer" : "keyword",
   "filter" : ["lowercase"],
   "text" : "Hi my name is Joon. this is a test "
}
```

## 좌표검색

```
POST  http://localhost:9200/news/_mapping/store
{
   "properties" : {
      "store_name" : {
         "type" : "string"
      },
      "location" : {
         "type" : "geo_point"
      }
   }
}
```


데이터 입력
```
  "store_name" : "스타벅스 영천점"
  "location" : "37.52334,126.22523" 
```

좌표검색
```
{
   "query" : {
      "filtered" : {
         "query" : {
            "match_all" : {}
         },
         "filter" : {
            "geo_distance" : {
               "distance" : "200km",
               "location" : {
                   "lat" : 37.2234,
                   "lon" : 126.33242
               }               
            }
         }
      }
   }
}  
```
