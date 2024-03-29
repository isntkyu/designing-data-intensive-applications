# 다중 리더 복제

쓰기를 허용하는 노드를 하나 이상 두는것.
각 리더는 동시에 다른 리더의 팔로워역할도 함.

## 사례

데이터 센터를 여러개 두는 방식.
가장 큰 단점은 쓰기 충돌 해소이다.
오토인크리먼트, 트리거, 무결성제약은 문제의 소지가 있게된다.

**모바일 캘린더 예시**

오프라인 상태에서도 일정이 저장되어야하고, 온라인이 되었을 때 서버와 동기화해야한다.
때문에 쓰기요청을 받을 로컬DB가 존재한다.

이런 상황에서 각 디바이스가 데이터센터 역할을 하는 다중리더 아키텍처로 볼 수 있는데,
복제 지연이 클 수 있다.

이런 문제를 해결하기 위한 도구로 **CouchDB**가 있다.

**구글 독스**

협업툴, 동시 편집이 가능한 구글 독스, 아주 작은 단위 변경의 잠긍을 사용
A가 제목을 1로 바꾸고 B가 제목을 2로 바꾼다.
각 로컬 데이터베이스 리더에서는 쓰기가 성공했지만 변경을 비동기로 복제할 때 충돌이 생긴다. (단일 리더에서는 일어나지 않는 문제)

동기식 복제로 해결할 수 있지만, 이러면 다중 리더복제의 독립적 쓰기의 장점을 크게 포기하는 것.

---

## 일관된 상태로 수렴

- 쓰기 요청에 고유id나 타임스탬프를 부여하고 최종쓰기를 채택한다. 하지만 중간 데이터 유실이 발생할 수 있다. (대중적이긴함)
- 어떻게든 사전순 정렬한 후 연결해서 병합한다.
- 충돌과정에 생기는 모든 정보를 기록하고, 나중에 충돌 해소하는 애플리케이션 코드를 작성한다.

---

## 충돌 해소 로직

- 쓰기 수행중

복제된 변경사항 로그에서 충돌을 감지하자마자 충돌 핸들러를 호출한다.
사용자에겐 표시하지 않고 백그라운드에서 빠르게 실행.

- 읽기 수행중

충돌 감지하면 모든 충돌 쓰기를 저장한다. 다음 번 데이터를 읽을 때 여러 버전의 데이터가 애플리케이션에 반환된다.
애플리케이션은 충돌을 자동으로 해소하거나 사용자에게 보여준다. 결과는 디비에 기록한다(CouchDB방식)

충돌 해소는 트랜잭션보다는 개별 로우나 문서수준에서해결된다.

---

## 토폴로지

쓰기를 한 노드에서 다른 노드로 저달하는 통신 경로

일반적으로는 전체연결을 사용, mysql은 원형 연결 토폴로지
