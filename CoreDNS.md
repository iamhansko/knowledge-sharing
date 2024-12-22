# CoreDNS

Golang = Memory-Safe 적용된 프로그래밍 언어
DNS의 취약점은 Memory Access Error - 버퍼 오버플로우 또는 오버런 및 댕글링 포인터로 발생
-> Memory-Safe 적용된 CoreDNS는 발생 X

Golang = 동시성 또는 병렬 실행 지원
멀티프로세싱, 멀티태스킹 시스템에서 성능 끌어올리기 유용
단, C, C++보다는 약간 느리게 실행

CoreDNS
서비스 디렉터리 
  = 특정 서비스 제공하는 컨테이너의 IP 주소 결정
  = 특정 서비스가 실행되는 위치 추적
고급 기능
- 질의 및 응답 즉시 재작성
- GitHub 또는 Route53에서 Zone 자동 로드

Caddy를 Fork
2019.1 CNCF 졸업

Plugins
Caching Plugin
orwarding Plugin
Primary DNS Server Plugin
Secondary DNS Server Plugin
기능 확장 시 직접 Plugin 작성

컨테이너, etcd, k8s와 통신 가능

DNS Namespace 
Root DNS 서버에서 시작해 Root DNS 서버에 대한 질의 권한 있는 DNS 서버 중 하나로부터 응답받을 때까지 Referral 따라 질의 처리 불가. 대신 다른 DNS 서버 Forwarder에 의존 

Recursion 가능??

DNS
IP 주소, 메일 라우팅 등의 데이터에 이름을 매핑하는 명명 시스템
분산 데이터베이스의 일종
DNS 클라이언트가 DNS 서버를 질의해 분산된 DB에 저장된 데이터를 검색하는 클라이언트-서버 시스템이기도 함

Domain Name
DNS Namespace = 반전 트리, 루트 노드가 상단, 각 노드는 임의 수의 자식 노드를 가질 수 있음. 
각 노드에는 Label 최대 63개 ASCII 문자 길이로 명시
루트 노드는 0의 null label

label은 한 노드와 비슷한 동기 노드를 구별하는 데에만 유용
노드의 domain name은 해당 노드에서 루트까지의 경로에 있는 label 목록
domain name은 식별자로, 전체 네임스페이스에서 특정 노드를 식별하는 데 필요

도메인, 위임, 영역
Domain = 네임스페이스의 특정 하위 트리에 있는 노드 그룹, 도메인은 정점(최상위 노드)의 노드로 식별됨
Delegation = 특정 조직에 특정 하위 트리(서브 도메인) 제어하게끔 맡김
영역 = 다른 곳에 위임된 하위 도메인을 뺀 도메인

DNS 데이터는 리소스 레코드 단위로 저장 (다양한 클래스 + 타입)
각 레코드 타입에는 특정 형식으로 짧은 RDATA라는 레코드별 데이터 필요
IN 클래스
  레소스 레코드 타입
    - A (IPv4)
      도메인명을 단일 IPv4 주소로 매핑
    - AAAA (IPv6)
      도메인명을 단일 IPv6 주소로 매핑
    - CNAME (Alias)
      도메인명을 단일 다른 도메인명으로 매핑
    - MX (Mail Exchanger)
      이메일 대상에 대한 메일 교환기 이름 지정
    - NS (Name Server)
      영역에 대한 네임 서버의 이름
    - PTR (Pointer)
      IP 주소를 도메인명에 다시 매핑
    - SOA (Start Of Authority)
      영역에 대한 매개 변수 제공
Hesiod, Chaosnet 클래스는 거의 쓰이지 않음

DNS 서버
데이터 파일 또는 이와 동등한 마스터 파일에서 영역 데이터 로드
영역 전송 - 다른 DNS 서버의 영역 데이터 로드

(해당 영역의) 주 DNS 서버
영역 데이터 파일에서 영역에 대한 정보를 로드하는 DNS 서버
보조 DNS 서버가 영역을 전송받는 DNS 서버
영역 전송 후 보조 DNS 서버는 영역 데이터의 복사본을 디스크에 저장, 때로는 백업 영역 데이터 파일에 저장
보조 DNS 서버가 주기적으로 마스터 DNS 서버에 영역 전송 요청 -> 새 버전의 영역 전송받으면 디스크 데이터 업데이트

(해당 영역의) 보조 DNS 서버
다른 DNS 서버에서 영역에 대한 정보를 로드하는 DNS 서버

디스크 -> (영역 데이터 파일 적재) -> 주 DNS 서버 -> (영역 전송) -> 보조 DNS 서버 -> (백업 영역 데이터) -> 디스크

본인 영역의 도메인명에 대해서는 어떤 질의에도 확실하게 응답 가능
일부 영역에서는 주 DNS 서버, 다른 영역에서는 보조 DNS 서버일 수 있음

Resolver
도메인명 정보 요청, DNS 질의로 변환, 질의를 DNS 서버로 보내고 응답 기다림
합리적인 시간 내에 응답 받지 못하면 동일 DNS 서버로 재질의 또는 다른 DNS 서버로 질의