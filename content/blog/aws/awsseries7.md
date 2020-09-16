---
title: '(AWS시리즈7) EC2 서버에 프로젝트 배포'
date: 2020-09-16 17:00:00
category: 'aws'
draft: false
---

## EC2에 프로젝트 clone 받기

- 깃허브에서 코드를 받아올 수 있게 EC2 서버에 git 설치
- (윈도우는 putty로) EC2로 접속해서 다음 명령어들 입력

```
sudo yum install git
```

- 설치 완료되면 다음 명령어로 설치 상태 확인

```
git --version
```

- 깃이 성공적으로 설치되면 git clone으로 프로젝트를 저장할 디렉토리 생성

```
mkdir ~/app && mkdir ~/app/step1
```

- 생성된 디렉토리로 이동

```
cd ~/app/ste1
```

- 깃허브 repository(프로젝트 있는) 주소 복사

```
git clone (복사한주소)
```

- clone 끝났으면 클론된 프로젝트로 이동해서 파일들이 잘 복사되었는지 확인
  - 아래 명령어를 통해, README.md, build, build.gradle.. 등이 있으면 된다.

```
cd 프로젝트명(freelec-springboot2-webservice)
ll
```

- 코드들이 잘 수행되었는지 테스트로 검증

  - 만약 테스트가 실패해서 수정하고 깃허브에 푸시 했다면 프로젝트 폴더 안에서

  ```
  git pull
  ```

  - 테스트 검증

  ```
  ./gradlew test
  ```

  - 만약 permission denined가 되면

  ```
  chmod +x ./gradlew
  ./gradlew test
  ```

## 배포 스크립트 만들기

- 작성한 코드를 실제 서버에 반영하는 것을 배포라고 한다. 배포라 함은 다음의 과정을 모두 포괄하는 의미이다

  - git clone 혹은 git pull을 통해 새 버전의 프로젝트를 받음
  - Gradle이나 Maven을 통해 프로젝트 테스트와 빌드
  - EC2 서버에서 해당 프로젝트 실행 및 재실행

- 위의 과정을 배포할 때마다 개발자가 하나하나 명령어를 실행하는 것은 불편함이 많다. 그래서 이를 `쉘 스크립트`로 작성해 스크립트만 실행하면 앞의 과정이 차례로 진행되도록 하는 것. 참고로 쉘 스브립트와 빔(vim)은 서로 다른 역할을 한다. 쉘 스크립트는 `.sh라는 파일 확장자를 가진 파일`이다. 노드 JS가 .js라는 파일을 통해 서버에서 작동하는 것처럼 `쉘 스크립트 역시 리눅스에서 기본적으로 사용할 수 있는 스크립트 파일의 한 종류`이다.
  `빔`은 `리눅스 환경과 같이 GUI(윈도우와 같이 마우스를 사용할 수 있는 환경)가 아닌 환경에서 사용할 수 있는 편집 도구`. 리눅스에선 빔 이외에도 이맥스, 나노 등의 도구를 지원한다. 하지만 빔(vim)이 가장 대중적인 도구이므로 빔을 선택!

- ~/app/step1/에 deploy.sh 파일을 하나 생성

```
vim ~/app/step1/deploy.sh
```

