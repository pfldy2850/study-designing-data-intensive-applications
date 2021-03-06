# 신뢰할 수 있고 확장 가능하며 유지보수하기 쉬운 애플리케이션

- 오늘날 많은 애플리케이션은 계산 중심(compute-intesive) < 데이터 중심(data-intensive)
- 데이터 중심 애플리케이션은 표준 구성 요소(standard building block)로 만듦
  - 데이터를 지정(데이터베이스)
  - 값비싼 수행 결과를 기억(캐시)
  - 데이터 검색 제공(검색 색인)
  - 비동기 처리를 위한 다른 프로세스로 메세지 보내기(스트림 처리)
  - 주기적으로 대량의 누적된 데이터를 분석(일괄 처리)
- 애플리케이션마다 요구사항이 다름 -> 도구와 접근 방식이 적합한지 고려해야 함

## 데이터 시스템에 대한 생각

- 새로운 도구들은 다양한 use case에 최적화됐음 -> 전통적인 분류X
  - 메시지 큐로 사용하는 데이터스토어인 redis
  - 데이터베이스처럼 지속성(durability)를 보장하는 메세지 큐인 Apache kafka
  - 즉, 분류 간 경계가 흐려지고 있음
- 단일 도구로는 더 이상 데이터 처리와 저장 모두를 만족시킬 수 없음
- 작업(work)은 단일 도구에서 효율적으로 수행할 수 있는 태스크(task)로 나눔
- 다양한 도구들은 애플리케이션 코드를 이용해 서로 연결

이 책에서는 세 가지 관심사에 중점을 둔다.

- 신뢰성(Reliablitiy)
  하드웨어나 소프트웨어의 결함, human error 같은 역경에 직면하더라도 시스템은 지속적으로 올바르게 동작해야 함

- 확장성(Scalability)
  시스템의 데이터 양, 트래픽 양, 복잡도가 증가하면서 이를 처리할 수 있는 적절한 방법이 있어야 함

- 유지보수성(Maintainability)
  모든 사용자가 시스템 상에서 생산적으로 작업할 수 있게 해야 함

## 신뢰성

소프트웨어의 경우 일반적인 기대치는 다음과 같음

- 애플리케이션은 사용자가 기대한 기능을 수행
- 시스템은 사용자가 범한 실수나 예상치 못한 소프트웨어 사용법을 허용할 수 있다.
- 시스템 성능은 예상된 부하와 데이터 양에서 필수적인 사용 사례를 충분히 만족한다.
- 시스템은 허가되지 않은 접근과 오남용을 방지한다.


- 결함(fault): 잘못될 수 있는 일
- 내결함성(fault-tolerant): 결함을 예측하고 대처할 수 있는 능력 (= resilient)
- 모든 종류의 결함을 견딜 수 있는 시스템은 없음
- 결함은 장애(failure)와 동일 하지 않음
- 장애(failure): 사용자에게 필요한 서비스를 제공하지 못하고 시스템 전체가 멈춘 경우


### 하드웨어 결함
- 하드디스크가 고장 나고, 램에 결함이 생기고, 정전이 발생하고 등등
- 하드웨어 구송 요소에 중복(redundancy)를 추가하는 방법
	- 디스크는 RAID 구성으로 설치
	- 서버는 이중 전원 디바이스와 핫스왑 가능한 CPU
	- 데이터센터는 건전지와 예비 전원용 발전기
- 최근까지만해도, 중복으로도 충분했음
- 데이터 양과 애플리케이션의 계산 요구가 늘어나면서 하드웨어 결함율도 증가

### 소프트웨어 오류
- 하드웨어 결함은 보통 서로 독립적이라고 생각함
- 시스템 내 체계적 오류(systematic error)는 노드간 상관관계 때문에 시스템 오류를 더욱 많이 유발함
- 신속한 해결책이 없음
	- 시스템의 가정과 상호작용에 대해 주의 깊게 생각하기
	- 빈틈없는 테스트
	- 프로세스 격리
	- 죽은 프로세스의 재시작 허용
	- 프로덕션 환경에서 시스템 동작의 측정, 모니터링, 분석하기 등

