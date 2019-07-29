# Elastic-search-study (ES)

## [Installation](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)
- cluster에서 여러 node로 운영을 해보기 위해서 docker로 띄움

## [Getting starged with elastic search](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)
- rest API로 모든 커뮤니케이션이 가능한것 같다 (인덱스 생성부터 헬스체크, CRUD, query, script를 통한 manipulate 등)

- `curl -XGET localhost:9200/_cat/health?v`로 보면 status 필드에 green, yello, red 상태가 있는데 알아두어야 할 것 같다
  - green: 말 그대로 이상 없음
  - yello: 데이터가 모두 서비스가 가능하나 몇 replica들이 할당이 안되서 완전한 기능(고가용성을 말하는것 같음)을 제공할 수 없을때
  - red: 어떤 일부 데이터가 서비스가 안될수 있다
  - docker로 node를 하나만 띄웠는데, `active_shard_percent`가 50%로 표시된다
  - data가 없을때는 shard가 0이었는데, 데이터가 추가되면서 shard 개수가 늘어났다

- rest API 구성형식이 모든 API에 대해 일관성있어서 알고 있어야 한다
  - `<HTTP Verb> /<Index>/<Endpoint>/<ID>`
  - example
    - create index `curl -XPUT 'localhost:9200/customer'
    - create doc `curl -XPUT 'localhost:9200/customer/_doc/1?pretty' -H 'Content-Type:application/json;charset=utf8' -d '{ "name": "nick wilde" }'
    - delete doc 'curl -XDELETE 'localhost:9200/customer/_doc/1'
    - bulk 인서트나 스크립트를 이용한 수정도 가능한데 POST를 이용해서 가능하다 `curl -XPOST 'localhost:9200/customer/_update/1' ...` [link](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-update-documents.html)

### [Batch processing](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-batch-processing.html)
  - bulk insert를 이용
  
  - 순차 처리돠다가 실패한 건은 오류로 처리하고 다음것으로 넘어간다
  
### [Search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-search-API.html)
  - 같은 Endpoint에 query param 또는 GET에 본문(body)에 json을 실어서 동일하게 검색할 수 있다. 다만 파라미터 이름은 길이를 위해 차이가 있다. ex_) `q=*` => `{ query: { match_all: {} } } `
  - 다른 스토리지 플랫폼과 구별되는 가장 큰 차이점이 있다. server-side cursor를 유지하지 않는다는 것이다. 보통 스토리지 플랫폼에서 많은 양의 데이터를 조회하기 위해 조회시 cursor를 반환해서 다음 요청시 cursor가 가리키는 준비된 데이터의 다음을 읽게하지만 ES는 그렇지 않고 rest API의 특성인 stateless로 완전히 응답시 모든 상태를 끊는다

#### [Query language(Query DSL)](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-query-lang.html)
  - `mill` 과 `lane`이 둘 중에 하나라도 들어있는것을 찾는다
    ```zsh
    curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
    {
      "query": { "match": { "address": "mill lane" } }
    }
    '
    ```
  - `mill`과 `lane`이 둘 다 들어있는(AND)로 검색할려면 bool query로 요청해야 한다
    ```zsh
    curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
    {
      "query": {
        "bool": {
          "must": [
            { "match": { "address": "mill" } },
            { "match": { "address": "lane" } }
          ]
        }
      }
    }
    '
    ```
  
