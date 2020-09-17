---
title: '(AWS시리즈6) AWS RDS'
date: 2020-09-14 16:00:00
category: 'aws'
draft: false
---

## AWS에 데이터베이스 환경 만들기

- 데이터베이스를 구축하고 EC2 서버와 연동하기.
- 직접 데이터베이스를 설치하지 않는다. 직접 데이터베이스를 설치해서 다루게 되면 모니터링, 알람, 백업, HA 구성 등을 모두 직접 해야만 하므로 번거롭다.
- AWS는 위의 작업을 모두 지원하는 관리형 서비스 RDS(Relational Database Service)를 제공한다. RDS는 AWS에서 지원하는 클라우드 기반 관계형 데이터베이스다. 하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같이 잦은 운영 작업을 자동화하여 개발자가 개발에 집중할 수 있게 지원하는 서비스다. 조정 가능한 용량을 지원하여 예상치 못한 양의 데이터가 쌓여도 비용만 추가로 내면 정상적으로 서비스가 가능한 장점도 있다.

### RDS 인스턴스 생성하기

- AWS의 서비스 검색창에 `rds`를 검색
- RDS 대시보드에서 `데이터베이스 생성` 버튼 클릭
- DB 엔진 선택 화면에서 Maria DB 선택
  - cf) [왜 마리아 DB를 선택하는가](<#참고(MariaDB선택이유)>)
- `프리티어` 선택
- 상세 설정 1
  - 스토리지 유형은 `범용(SSD)`, 할당된 스토리지는 `20`
  - DB 인스턴스 클래스 : db.t2.micro - 1 vCPU, 1 GiB RAM
- 상세 설정 2
  - DB 인스턴스 식별자 : `freelec-springboot2-webservice`
  - 자격 증명 설명(마스터 사용자 이름) : id/password 입력
  - 네트워크 : 퍼블릭 액세스 `예`. 이후 보안 그룹에서 지정된 IP만 접근하도록 막을 예정
- 데이터베이스 옵션
  - 데이터베이스 이름 `freelec_springboot2_webservice
  - 포트 `3306`
  - DB파라미터 그룹 : `default.mariadb10.2`
  - 옵션 그룹 : `default.mariadb-10-2`
- `완료`
  - 생성 과정이 진행. `DB 인스턴스 세부 정보 보기` 클릭

### RDS 운영환경에 맞는 파라미터 설정하기

- RDS를 처음 생성하면 몇 가지 설정을 필수로 해야 한다.(타임존, character set, max connection)
- 왼쪽의 `데이터베이스` 카테고리 밑에서 `파라미터 그룹` 클릭
- `파라미터 그룹 생성` 클릭
- 세부 정부 위쪽에 DB엔진을 선택하는 항목이 있는데, 위에서 생성한 MariaDB와 같은 버전을 맞춰야 한다. 만약 위에서 10.2.xx버전을 선택했으면 같은 버전대인 `10.2`를, 만약 10.3.xx버전으로 생성하였다면 `10.3`을 선택하면 된다. 그룹 이름과 설명은 둘 다 `freelec-springboot-webservice` 로 선택
- 생성이 완료되면 파라미터 그룹 창에 새로 생성된 그룹 `freelec-springboot2-webservice` 클릭
- 오른쪽 위의 `파라미터 편집` 클릭

1. 타임존

   - time_zone을 검색해서 `asia/seoul` 클릭

1. Character Set

   - 아래는 다 `utf8mb4`
     - character_set_clinet
     - character_set_connection
     - character_set_database
     - character_set_filesystem
     - character_set_results
     - character_set_server
   - 아래는 다 `utf8`
     - collation_connection
     - collation_server
   - `utf8mb4`와 `utf8`의 차이는 이모지 저장 가능 여부

1) Max Connection
   - rds의 max connection은 인스턴스 사양에 따라 자동으로 정해진다
   - `150`
   - 추후에 사양을 더 높이게 된다면 기본값으로 다시 돌려놓으면 된다

### 생성된 파라미터 그룹을 데이터베이스에 연결

- `데이터베이스` - 생성된 데이터 베이스 `체크` - `수정`
- 옵션 항목에서 DB 파라미터 그룹은 default로 되어 있음.
- DB파라미터 그룹을 방금 생성한 신규 파라미터 그룹으로 변경(freelec-springboot2-webservice)
- `저장`을 누르면 수정사항이 요약된 것 확인가능
- 수정 사항 요약은 `즉시 적용`
- 파라미터 그룹 반영 위해 데이터베이스 `재부팅`

### 내 pc에서 RDS에 접속 (헷갈리면 책 참조 p281)

- 로컬 pc에서 rds로 접근하기 위해 RDS의 보안 그룹에 본인 pc의 ip를 추가
- RDS의 세부 정보 페이지에서 `보안그룹` 항목 클릭
- 새 브라우저에서 EC2에 사용된 보안 그룹의 그룹id 복사
- 복사된 `보안 그룹 ID`와 `본인의 IP`를 RDS 보안 그룹의 인바운드에 추가
- 인바운드의 규칙 유형에서는 `MYSQL/Aurora`를 선택하면 자동으로 3306포트가 선택된다.
  - 보안그룹 첫번째 줄 - 현재 내 pc의 ip 등록
  - 보안그룹 두번째 줄 - EC2의 보안 그룹 추가
    - 이렇게 하면 EC2와 rds 간에 접근 가능

### Database 플러그인 설치 및 설정

- 로컬에서 원격 데이터베이스로 붙을 때 GUI 클라이언트를 많이 사용
- MYSQL의 대표적인 클라이언트로 Workbench, SQLyo, 등이 있는데 가장 좋아하는 툴로 그냥 하면 됨
- 인텔리제이에 database 플러그인 설치
- RDS의 정보 페이지에서 `엔드 포인트`를 확인. 엔드 포인트가 접근 가능한 url이다.
- 책에서는 `databse navigator` 플러그인 설치(하지만 할 필요 없다)
- 새로 업데이트 된 인텔리J(ultimate 버전) 에서는 Database navigator플러그인이 포함되어 있다. `databse tools and sql`이다. 실행은 view - tool windows - databse
- 프로젝트 왼쪽 사이드바에 `DB Browser`가 노출되는데, DB Browser 탭 하단에 기존에 노출되던 프로젝트 항목들이 보인다. MariaDB는 MySQL 기반이므로 MySQL을 사용하면 된다.
- Host에 rds의 `엔드 포인트`(url) 넣고 `마스터 계정/비밀번호` 입력
- `Test Connection` 로 연결 테스트 설정
- `Conenction SuccessFul` 메시지 보면 apply->ok
- `RDS의 스키마`가 노출된다. 위쪽에 `Open SQL Console` 버튼 클릭 해서 `New SQL Console` 항목 선택해서 콘솔창 오픈
- 쿼리가 수행될 datase 선택 쿼리

  ```
  use AWS RDS 웹 콘솔에서 지정한 데이터베이스명;
  ```

  ```
  use freelec_springboot2_webservice;
  ```

- `Execute Console` 에서 `SQL statement executed successfully` 가 뜨면 쿼리가 정상수행 된 것임

- 데이터베이스 선택된 상태에서 현재의 `character_set, collation` 설정 확인

  ```
  show variables like 'c%';
  ```

- 쿼리 결과보면 character_set_database, collation_connection 2가지 항목이 latin1로 되어 있는데, 이것들은 MariaDB에서만 RDS 파라미터 그룹으로는 변경이 안 된다.

  ```
  ALTER DATABASE 데이터베이스명
  CHARACTER SET = `utf8mb4`
  COLLATE = `utf8mb4_general_ci`;
  ```

  적용 후 다시 확인

  ```
  show variables like 'c%';
  ```

- 타임존도 확인

  ```
  select @@time_zone, now();
  ```

- 한글명이 잘 들어가는지 간단한 테이블 생성과 insert 쿼리를 실행

  - 테이블 생성은 인코딩 설정 변경 전에 생성하면 안 된다. 만들어질 당시의 설정값을 그대로 유지하고 있어, 자동 변경이 되지 않고 강제로 변경해야만 한다. 웬만하면 테이블은 모든 설정이 끝난 후에 생성하는 것이 옳다.

  ```
  CREATE TABLE    test (
      id bigint(20) NOT NULL AUTO_INCREMENT,
      content varchar(255) DEFAULT NULL,
      PRIMARY KEY(id)
  )   ENGINE=InnoDB;

  insert into test(content) values (`테스트`);

  select * from test;

  ```

- 이렇게 RDS에 대한 설정 끝

### EC2에서 RDS로 접근 확인

- (윈도)에서 ec2에 ssh 접속
- MySQL 접근 테스트 위해 MySQL CLI 설치 (실제 EC2의 MySQL를 설치해서 쓰는게 아니라 명령어 라인만 쓰기 위한 설치임)
  ```
  sudo yum install mysql
  ```
- RDS에 접속

  ```
  mysql -u 계정 -p -h host주소
  ```

  ```
  mysql -u june -p -h (엔드포인트 주소)
  ```

- 패스워드 입력하라고 하면 입력하면, EC2에서 RDS로 접속되는 것 확인가능

- 실제로 생성한 RDS가 맞는지 간단한 쿼리 사용

  ```
  show databases;
  ```

### 참고(MariaDB선택이유)

- RDS에는 오라클, MSSQL, PostgreSQL 등이 있으며, 당연히 본인이 가장 잘 사용할 줄 아는 데이터베이스를 고르면 되지만, 꼭 다른 데이터베이스를 선택해야 할 이유가 있는 것이 아니라면 MySQL, MariaDB, PostgreSQL 중에 고르길 추천. 그 중에서 MariaDB를 추천하며 그 이유는 아래와 같다.

  1.  가격
      - RDS의 가격은 라이센스 `비용 영향`을 받는다. 상용 데이터베이스인 오라클, MYSQL이 오픈소스인 MySQL, MariaDB, PostgreSQL보다는 동일한 사양 대비 가격이 더 높다. 결국 프리티어 기간인 1년이 지나면 비용을 지불하면서 RDS를 써야 한다.
  1.  Amazon Aurora(오로라) 교체 용이성
      - Amazon Aurora는 AWS에서 MySQL과 PostgreSQL을 클라우드 기반에 맞게 재구성한 데이터베이스이다. 공식 자료에 의하면 RDS MySQL 대비 5배, RDS PostgreSQL보다 3배의 성능을 제공한다. 더군다나 AWS에서 직접 엔지니어링하고 있기에 계속 발전 중. `Amazon Aurora는 성능 짱`
      - `클라우드 서비스에 가장 적합한 데이터베이스이기에 많은 회사가 Amazon Aurora를 선택`. 그러다 보니 호환 대상이 아닌 오라클, MSSQL을 굳이 선택할 필요가 없다. 이렇게 보면 Aurora를 선택하면 가장 좋을 것 같지만 `그런데?` 시작하는 단계에서는 Aurora를 선택하기 어렵다. 프리티어 대상이 아니다. `최저 비용이 월 10만원 이상`이기에 MariaDB로 시작
      - 차후 서비스 규모가 커지면 MariaDB -> Aurora로 이전하면 된다.

  - MariaDB??
    - 국내외를 가리지 않고 오픈소스 데이터베이스 중에서 가장 인기있는 제품이라면 MySQL이다. 단순 쿼리 처리 성능이 어떤 제품보다 압도적이며 이미 오래 사용되어 왔기 때문에 성능, 신뢰도 짱
    - MySQL을 기반으로 만들어졌으므로 쿼리를 비롯한 전반적인 사용법이 MySQL과 유사하다.
    - MariaDB는 MySQL 대비 다음과 같은 장점이 있다.
      - 동일 하드웨어 사양으로 MySQL보다 향상된 성능
      - 좀 더 활성화된 커뮤니티
      - 다양한 기능
      - 다양한 스토리지 엔진

참고 : 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 - 이동욱
