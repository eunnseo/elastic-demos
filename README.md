# elastic-demos
## elastic stack을 이용한 서울시 지하철 대시보드 만들기
https://github.com/eskrug/elastic-demos

## 1. 공공데이터로부터 추출, 색인, 매핑 및 템플릿 설정

### 공공 데이터
[서울 열린 데이터 광장](https://data.seoul.go.kr/) 에서 **서울교통공사 연도별 일별 시간대별 역별 승하차 인원** 데이터 사용

- 데이터 색인 완료 후 아래 명령으로 확인
```json
GET seoul-metro-logs-2018/_count
```
![count](https://user-images.githubusercontent.com/55284181/123420206-39017500-d5f6-11eb-9194-e69ddfc31657.png)

- 데이터 내용 확인
```json
GET seoul-metro-logs-2018/_search
```
![data](https://user-images.githubusercontent.com/55284181/123421554-f2ad1580-d5f7-11eb-8865-3e86320da54c.png)

- seoul-metro-logs* 템플릿 생성
```json
PUT _template/seoul-metro
{
  "order": 5,
  "index_patterns": [
    "seoul-metro-logs*"
  ],
  "settings": {
    "number_of_shards": 2
  },
  "mappings": {
    "properties": {
      "@timestamp": {
        "type": "date"
      },
      "code": {
        "type": "keyword"
      },
      "line_num": {
        "type": "keyword"
      },
      "line_num_en": {
        "type": "keyword"
      },
      "location": {
        "type": "geo_point"
      },
      "people": {
        "properties": {
          "in": {
            "type": "integer"
          },
          "out": {
            "type": "integer"
          },
          "total": {
            "type": "integer"
          }
        }
      },
      "station": {
        "properties": {
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

## 2. Kibana 대시보드
#### Run
```sh
$ docker-compose up
```
#### Stop
```sh
$ docker-compose down
```
#### Port Forwarding 
|Host|Container|Service|
|:---:|:---:|:---:|
|5601|5601|kibana|

### 호선 승하차 인원 역명 리스트
![kibana_list](https://user-images.githubusercontent.com/55284181/123420874-12900980-d5f7-11eb-8618-e4b3ec3d811a.png)

### 2018-01-01 ~ 2019-01-01 서울 지하철 대시보드
![final_year](https://user-images.githubusercontent.com/55284181/123422579-61d73980-d5f9-11eb-8483-379254b3734a.png)

### 2018-05-08 ~ 2018-05-24 강남, 역삼, 선릉, 삼성역 대시보드
![final_search](https://user-images.githubusercontent.com/55284181/123422585-63a0fd00-d5f9-11eb-8cd9-21df69584e23.png)
- 오전과 오후 승하차 인원의 뚜렷한 차이를 확인할 수 있다.
- 주말 대비 주중 승하차 인원이 더 많은 것을 확인할 수 있다.
