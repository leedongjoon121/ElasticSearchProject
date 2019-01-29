# ElasticSearch 테스트

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

