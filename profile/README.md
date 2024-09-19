Liar-Game Dev 버전 API 서버
======================

# 1. 주제 및 방식 
* MSA 아키텍처로 구성된 '라이어'를 찾는 실시간 웹 게임 1인 프로젝트 입니다. 
* 라이어 게임은 4 ~ 20명의 유저가 참여하여 시민 혹은 라이어를 배정받습니다.
* 시민에게는 주제를 알려주고, 라이어는 주제를 받지 못합니다.
* 주어진 주제를 바탕으로 그 주제를 라이어에게 들키지 않고 설명하고, 라이어는 시민이 설명하는 것을 읽고 자신의 정체가 들키지 않게 그 주제를 설명해야 합니다.
* 턴이 모두 종료되면 유저들은 투표를 통해 라이어를 지목합니다.
* 라이어를 지목한다면 주제를 맞출 수 있는 기회가 생기고, 라이어가 아닌 다른 시민이 다득표가 된다면(동률 포함) 라이어가 승리합니다.
* 신서유기 '라이어 게임' https://www.youtube.com/watch?v=oJdGyCobS4g

# 2. 기술 스택
#### - Framework: SpringBoot3.0.2, SpringCloud </br>
#### - ORM: Spring Data JPA </br>
#### - Infra: AWS EC2, AWS RDS(MySQL8.0.28), AWS ElasticCache(Redis), AWS SQS </br>
#### - OS: Linux Ubuntu22.04


# 3. 구조도
## 3.1 전체 구조도 인프라
![image](https://user-images.githubusercontent.com/88478829/229703902-6bfbfc68-c2e2-4521-8aa9-97264d1e98ac.png)

## 3.2 EC2 배포 인프라
![image](https://user-images.githubusercontent.com/88478829/229705478-5d6165f9-c21b-496f-bc04-210c0a16f5f4.png)
- 비용적인 문제로 인해, t2.micro의 메모리가 허용하는 한도까지 여러 개의 서버를 동일 인스턴스에 배포하였습니다. 
- 박스 표시는 t2.micro 인스턴스 하나에 포함되어 있는 서버를 의미합니다.

## 3.3 RDBMS 데이터베이스 인프라
![image](https://user-images.githubusercontent.com/88478829/229705548-30c05032-9ad9-4677-8d05-3e6dd0de61df.png)
- AWS 에서는 RDS를 활용하여 데이터베이스를 구현하였습니다.

## 3.4 NO-SQL 인프라
![image](https://user-images.githubusercontent.com/88478829/229705666-484faa1d-5194-424f-9bc0-486500a1d59c.png)
- Gateway, Member, Wait, Game 서버는 Redis를 인메모리 데이터베이스로 활용하고 있습니다.
- Result 서버는 RedissonClient를 활용하기 위한 Redis 서버로 활용하였습니다.

# 4. 주요 목표
- 테스트 코드 준수
- 트랜잭션 관리
- 동시성 관리
- 웹 소켓 적용
- 인프라 관리
- 보안 관리


# 5. 주요 기능 상세
전체 local 환경 프로젝트: https://github.com/gosekose/The-Liar-game

Eureka-Server
- Spring Cloud 서버 등록 관리

Config-Server
- bootstrap.yml로 동적 프로파일 변경

GateWay-Server
- AuthorizationHeaderFilter로 Jwt 인증 통합 관리
- https://gose-kose.tistory.com/36

Member-Server
- 중복 로그인 방지를 위한 LogoutSessionToken 추가
- 회원 가입 동시성 방지를 위한 RedissonClient 분산락 추가
- 로그아웃 시 발생하는 Redis Transaction 관리
- https://gose-kose.tistory.com/33

Wait-Server
- 웹 소켓으로 실시간 대기실 처리
- StompAccessHeader로 보안 처리
- https://gose-kose.tistory.com/35

Game-Server
- 웹 소켓으로 실시간 라이어 게임 진행
- 발언 순서를 지키고, 모든 순서가 종료되면 투표 진행
- 투표 결과에 따른 승패를 결정하고 Result-Server로 결과 저장 위임

Result-Server
- AWS SQS로 전달받은 데이터를 RDS에 저장
- 최근 게임 저장 조회 기능
- 레벨 업 시스템 및 랭킹 시스템 제공
- 최적의 조회 기능 도출을 위한 쿼리 실행 계획 비교
- https://gose-kose.tistory.com/34

# 6. 부족한 점
- 동시성 문제를 해결하는 과정에서 Result-Server에서 player의 레벨 업을 시키는 과정에 데이터 동기화가 이뤄지지 않았습니다.
- 일부 프론트를 생성하여 WebsocketTest를 진행하였지만, stmopClient의 경우 계속된 에러로 인해 테스트 방향을 API 호출 결과의 동일성 비교로 바꾸게 되었습니다. 이 부분은 추후 계속 연구해나가서 stompClient를 통해 프론트를 작성하지 않고도 소켓 API 테스트가 진행될 수 있도록 하겠습니다.
- Jenkins - Docker로 CI/CD를 구축하고 싶었지만, 도커 배포 시 많은 메모리 소모로 초과 비용이 발생하여 개별 배포로 진행하였습니다.

# 7. 배울 수 있었던 점
- 1인 프로젝트로 MSA를 구축하며, 서버 간 연동 시 중복되는 코드를 처리하기 위한 인증 통합화 과정을 배울 수 있었습니다.
- 비용 문제로 인해 도중에 서버를 내리고 다시 개설해야하는 상황이 생길 때, 매번 코드를 변경하기보다 Profile 설정으로 jar를 다양한 환경에 배포할 수 있다는 점을 배울 수 있었습니다.
- 다음 프로젝트는 Spring Cloud를 잘 사용하기 위해서킷 브레이커와 카프카를 도입하여 다양한 방식으로 기술을 적용해보고 싶습니다.




