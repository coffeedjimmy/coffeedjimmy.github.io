---
title: Elasticsearch 제대로 알기 01
date: 2019.12.09 12:00:00
categories:
- elasticsearch
tags:
- Elasticsearch
- Kibana
- Logstash
---

.


## 1. 검색 시스템이란?

- Query → docs
- 검색 엔진: 광활한 웹에서 정보를 수집해 검색 결과를 제공하느 ㄴ프로그램.
- 검색 서비스 > 검색 시스템 > 검색 엔진

### 1.1 검색 시스템의 구성 요소

- 정보를 수집하는 수집기
    - 크롤러, 스파이더, 웜, 웹 로봇 등올 불림
- 데이터를 저장하는 스토리지
    - 물리적인 저장소
- 데이터를 검색에 적절한 형태로 변환하는 색인기
    - 수집된 데이터를 검색 가능한 구조로 가공하고 저장
    - 다양한 형태소 분석기를 조합하여 정보에서 의미있는 용어를 추출하고 역색인 구조로 데이터를 저장
- 색인된 데이터에서 일치하는 문서를 찾는 검색기
    - 유사도 기반의 검색 순위 알고리즘으로 판단.

### 1.2. RDB와의 차이점

- 검색엔진은 데이터베이스에서는 불가능한 비정형 데이터를 색인하고 검색 가능
- 인덱스 → 데이터베이스
- 샤드 → 파티션
- 타입 → 테이블
- 문서 → 행
- 필드 → 컬럼
- 매핑 → 스키마
- Query DSL → SQL
- 6.1 버전부터 하나의 인덱스에 하나의 타입만을 구성하도록 강제.

### 1.3. RESTful API

```bash
curl -X(method) http://host:port/index/type/id -d '{json data}'
```

- -X : 요청 메서드 지정 (default: GET)
- -d : 요청 본문을 지정

## 2.  검색 시스템과 엘라스틱 서치

### 2.1 엘라스틱 서치가 강력한 이유

- 오픈소스 검색엔진
- 전문 검색 (Full Text)
    - 내용 전체를 색인해서 특정 단어가 포함된 문서를 검색하는 것.
- 통계 분석
- 스키마리스
- RESTful API
- 멀티테넌시
    - 서로 상이한 인덱스라도 검색할 필드명만 같으면 여러 개의 인덱스를 한번에 조회 가능
- Document-oriented
    - 계층 구조로 구성된 문서도 쉽게 쿼리가능
- 역색인 (Inverted Index)
    - MongoDB, Cassandra 등과 같은 일반적인 NoSQL은 역색인을 지원하지 않음
- 확장성과 가용성

### 2.2. 엘라스틱 서치의 약점

- 실시간이 아니다
    - 색인에 걸리는 시간이 있어서 준실시간이라고 봐야함
- 트랜잭션과 롤백을 지원하지 않음
    - 데이터 손실 위험이 있음 → 대신 레플리카로 커버
- 데이터의 업데이트를 제공하지 않음
    - 기존 문서를 삭제하고 변경된 내용으로 새 문서를 생성하는 방식
        - 상대적으로 큰 비용 소요 → 불변성이라는 장점이 있기도 함.

## 3. 엘라스틱 서치 살펴보기

### 3.1 엘라스틱 서치 구성 개념

- 인덱스
    - 데이터 저장 공간
    - 1 index = 1 type (>6.0.0)
    - 인덱스 생성시 기본적으로 5개의 primary 샤드와 1개의 레플리카 샤드를 구성
    - 인덱스 이름은 소문자
- 샤드
    - 색인된 문서는 하나의 인덱스에 담김
    - 인덱스 내부에 색인된 데이터는 물리적인 공간에 여러개의 파티션으로 나뉘어 구성 → 이를 샤드라고 부름
- 타입
    - 6.0.0 이하에서는 하나의 인덱스에 다수의 타입을 지정해서 분리 관리가 가능했는데, 현재 최신버전에서는 권장하지 않고 막는 기조
- 문서
    - 엘라스틱서치에서 데이터가 젖아되는 최소 단위
- 필드
    - 문서를 구성하기 위한 속성
    - 하나의 필드는 목적에 따라 다수의 데이터 타입을 가질 수 있음
- 매핑
    - 문서의 필드와 필드의 속성을 정의하고 그에 따른 색인 방법을 정의하는 프로세스



### 3.2 노드의 종류

- 클러스터는 물리적인 노드 인스턴스들의 모임
    - 모든 노드의 검색과 색인 작업을 관장하는 논리적인 개념
- 노드 유형
    - 마스터 노드
        - 클러스터 관리
        - 노드 추가/제거 같은 클러스터의 전반적인 관리 담당
        - 속도가 빠르고 지연이 없는 장비로 사용하는 것이 좋음
```bash
node.master: true
node.data: false
node.ingest: false
search.remote.connect: false
```
    - 데이터 노드
        - 데이터 저장
        - 검색과 통계같은 데이터 작업 수행
        - 기본적으로 마스터 노드와 분리해서 세팅하는 것을 권장하지만, 색인할 문서 수가 적다면 함께 구성해도 상관 없음
