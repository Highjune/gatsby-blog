---
title: 'git stash'
date: 2023-01-18 22:37:00
category: 'git'
draft: false
---

## stash. 작업 중 다른 브랜치로 checkout 해야 할 경우
- 완료하지 않은 일을 커밋하고 checkout 하기에는 애매한 경우. stash
- stash할 때와 같은 브랜치에 적용해야 하는 것 아니다. A 브런치에서 stash 한 것을 다른 B 브런치에서 stash 를 복원해도 된다.
- 꼭 깨끗한 워킹 디렉터리가 아니어도 된다.
- 워킹 디렉토리에서 수정한 파일들만 저장(즉, 추적중인 파일만 저장)
  - Modified이면서 Tracked 상태인 파일과 Staging Area에 있는 파일들을 보관해두는 장소다. (새로 생성한 파일은 X)
  ```
  git stash
  git stash save
  ```
  - 추적 중이지 않은 파일을 같이 저장하려면.(아래 두 명령어는 같다.)
  ```
  git stash -u 
  git stash --include-untracked
  ```
- stash에 저장한 파일 확인
```
git stash list
```
- stash 에 저장한 파일을 복원
  ```
  git stash pop
  git stash apply
  ```
  - staged 에 있던 파일들을 stash 에 담은 후 다시 그대로(위 명령어) 꺼내면 unstaged가 된다. 만약 staged 상태를 유지하고 싶다면 --index
  ```
  git stash apply --index
  ```
- stash 에 저장한 파일들 중 일부분만 복원
  - 우선 list 확인해서 제일 앞에 stash@{n} 으로 숫자 나와있음
  ```
  git stash list
  ```
  - 해당하는 것만 복원 
  ```
  git stash apply stash@{n}
  ```
- staging area에 들어있는 파일들 제외 stash 에 넣기
  - 많은 파일들을 변경했지만 몇몇 파일만 커밋하고 나머지 파일은 나중에 처리하고 싶을 때 유용하다. 
  ```
  git stash --keep-index
  ```

- stash 제거
  - git stash apply 는 단순히 stash를 적용하는 것뿐이다. stash는 여전히 스택에 남아 있다.
  ```
  git stash drop
  ```
  - 참고로 git stash pop은 stash를 적용하고 나서 바로 스택에서 제거해준다.
- stash를 적용한 브랜치 만들기
  - 보통 stash 에 저장하면 한동안 그대로 유지한 채로 그 브랜치에서 계속 새로운 일을 한다. 그러면 이제 저장한 Stash를 적용하는 것이 문제가 된다. 수정한 파일에 stash를 적용하면 충돌이 일어날 수도 있고 그러면 또 충돌을 해결해야 한다. 필요한 것은 stash한 것을 다시 테스트하는 것. 아래 명령어를 사용하면 stash할 당시의 커밋을 checkout한 후 새로운 브랜치를 만들고 여기에 적용한다. 이 모든 것이 성공하면 stash를 삭제한다.
  - 브랜치를 새로 만들고 stash를 복원해주는 매우 편리한 도구다.
  ```
  git stash branch testchanges
  ```
- 워킹 디렉터리 청소하기
  - 작업하고 있는 파일들을 stash하지 않고 단순히 그 파일들을 치워버리고 싶을 때
  - 보통은 merge나 외부 도구가 만들어낸 파일을 지우거나 이전 빌드 작업으로 생성된 각종 파일을 지우는데 필요하다.
  - 신중해야 하는 명령어. 이 명령을 사용하면 워킹 디렉터리의 추적하고 있지 않은 모든 파일이 지워지기 때문. 복구 불가능. `git stash -all` 명령을 이용하면 지우는 건 똑같지만, 먼저 모든 파일을 stash 하므로 좀 더 안전하다. 
  - 추적 중이지 않은 정보를 워킹 디렉터리에서 지우고 싶다면. 하위 디렉터리까지 모두 지움. `-f` 옵션은 강제(force)의 의미.
  ```
  git clean -f -d
  ```
  - 명령 실행 시 결과를 미리 보고 싶다면 `-n` 옵션
  ```
  git clean -d -n
  ```
  - 무시된 파일까지 함께 지우려면 `-x` 옵션. 원래 .gitignore에 명시했거나 무시되는 파일은 지우지 않는다.
  ```
  git clean -n -d
  ```