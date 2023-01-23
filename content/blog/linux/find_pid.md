---
title: '포트번호로 실행중인 프로세스 찾기'
date: 2023-01-23 20:09:00
category: 'linux'
draft: false
---
## 명령어
- 종종 쓰지만 깜빡한 명령어임
- 8080 포트번호로 실행중인 프로세스 찾기
```linux
lsof -i tcp:8080
```
- 만약 pid가 2125라면, 해당 pid로 프로세스 종료 시키기
```
sudo kill -9 2185
```

