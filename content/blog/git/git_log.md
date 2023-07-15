---
title: 'git log'
date: 2023-01-18 22:37:00
category: 'git'
draft: false
---
 
## git log, 커밋 조회하기
- `-n` 옵션
  - 최근 것들 중에서 n개만 보여준다.
```
git log -2
```
- `-p` 옵션
  - 각 커밋의 diff 결과 보여줌
  - 직접 diff를 실행한 것과 같은 결과를 출력하기 때문에 동료가 무엇을 커밋했는지 리뷰하고 빨리 조회하는데 유용하다.
```
git log -p -2
```
- `--stat` 옵션
  - 어떤 파일이 수정됐는지, 얼마나 만흔 파일이 변경됐는지, 또 얼마나 많은 라인을 추가, 삭제했는지 보여준다. 요약정보는 가장 뒤쪽에 보여준다.
  ```
  git log --stat
  ```
- `--pretty` 옵션
  - 히스토리 내용을 보여주는 형식을 선택할 수 있다.
  - oneline, short, full, fuller 순으로 갈수록 조금씩 가감해서 보여준다.
  ```
  git log --pretty=oneline
  ```
  - `format` 옵션도 붙일 수 있다.
    - 나만의 포맷으로 출력 가능. 특히 결과를 다른 프로그램으로 파싱하고자 할 때 유용.
    - Git을 새 버전으로 바꿔도 결과 포맷이 바꾸지 않는다.
      - %h : 짧은 길이 커밋 해시
      - %an : 저자 이름
      - %ar : 저자 상대적 시각
      - %s : 요약
    ```
    git log --pretty=format:"%h - %an, %ar : %s"
    ```
  - oneline 옵션과 format 옵션은 `--graph` 옵션과 함께 사용할 때 더 빛난다.
  ```
  git log --pretty=format:"%h %s" --graph
  ```

- `--abbrev-commit`. 짧고 중복되지 않는 해시값으로 로그 확인하기 
```
git log --abbrev-commit --pretty=oneline
```
- 커밋 조회 (아래 3개는 다 같음. 단, 짧은 해시값이 다른 커밋과 중복되지 않다고 가정)
```
git show 7b2a5f9937165a081bef7670a0c0d4b174106c72
git show 7b2a5f9937165a081b
git show 7b2a5f
```
- main 브런치가 7b2a5f9 를 가리키고 있으면 아래 명령어는 같다.
```
git show main
git show 7b2a5f9
```
- refLog 보여주기
  - Git은 자동으로 브랜치와 HEAD가 지난 몇 달 동안에 가리켰었던 커밋 모두 보여줌
  - Git은 브랜치가 가리키는 것이 달라질 때마다 그 정보를 임시 영역에 저장한다. 예전에 가리키던 것이 무엇인지 확인 가능
  - Reflog의 일은 모두 로컬의 일이므로 동료의 저장소에는 없다.
  ```
  git reflog
  ```
  - @{n} 규칙으로 HEAD가 5번 전에 가리켰던 것을 알 수 있다.
  ```
  git show HEAD@{5}
  ```
  - 시간도 사용가능
    - 이 명령은 Reflog에 남아있을 때만 조회할 수 있기 때문에 너무 오래된 커밋은 조회할 수 없다.
  ```
  git show master@{yesterday}
  ```
  - git reflog 결과를 git log 명령과 같은 형태로 볼 수 있다.
  ```
  git log -g 
  ```
- merge 하기 전 또는 push 하기 전에 무엇이 변경되었는지 확인해보기
  - double dot(..). 현재 experiment 브랜치에는 없고 main 브랜치에만 있는 커밋들 확인하기
  ```
  git log experiment.main
  ```
  - origin 저장소의 master 브랜치에는 없고 현재 checkout 중인 브랜치에만 있는 커밋 보여주기. checkout한 브런치가 master/origin이라면 아래 명령어가 보여주는 커밋이 push하면 서버에 전송될 커밋들이다.
  ```
  git log origin/master..HEAD
  ```
- 히스토리에서 언제 추가되거나 변경됐는지 찾아보기. 
  - `-S` 옵션. 해당 문자열이 추가된 커밋과 없어진 커밋만 검색할 수 있다.
  ```
  git log -S haha --oneline
  ```
- 라인 히스토리 검색 (매우 유용)
  - `-L` 옵션. 어떤 함수나 한 라인의 히스토리를 볼 수 있다.
  - 함수의 시작과 끝을 인식해서 함수에서 일어난 모든 히스토리를 함수가 처음 만들어진 때부터 Patch를 나열하여 보여준다. 
  - ex) zlib.c 파일에 있는 git_delfate_bound 함수의 모든 변경사항을 보기
  ```
  git log -L :git_deflate_bound:zlib.c
  ```