- deploy.sh

  ```
    #!/bin/bash

    REPOSITORY=/home/ec2-user/app/step2
    PROJECT_NAME=freelec-springboot2-webservice

    cd $REPOSITORY/$PROJECT_NAME/

    echo "> Git Pull"

    git pull

    echo "> 프로젝트 build 시작"

    ./gradlew build

    echo "> step1 디렉토리로 이동"

    cd $REPOSITORY

    echo "> Build 파일 복사"

    cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

    echo "> 현재 구동중인 애플리케이션 pid 확인"

    CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)

    echo "현재 구동중인 어플리케이션 pid: $CURRENT_PID"

    if [ -z "$CURRENT_PID" ]; then
        echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
    else
        echo "> kill -15 $CURRENT_PID"
        kill -15 $CURRENT_PID
        sleep 5
    fi

    echo "> 새 어플리케이션 배포"

    JAR_NAME=$(ls $REPOSITORY/*.jar | tail -n 1)

    echo "> JAR Name: $JAR_NAME"

    nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &
  ```

  - `REPOSITORY=/home/ec2-user/app/step2` , `PROJECT_NAME=freelec-springboot2-webservice`

    - 프로젝트 디렉토리 주소는 스크립트 내에서 자주 사용하는 값이므로 이를 변수에 저장
    - 쉘에서는 타입 없이 선언하여 저장
    - 쉘에서는 \$ 변수명으로 변수를 사용할 수 있다

  - `cd $REPOSITORY/$PROJECT_NAME/`

    - 제일 처음 git clone 받았던 디렉토리로 이동
    - /home/ec2-user/app/step1/freelec-springboot2-webserivce로 이동

  - `./gradlew build`

    - 프로젝트의 내부의 gradlew로 build를 수행

  - `cd ./build/libs/*.jar $REPOSITORY/`

    - build의 결과물인 jar 파일을 복사해 jar파일을 모아둔 위치로 복사

  - `CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)`

    - 기존에 수행 중이던 스프링 부트 애플리케이션을 종료
    - pgrep은 process id만 추출하는 명령어
    - -f옵션은 프로세스 이름으로 추천

  - `if ~ else ~ fi`

    - 현재 구동중인 프로세스가 있는지 없는지를 판단해서 기능을 수행
    - process id 값을 보고 프로세스가 있으면 해당 프로세스 종료

  - `JAR_NAME=$(ls $REPOSITORY/*.jar | tail -n 1)`

    - 새로 실행할 jar 파일을 찾는다
    - 여러 jar 파일이 생기기 때문에 tail -n 로 가장 나중의 jar파일(최신파일) 을 변수에 저장한다.

  - `nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &`
    - 찾는 jar 파일명으로 해당 jar파일을 nohup으로 실행
    - 애플리케이션 실행자가 터미널을 종료해도 애플리케이션은 계속 구동될 수 있도록 nohup 명령어 사용(java -jar 명령어로도 자바 실행할 수 있지만 터미너 접속이 끊기면 애플리케이션도 같이 종료된다)
    - 스프링 부트의 장점으로 특별히 외장 톰캣을 설치할 필요가 없다
    - 내장 톰캣을 사용해서 jar파일만 있으면 바로 웹 애플리케이션을 서버를 실행할 수 있다.

- 이렇게 생성한 스크립트에 실행 권한 추가 후 권한 확인

```
chmod +x ./deploy.sh
ll
```

- 실행

```
./deploy.sh
```

- 잘 실행되었으니 nohup.out 파일을 열어 로그 화인. nohup.out은 실행되는 애플리케이션에서 출력되는 모든 내용을 갖고 있다.

```
vim nohup.out
```

- nohup.out제일 아래로 가면 `ClientRegistrationRepiostory`를 찾을 수 없다(that could not be found).는 에러가 발생하면 애플리케이션 실행에 실패했다는 것을 알 수 있다. `왜 그럴까?`

## 외부 Security 파일 등록하기

- 위의 오류에 대한 이유는? `ClientRegistrationRepiostory`를 생성하려면 `clientId`와 `clientSecret`가 필수다. 로컬 pc에서 실행할 때는 application-oauth.properties가 있어서 문제가 없었다. 하지만 이 파일은 .gitignore로 git에서 제외 대상이므로 깃허브에 올라가있지 않다.

- 애플리케이션 실행하기 위해 공개된 저장소에 id와 secret을 올릴 수 없으므로 서버에서 직접 이 설정들을 가지고 있게 한다

- 먼저 step1이 아닌 app디렉토리에 properties 파일을 생성한다

```
vim /home/ec2-user/app/application-oauth.properties
```

- 그리고 로컬에 있는 application-oauth.properties 파일 내용을 그대로 붙여넣기. 해당 파일을 저장하고 종료(:wq). 그리고 방금 생성한 application-oauth.properties을 쓰도록 deploy.sh 파일을 수정한다. `vim deploy.sh` 로 파일 열기

  ```
  nohup java -jar \
  -Dspring.config.location=classpath:/application.properties,/home/ec2-user/app/application-oauth.properties \
  $REPOSITORY/$JAR_NAME 2>&1 &
  ```

  - `-Dspring.config.location`

    - 스프링 설정 파일 위치를 설정
    - 기본 옵션들을 담고 있는 application.properties과 OAuth 설정들을 담고 있는 application-oauth.properties의 위치를 지정한다
    - classpath가 붙으면 jar 안에 있는 resources 디렉토리를 기준으로 경로가 생성
    - application-oauth.properties 은 절대경로를 사용. 외부에 파일이 있기 때문

- 수정이 다 되었다면 deploy.sh를 실행 `./deploy.sh` 명령어로
- `vim nohup.out`으로 확인

## 스프링 부트 프로젝트로 RDS 접근하기

