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

