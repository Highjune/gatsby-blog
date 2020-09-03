---
title: '(AWS시리즈4) EC2 서버에 접속하기'
date: 2020-09-03 20:00:00
category: 'aws'
draft: false
---

## EC2 서버에 접속하기

- 생성한 EC2로 접속하는 것
- Mac & Linux 2가지 방법이 존재한다

### Max & Linux 로 EC2 서버에 접속방법

- 터미널에 아래 명령어 입력

```
ssh -i pem 키위치 EC2의 탄력적 IP 주소
```

- 다운받은 키페어 pem 파일을 ~/.ssh/로 복사
  - ~/.ssh/ 디렉토리로 pem 파일을 옮겨 놓으면 ssh 실행 시 pem 키 파일을 자동으로 읽어 접속을 진행
  - 이후부터는 별도로 pem 키 위치를 명령어로 지정할 필요가 없게 된다.

```
cd pem 키를 내려받은 위치 ~/.ssh/
```

- pem 키가 잘 복사되었는지 ~/.ssh 디렉토리로 이동해서 파일 목록 확인하기

```
cd ~/.ssh/
ll
```

- 복사되었으면 pem 키의 권한을 변경

```
chmod 600 ~/.ssh/pem키이름
```

- 권한 변경했으면 pem 키가 있는 ~/.ssh 디렉토리에 config 파일을 생성

```
vim ~/.ssh/config
```

- Host 등록
  - Host는 앞으로 접속할 키 값임 ex)
    - ex) Host abc로 등록하면 ssh abc로 해당 EC로 접속할 수 있다.
    - Host 외에 HostName은 탄력적 IP 주소를 사용하면 된다

```
Host 본인이 원하는 서비스명
    HostName ec2의 탄력적IP 주소
    User ec2-user
    IdentityFile ~/.ssh/pem키 이름
```

- 작성 끝났으니 :wq 명령어로 저장 종료

```
:wq
```

- 생성된 config 파일은 실행 권한이 필요하므로 권한 설정을 다음과 같이 설정

```
chmod 700 ~/.ssh/config
```

- 실행 권한까지 설정했다면 다음의 명령어로 접속

```
ssh config에 등록한 서비스명
```

- 'yes'를 입력하면 EC2에 접속이 성공한다

- 이제는 터미널에서 ssh 서비스명만 입력하면 쉽게 접속할 수 있다.

### 윈도우로 EC2 서버에 접속방법

- 윈도우에서는 Mac과 같이 ssh 접속하기엔 불편한 점이 많아 별도의 클라이언트 `putty` 를 설치한다.

- [putty 사이트](http://www.putty.org/) 에 접속하여 실행 파일 다운받기

- 실행 파일은 2가지다. 2가지 다 다운로드

  - putty.exe
  - puttygen.exe

- pem 키를 ppk 파일로 변환

  - putty는 pem 키로 사용이 안되며 pem키를 ppk파일로 변환을 해야한다
  - puttygen은 이 과정을 진행해 주는 클라이언트
  - puttygen.exe 파일 실행
  - puttygen 화면에서 상단 `Conversions` -> `Import key` 를 선택해서 내려받은 pem 키를 선택
  - 그러면 자동으로 변환이 진행
  - `Save private key` 버튼을 클릭해서 ppk 파일 생성
  - 경고 메시지가 뜨면 `예`
  - ppk 파일이 저장될 위치와 ppk 이름을 등록

- 접속

  - putty.exe 파일 실행
  - Host name : `ec2-user@탄력적IP주소` 등록
    - username@public.ip를 등록
    - Amazone Linux 는 ec2-user가 username이라서 ec-2user@탄력적 IP 주소를 등록
  - Port : ssh 접속 포트인 `22`를 등록
  - Connection Type : SSH 를 선택
  - 항목들을 다 채웠으면 왼쪽 사이드바에서 `Connection` -> `SSH` -> `Auth` 를 차례로 클릭해서 ppk 파일을 로드할 수 있는 화면으로 이동
  - `Browse...` 버튼 클릭
  - 이전에 생성한 ppk파일 선택해서 불러오기
  - 불러온 후 `Session` 탭으로 이동해서 `Saved Sessions` 에 현재 설정들을 저장할 이름을 등록하고 `Save` 버튼을 클릭
  - 저자안 뒤 `Open` 버튼을 클릭한 후 SSH 접속 알림이 등장하면 `예` 클릭

참고 : 스프링 부트와 AWS로 혼자 구현하는 웹 서비스