- RDS는 MariaDB를 사용중이다. 이 MariaDB에서 스프링부트 프로젝트를 실행하기 위해서 몇 가지 작업이 필요하다

  - 테이블 생성 : H2에서 자동생성해주는 테이블을 MariaDB에서 스프링부트 프로젝트를 실행하기 위해서 몇 가지 작업이 필요
  - 프로젝트 설정 : 자바 프로젝트가 MariaDB에 접근하려면 데이터베이스 드라이버가 필요하다. MariaDB에서 사용 가능한 드라이버를 프로젝트에 추가
  - EC2(리눅스 서버) 설정 : 데이터베이스의 접속 정보는 중요하게 보호해야 할 정보. 공개되면 외부에서 데이터를 모두 가져갈 수 있기 때문. EC2 서버 내부에서 접속 정보를 관리하도록 설정

1. RDS 테이블 생성

- RDS에 테이블을 생성. JPA가 사용될 엔티티 테이블과 스프링 세션이 사용될 테이블 2가지 종류를 생성. `JPA가 사용될 엔티티 테이블`과 `스프링 세션이 사용될 테이블` 2가지 종류를 생성

- 로컬(IDE - intelliJ에서)에서 테스트 코드를 수행하면 로그에 테이블 생성 query(posts, user 테이블 2개)를 볼 수 있다.

  - 이것을 복사하여 RDS에 반영(RDS의 console에서 실행)

- 스프링 세션 테이블은 schema-mysql.sql 파일에서 확인할 수 있다. File검색(Mac에선 Command+shift+O, 윈도우/리눅스에서는 Ctrl+Shift+N) 으로 찾을 수 있다. 해당 파일에는 `SPRING_SESSION` 테이블과 `SPRING_SESSION_ATTRIBUTES` 테이블 생성 쿼리가 있다.
  - 이를 복사하여 RDS에 반영

2. 프로젝트 설정

- 먼저 MariaDB 드라이버를 build.gradle에 등록한다

```
compile("org.mariadb.jdbc:mariadb-java-client")
```

- 서버에서 구동될 환경을 하나 구성

  - src/main/resources/에 application-real-properties 파일을 추가한다. application-real.properties로 파일을 만들면 profile=real인 환경이 구성된다고 보면 된다. 실제 운영될 환경이기 때문에 보안/로그상 이슈가 될만한 설정들을 모두 제거하며 RDS 환경 profile 설정이 추가된다

  ```
  spring.profiles.include=oauth,real-db
  spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
  spring.session.store-type=jdbc
  ```

- 깃허브로 푸시

3. EC2 설정

- OAuth와 마찬가지로 RDS 접속 정보도 보호해야 할 정보이니 EC2 서버에 직접 설정 파일을 둔다.

- app 디렉토리에 application-real-db.properties 파일 생성

```
vim ~/app/application-real-db.properties
```

- 아래 내용 추가

  ```
  spring.jpa.hibernate.ddl-auto=none

  spring.datasource.url=jdbc:mariadb://rds주소:3306/databsae이름
  spring.datasource.username=db계정
  spring.datasource.password=db계정 비밀번호
  spring.datasource.driver-class-nad.mariadb.jdbc.Driver
  ```

  - `spring.jpa.hibernate.ddl-auto=none`
    - JPA로 테이블이 자동 생성되는 옵션을 None(생성하지 않음)으로 지정한다
    - RDS에는 실제 운영으로 사용될 테이블이니 절대 스프링 부트에서 새로 만들지 않도록 해야 한다
    - 이 옵션을 하지 않으면 자칫 테이블이 모두 새로 생성될 수 있다.
    - 주의해야 하는 옵션

- 마지막으로 deploy.sh가 real profile을 쓸 수 있도록 다음과 같이 개선 `vim ~/app/step1/deploy.sh` 명령어 실행

  ```
  nohup java -jar \
    -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
    -Dspring.profiles.active=real \
    $REPOSITORY/$JAR_NAME 2>&1 &

  ```

  - `Dspring.profiles.active=real`
    - application-real.properties를 활성화
    - application-rela.properties의 spring.profiles.include=oauth, real-db 옵션 때문에 real-db 역시 활성화 대상에 포함된다

- 다시 한번 `./deploy.sh`명령어로 deploy.sh를 실행. `vim nohup.out`명령어로 nohup.out파일을 열어 다음과 같은 로그 확인

```
Tomcat started on port(s): 8080(http) with context path ``
(..생략..)
```

- curl 명령어로 html 코드가 정상적으로 보인다면 성공(EC2에 서비스가 잘 배포된 것 확인)

```
curl localhost:8080
```

## EC2에서 소셜 로그인하기

- curl명령어를 통해 EC2 서비스가 잘 배포된 것 확인했으니, 브라우저에서 확인해볼 텐데, 그 전에 몇 가지 작업

