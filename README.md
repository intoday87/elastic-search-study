# elastic-search-study

## installation
- cluster에서 여러 node로 운영을 해보기 위해서 docker로 띄움 - [link](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

## getting starged with elastic search - [link](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)
- rest API로 모든 커뮤니케이션이 가능한것 같다 (인덱스 생성부터 헬스체크, CRUD, query, script를 통한 manipulate 등 )

- `curl -XGET localhost:9200/_cat/health?v`로 보면 status 필드에 green, yello, red 상태가 있는데 알아두어야 할 것 같다
  - green: 말 그대로 이상 없음
  - yello: 데이터가 모두 서비스가 가능하나 몇 replica들이 할당이 안되서 완전한 기능(고가용성을 말하는것 같음)을 제공할 수 없을때
  - red: 어떤 일부 데이터가 서비스가 안될수 있다
