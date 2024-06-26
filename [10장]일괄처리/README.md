파생 데이터: 레코드 시스템(진실의 근원)으로부터 파생된 데이터(캐시, 뷰, 색인 등)

---

# 일괄 처리

- 서비스(온라인 시스템): 응답시간이 성능 지표, 가용성이 중요
- 일괄처리 시스템(오프라인 시스템): 처리량이 성능지표, 배치
- 스트림 처리 시스템(준실시간 시스템): EDA, kafka 등. 일괄처리를 기반함

---

### 맵리듀스

HDFS 같은 분산 파일 시스템 위에서 대용량 데이터셋을 처리하는 코드를 작성하는 프레임워크

매퍼: 모든 입력 레코드마다 한 번씩만 호출. 입력으로부터 키와 값을 추출. 각 레코드를 독립적으로 처리

리듀서: 매퍼가 생산한 키-값 쌍을 받아 같은 키를 가진 레코드를 모으고 해당 값의 집합을 반복해 리듀서 함수를 호춣한다. 출력 레코드를 생산한다.

파티셔닝을 기반으로 한 병렬처리

---

### 리듀스 사이드 조인과 그릅화

조인

대량의 레코드를 대상으로 전체 테이블 스캔(FUll SCAN)을 사용하는건 합리적이다. 여러 장비에 걸쳐 병렬 처리가 가능한 경우 특히 그렇다.  
일괄처리 맥락에서는 조인은 데이터셋 내 모든 연관관계를 다룬다는 뜻이라서다. 특정 사용자의 데이터만 찾는다면 인덱스가 효율적.

조인 대상이 될 데이터베이스의 사본을 추출해 가져와 효율적인 처리를 한다.

#### 정렬 병합 조인

보조정렬(재배열) 후에 병합

#### 그룹화

매퍼가 만든 키를 기준으로 그룹화 한다.(세션화)

#### 쏠림 다루기

핫키: 같은 키를 가지는 레코드가 너무 많음(ex 팔로워가 많은 사람)

쏠린 조인 해결을 위해 핫 키를 여러 리듀서에 퍼트려서 처리하게 한다.

---

### 맵 사이드 조인

리듀서가 아닌 매퍼 단에서늬 조인 수행  
입력 데이터에 대한 가정이 가능할 때.

#### 브로드캐스트 해시 조인

매퍼가 시작할 때 분산 파일 시스템에서 DB를 읽어서 인메모리 해시 테이블에 넣는다.  
매퍼 태스크를 여러개 사용할 수도 있다.  
모든 매퍼는 작은 입력 전체를 메모리에 적재한다.  
작은 입력을 매퍼가 읽어서 파티션 전체로 브로드캐스트

#### 파티션 해시 조인

맵 사이드 조인의 입력을 파티셔닝한다.  
해시 조인 접근법을 각 파티션에서 독립 수행

#### 맵 사이드 병합 조인

입력 데이터셋이 파티셔닝 뿐 아니라 같은 키 기준으로 정렬