```bash
node.master: false
node.data: true
node.ingest: false
search.remote.connect: false
```

    - Coordinating Node
        - 사용자의 요청만 받아서 라운드로빈 방식으로 처리
        - 클러스터 관련 요청은 마스터에 데이터 관련 요청은 데이터 노드에 전달
```bash
node.master: false
node.data: false
node.ingest: false
search.remote.connect: false
```

    - Ingest Node
        - 문서의 전처리 작업을 담당
        - 인덱스 생성 전 문서의 형식을 다양하게 변경 가능
```bash
node.master: false
node.data: false
node.ingest: true
search.remote.connect: false
```

### 3.3 주요 API

- HTTP 통신을 위해 9200포트 사용
- 문서를 색인하기 위해 기본적으로 인덱스라는 그릇 생성 필요
- API 종류
    - Indices API
    - DocumentAPI
    - Search API
    - Aggregation API
- 스키마리스는 가급적이면 사용하지 않는 것이 좋음
    - 특수한 상황에서 제한적으로 사용
```bash
# elasticsearch.yml
action.auto_create_index: false  # 자동으로 인덱스 생성되는 것을 막음
index.mapper.dynamic: false  # 특정 컬럼의 자동 매핑 생성을 비활성화
```

### 3.4. Indices API

- 인덱스를 추가하거나 삭제 가능
- 인덱스 생성 시 스키마리스를 권장하지 않으므로 매핑을 지정해줌.
    - 한번 생성된 매핑 정보는 변경할 수 없음
    - e.g. keyword 타입: 단순 문자열 저장, text 타입: 형태소 분석
    ```bash
    PUT /movie  # 인덱스 생성
    ...  # 매핑 정보


    DELETE /movie  # 인덱스 삭제
    ```

- 인덱스는 한 번 삭제하면 복구 불가능하므로 신중하게 진행 필요

### 3.5. Document API

- 문서 조회, 수정, 삭제
- Search API도 있지만, 색인된 문석의 ID를 기준으로 한 건의 문서를 다룬다는 점에서 차이.
- 종류
    - Index API: 한 건 문서 색인
    - Get API : 한 건 문서 조회
    - Delete API : 한 건 문서 삭제
    - Update API : 한 건 문서 업데이트
    - Multi Get API : 다수의 문서 조회
    - Bulk API : 대량의 문서 색인
    - Delete By Query API : 다수의 문서 삭제
    - Update By Query API : 다수의 문서 업데이트
    - Reindex API : 인덱스의 문서를 다시 색인

```bash
    POST /movie/_doc/1  # 문서 생성
    ...

    GET /movie/_doc/1  # 문서 조회

    DELETE /movie/_doc/1  # 문서 삭제

    POST /movie/_doc  # 문서 아이디 자동생성
    ...

```

- 문서 아이디를 자동 생성 시킬 경우 UUID를 통해 무작위 생성
- 엘라스틱 서치와 RDB를 연계해서 사용하는 경우 RDB의 PK와 엘라스틱 서치의 문서 아이디 매핑정보를 가지고 있기 쉽지 않으므로 PK를 엘라스틱서치의 문서 id로 사용하는 것이 편리.

### 3.6. Search API

- 검색 API는 두 가지 방식으로 사용가능
    - HTTP URI형태의 파라미터를 URI에 추가해 검색하는 방법
    - RESTful API 방식인 QueryDSL을 사용해 요청 본문(Request Body)에 질의 내용을 추가해 검색
- 두 방식을 섞어서 사용도 가능

**URI 방식의 검색 질의**

    ```bash
    GET /movie/_doc/ABDAHKASDHBKSAD?pretty=true

    POST /movie/_search?p=장편

    POST /movie/_search?p=typeNm:장편
    ```

**Request Body 방식의 검색 질의**

- URI 검색 질의는 여러 필드를 각기 다른 검색어로 질의하는 것이 어려움

```bash
    POST /index/_search
    {
    JSON query
    }
```

### 3.7 집계 API

- 과거 엘라스틱서치 버전에서는 루씬이 제공하는 Facets 기능을 활용
    - 디스크 기반으로 동작하고 분산환경에 최적화 되지 않았기 때문에 대용량 데이터의 통계 작업에 비적합
- 집계 API를 5.0 이후에 단독으로 내놓고 메모리 기반으로 동작하도록 함

```bash
    POST /movie/_search?size=0
    {
    	"aggs":{
    	...
    	}
    }
```

데이터 집계 타입은 4가지가 존재

- 버킷 집계
    - 가장 많이 사용됨. 문서의 필드기준으로 버킷을 집계
- metric agg
    - 문서에서 추출된 값을 가지고 Sum, Max 등을 계산
- Matrix Agg
    - 행렬의 값을 합하거나 곱함
- Pipeline Agg
    - 버킷에서 도출된 결과를 다른 필드 값으로 재분류 → Facets보다 강력한 이유

.

2부에서 계속.