### 인적 오류
- 오류의 가능성을 최소화하는 방향으로 시스템을 설계하라.
- 사람이 가장 많이 실수하는 장소에서 장애가 발생할 수 있는 부분을 분리하라.
- 단위 테스트부터 통합 테스트와 수동 테스트까지 철저하게 테스트하라.
- 인적 오류를 빠르고 쉽게 복구할 수 있게 하라.
- 명확한 모니터링 대책을 마련하라.
- 조작 교육과 실습을 시행하라.

## 확장성

확장성은 증가한 부하에 대처하는 시스템 능력을 설명하는데 사용하는 용어지만, 시스템에 부여하는 일차원적인 표식이 아님.

### 부하 기술하기

- 부하는 부하 매개변수(load parameter)라 부르는 몇 개의 숫자로 나타낼 수 있음
- 웹 서버의 초당 요청 수, 데이터베이스의 읽기 대 쓰기 비율, 대화방의 동시 활성 사용자, 캐시 적중률 등
- 평균적인 경우가 중요할 수도 있고, 소수의 극단적인 경우가 병목 원인일 수 있음

### 성능 기술하기

시스템 부하를 기술하면 부하가 증가할 때 어떤 일이 일어나는지 조사할 수 있음
- 부하 매개변수를 증가시키고 시스템 자원은 변경하지 않고 유지하면 시스템 성능은 어떻게 영향을 받을까?
- 부하 매개변수를 증가시켰을 때, 성능이 변하지 않고 유지되길 원한다면 자원을 얼마나 많이 늘려야 할까?


- 하둡 같은 일괄 처리 시스템은 보통 처리량(throughput)에 관심을 가짐.
- 온라인 시스템에서는 응답 시간(response time)을 더 중요시 함.
- 성능은 단일 숫자가 아니라 분포로 생각해야 함
- 평균 또는 백분위를 사용하는데, 백분위를 사용하는 편이 더 좋음

### 부하 대응 접근 방식
- 용량 확장(scaling up) = 수직 확장(vertical scaling)
- 규모 확장(scaling out) = 수평 확장(horizontal scaling)
- 탄력적(elastic) 시스템: 부하를 감지하면 자동으로 자원을 추가
- stateless 서비스를 배포하는건 간단
- stateful 데이터 시스템을 분산 설치하는 일은 복잡도가 높음
- 고가용성 요구가 있을 때까지 단일 노드에 유지하는 것이 최근까지의 통념


## 유지보수성
주의를 기울여야 할 소프트웨어 시스템 설계 원칙은 다음 세가지임
- 운용성(operability)
	운영팀이 시스템을 원할하게 운영할 수 있게 만들어라
- 단순성(simplicity)
	시스템에서 복잡도를 최대한 제거해 시스템을 이해하기 쉽게 만들어라
- 발전성(evolvability)
	시스템을 쉽게 변경할 수 있게 하라. extensibility, modifiability, plasticity

### 운용성: 운영의 편리함 만들기

운영 중 일부 측면은 자동화할 수 있고, 자동화해야 한다.  
좋은 운영성이란 동일하게 반복되는 태스크를 쉽게 수행하게끔 만들어 운영팀이 고부가가치 활동에 노력을 집중한다는 의미.


### 단순성: 복잡도 관리

- 우발적 복잡도를 제거하기 위한 최상의 도구는 추상화
- 좋은 추상화는 깔끔하고 직관적인 외관 아래로 많은 세부 구현을 숨길 수 있음
- 다양한 애플리케이션에서도 사용 가능

### 발전성: 변화를 쉽게 만들기

- 애자일 작업 패턴은 변화에 적응하기 위한 프레임워크를 제공
- 테스트 주도 개발(TDD)
- 리팩토링
