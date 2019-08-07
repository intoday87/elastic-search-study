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
  
### [Query a document](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-query-document.html#getting-started-query-document)
  - Search API로 넘어가면서 목록을 가져오는건 어떻게 했지? 라고 갑자기 헷갈릴수 있는데, 이 부분에서는 `curl -X GET 'localhost:9200/bank/_doc/1?pretty'` 이런식으로 단건만 가져올 수 있고 목록은 search API를 이용
  
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
## Elasticsearch in action
- site에 document로 학습하는데는 사용성에 측면에서는 좀 더 빠른길이지만 이해를 필요로 하는 부분에서는 빠른 길을 제공하는것 같지 않아서 책을 통해 학습하는 내용을 정리

### Elasticsearch에 대해
- 루씬 기반 검색엔진 라이브러리 기반으로 만들어짐
- 루씬 기반이기 때문에 역색인을 제공할 수 있다 -> 검색된 해당 도큐먼트가 어디에 속해 있는지 알 수 있다 -> 속한 도큐먼트의 개수를 알 수 있다
- 점수(score)를 계산하는 알고리즘을 사용할 수 있다
  - ex_) `투표`라는 검색결과에 `투표`라는 키워드가 많이 포함되어 있다면 점수가 높다고 계산할 수 있다
- 트랜잭션을 지원하지 않는다. 동시성을 관리하기 위해 버전을 사용한다
- elasticsearch는 로깅 시스템에 이미 대중적이다
- 기존 다른 스토리지와 연동할 수 있는 플러그인이 대부분 제공된다 - 데이터 삽입시는 트랜잭션이 지원되는 관계형 DB에 색인은 동기화 플러그인 또는 작성된 서비스로부터 elasticsearch에 색인되는 형태
- 데이터를 레코드나 행으로 저장하는 관계형 데이터베이스와 다르게 엘라스틱서치는 데이터를 문서 단위로 저장한다
  - 데이터가 테이블이나 엔티티 단위로 정규화되어 있지 않고 하나의 문서가 저장되어 있다보니 조인 연산같은게 필요 없어서 검색이 빠르다
