---
title: '(AWS시리즈5) 아마존 리눅스1 서버 생성시 꼭 해야할 설정들'
date: 2020-09-03 21:00:00
category: 'aws'
draft: false
---

## 필수적 설정들

- 아마존 리눅스 1서버를 처음 받았다면, 몇 가지 설정들이 필요하다
- 이 설정들은 모두 자바 기반의 웹 애플리케이션 (톰캣과 스프링부트)가 작동해야 하는 서버들에겐 필수로 해야 하는 설정들이다

1. Java 8 설치
   - 만약 진행하고 있는 프로젝트의 버전이 Java 8이라면 필수
   - 아마존 리눅스1의 경우 기본 자바 버전이 7이다.
1. 타임존 변경
   - 기본 서버의 시간이 미국 시간대. 한국 시간대가 되어야만 우리가 사용하는 시간이 모두 한국 시간으로 등록되고 사용된다.
1. 호스트네임 변경
   - 현재 접속한 서버의 별명을 등록. 실무에서는 한 대의 서버가 아닌 수십 대의 서버가 작동되는데, IP만으로 어떤 서버가 어떤 역할을 하는지 알 수 없다. 이를 구분하기 위해 보통 호스트 네임을 필수로 등록한다.

---

### 1. EC2 서버에 Java 8 설치

- (windows) putty 로 EC2 접속하기
- 다음 명령어 실행

```
sudo yum install -y java-1.8.0-openjdk-devel.x86_64
```

- 설치가 완료되었다면 인스턴스의 Java버전을 8로 변경

```
sudo /usr/sbin/alternatives --config Java
```

- 선택화면에서 Java 8 선택

  - `2`를 입력

- 버전이 변경되었으면 사용하지 않는 Java 7을 삭제

```
sudo yum remove java-1.7.0-openjdk
```

- 현재 버전이 Java 8 이 되었는지 확인

```
java -version
```

### 2. 타임존 변경

- EC2 서버의 기본 타임존은 UTC이다. 이는 세계 표준 시간으로 한국의 시간대가 아님. 즉, 한국의 시간과는 9시간 차이가 발생한다. 이렇게 되면 서버에서 수행되는 Java 애플리케이션에서 생성되는 시간도 모두 9시간씩 차이가 나기 때문에 꼭 수정해야 할 설정.

```
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```

- 정상적으로 수행되었다면 date 명령어로 타임존이 KST로 변경된 것을 확인할 수 있다.

### 3. HostName 변경

- 여러 서버를 관리 중일 경우 IP만으로 어떤 서비스의 서버인지 확인이 어렵다
- 그래서 각 서버가 어느 서비스인지 표현하기 위해 HOSTNAME을 변경한다. 다음 명령어로 편집 파일 열기

```
sudo vim /etc/sysconfig/network
```

- 화면에서 노출되는 항목 중 HOSTNAME으로 되어있는 부분을 본인이 원하는 서비스명으로 변경가능하다

##### 변경전

```
NETWORKING=yes
HOSTNAME=localhost.localdomain
NOZEROCONF=yes
```

##### 변경후

```
NETWORKING=yes
HOSTNAME=freelec-springboot2-webservice
NOZEROCONF=yes
```

- 변경한 후 다음 명령어로 서버 재부팅

```
sudo reboot
```

- 재부팅이 끝나고 다시 접속해 보면 HOSTNAME이 잘 변경된 것 확인가능

### 3.1 Hostname이 등록되었다면 한 가지 작업을 더 해야 한다.

- 호스트 주소를 찾을 때 가장 먼저 검색해 보는 /etc/hosts에 변경한 hostname을 등록한다.
- 다음 명령어로 /etc/hosts 파일 열기

```
  sudo vim /etc/hosts
```

- 아래의 등록글을 화면에 등록한다

```
  127.0.0.1 등록한 HOSTNAME
```

```
  127.0.0.1 localhost localhost.localdomain ..(생략)
  ::1       local ..(생략)..

  127.0.0.1 freelec-springboot2-webservice
```

- :wq 명령어로 저장하고 종료한 뒤 정상적으로 등록되었는지 확인해 봅니다. 확인 방법은 다음 명령어로 확인. 만약 잘못 등록 되었다면 찾을 수 없는 주소라는 에러 발생

```
curl 등록한 호스트 이름
```

만약 잘못 등록되었다면 ?

```
could not resolve host ~
(생략)
```

잘 등록되었다면 다음과 같이 80 포트로 접근이 안된다는 에러가 발생

```
Failed to connect to freelec-springboot2-webservice port 80:
```

이는 아직 80포트로 실행된 서비스가 없음을 의미한다. 즉 curl 호스트 이름으로 실행은 잘 되었음을 의미.

참고 : 스프링 부트와 AWS로 혼자 구현하는 웹 서비스 - 이동욱
