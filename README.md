# elastic-search-study

## installation
- cluster에서 여러 node로 운영을 해보기 위해서 docker로 띄움 - [link](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

## getting starged with elastic search - [link](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)
- rest API로 모든 커뮤니케이션이 가능한것 같다 (인덱스 생성부터 헬스체크, CRUD, query, script를 통한 manipulate 등 )

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
    - bulk 인서트나 스크립트를 이용한 수정도 가능한데 POST를 이용해서 가능하다 `curl -XPOST 'localhost:9200/customer/_update/1' ...