1. AWS 보안 그룹 변경
   - 먼저 EC2에 스프링 부트 프로젝트가 8080 포트로 배포되었으니, 8080포트가 보안 그룹에 열려 있는지 확인
   - `AWS` - `네트워크 및 보안` - Name이 `ec2`(나는 springboot2-webservice-ec2), 그룹 이름이 `freelec-springboot2-webservice`(나는 launch-wizard-1) 클릭 - 밑에 인바운드 탭에 8080포트가 `0.0.0.0/0, ::/0` 2가지 열려있는지 확인 - 8080 열려 있다면 OK, 안되어 있다면 `편집` 버튼을 눌러 추가
2. AWS EC2 도메인으로 접속
   - 왼쪽 사이드바의 `인스턴스` 메뉴 클릭. 본인이 생성한 EC2 인스턴스를 선택하면 다음과 같이 상세 정보에서 `퍼블릭 DNS` 를 확인
   - 이 주소가 EC2에 자동으로 할당된 `도메인`이다. 인터넷이 되는 장소 어디나 이 주소를 입력하면 우리의 EC2 서버에 접근할 수 있다.
   - 이 도메인 주소에 `:8080`을 붙여 브라우저에 입력
   - 이제 드디어 `우리 서비스가 도메인을 가진 서비스가 되었다`
   - 그런데 현재 상태에서는 해당 서비스에 EC2의 도메인을 등록하지 않았기 때문에 구글과 네이버 로그인이 작동하지 않는다. 그래서 등록해야 함
3. 구글에 EC2 주소 등록

   - [구글 웹 콘솔](https://console.cloud.google.com/home/dashboard) 로 접속하여 본인의 프로젝트로 이동한 다음 `API 및 서비스` - `사용자 인증 정보` 로 이동
   - `OAuth 동의 화면` 탭을 선택하고 아래에서 `승인된 도메인` 에 `http://` 없이 EC2의 퍼블린 DNS를 등록한다.
   - `사용자 인증 정보` 탭을 클릭하여 본인이 등록한 서비스의 이름을 클릭한다
   - 퍼블릭 DNS 주소에 `:8080/login/oauth2/code/google` 주소를 추가하여 승인된 리디렉션 URI에 등록한다
   - EC2 DNS 주소로 이동해서 다시 구글 로그인을 시도해보면 구글 로그인이 성공하는 것을 확인 가능하다

4. 네이버에 EC2 주소 등록

   - [네이버 개발자 센터](https://developers.naver.com/apps/#/myapps)로 접속해서 본인의 프로젝트로 이동
   - `PC 웹` 항목이 있는데, 여기서 `서비스 URL`과 `Callback URL` 2개를 수정
   - `서비스 URL`

     - 로그인을 시도하는 서비스가 네이버에 등록된 서비스인지 판단하기 위한 항목
     - 8080포트는 제외하고 실제 도메인 주소만 입력
     - 네이버에서 아직 지원되지 않아 하나만 등록 가능하다. 즉, EC2 주소를 등록하면 기존의 localhost가 안 된다. 그래서 개발 단계에서는 등록하지 않는 것을 추천
     - localhost도 테스트하고 싶으면 네이버 서비스를 하나 더 생성해서 키를 발급받으면 된다.

   - `Callback URL`

     - 전체 주소를 등록한다(EC2 퍼블릿 DNS: 8080/login/oauth2/code/naver).

   - 이렇게 2개 항목을 모두 수정/추가하였다면 구글과 마찬가지로 네이버 로그인을 시도해보면 로그인 성공을 확인 가능

## 아직 남은 문제점들

- 스프링 부트 프로젝트를 EC2에 배포했고 구글, 네이버 로그인도 EC2와 연동을 했다. `하지만`

1. 수동 실행되는 Test

   - 본인이 짠 코드가 다른 개발자의 코드에 영향을 끼치지 않는지 확인하기 위해 전체 테스트 수행해야 한다.
   - 현재 상태에선 항상 개발자가 작업 수행할 때마다 수동으로 전체 테스트 수행해야만 한다.

2. 수동 Build -다른 사람이 작성한 브랜치와 본인이 작성한 브랜치가 합쳐졌을 때(Merge) 이상이 없는지는 Build를 수행해해야만 알 수 있다.
   - 이를 매번 개발자가 직접 실행해봐야만 한다.

- 그래서 다음 시리즈에서는 이런 수동 Test & Build를 자동화시키는 작업을 진행하겠다. 깃허브에 푸시를 하면 자동으로 Test & Build & Deploy가 진행되도록 개선하는 작업

참고 : 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 - 이동욱
