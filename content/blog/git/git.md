---
title: 'git(pro git 정리한 것)'
date: 2023-01-17 14:38:00
category: 'git'
draft: false
---
- pro git 책 보고 정리한 것
- [깃허브 README.md](https://github.com/Highjune/git-test) 에 정리한 것


```
git help 명령어
git 명령어 --help
```
- 실전에서 자주 쓰이는 전략들은 다른 책 & 강의들을 통해서 공부해야 할 것 같다.
- 너무 세부적인 부분들은 넘겼다. 중요한 부분들은 다시 책 보면서 공부하기
- 여러 명령어 만지작 만지작


# 명령어 (기본적으로 git (명령어) --help 를 통해 직접 확인하는 습관을 가지자)

## Git 설정
- 총 3개의 설정 파일이 있다.
  - `/etc/gitconfig` 파일. Git이 제일 먼저 찾는 파일. 이 파일은 해당 시스템이 있는 모든 사용자와 모든 저장소에 적용되는 설정 파일. git config 명령에 `--system` 옵션을 주면 이 파일을 사용한다.
  - `~/.gitconfig` 은 해당 사용자에게만 적용되는 설정파일. `--global` 설정을 주면 이 파일을 사용한다.
  - `.git/config` 파일은 해당 저장소에만 적용된다.
  - 각 설정 파일에 중복된 설정이 있으면 "순서대로" 덮어쓴다.

- 설정값들 확인하기
  - alias 등 내가 설정한 값들을 확인할 수 있다. 
```
git config --list
```
- git alias 설정하기
  - 아래처럼 설정하면 ~/.gitconfig 파일에 저장된다.
  - 설정후에는 `git log` 명령어로만 편하게 사용 가능하다.
```
git config -global alias.lg log --oneline --decorate --graph --all
```

- core.editor 설정
  - 기본은 vim 사용
  ```
  git config --global core.editor emacs
  ```
- commit.template 설정
  - 만약 ~/.gitmessage.txt 라는 파일을 아래와 같이 만든다고 가정.
  ```
  subject line

  what happened
  ```
  - 템플릿 설정하기
  ```
  git config --global commit.template ~/.gitmessage.txt
  ```
  - 후에 git commit 명령어를 치면 편집기에 자동으로 템플릿 파일에 설정된 것을 채워준다.
- core.excludesfile
  - 한 저장소 안에서뿐 아니라(.gitignore 설정) 어디에서라도 git 에 포함되지 않은 파일을 설정할 수 있다.
  - 무시할 파일들을 설정할 수 있는데 ~/.gitignore_global 파일 안에 아래 내용 입력
  ```
  *~
  .DS_Store
  ```
  - 파일 설정에 추가
  ```
  git config --global core.excludesfile ~/.gitignore_global
  ```
- help.autocorrect
  - 잘 쓸 것 같진 않음.
  - 명령어를 잘못 입력하면 Git은 추측할 수 있는 메시지를 보여주긴 하지만 직접 실행하진 않는다. 그러나 help.autocorrect를 1로 설정하면 명령어를 잘못 입력해도 Git 이 자동을 해당 명령어를 찾아서 실행해준다.
  - 0.1 seconds 라는 것이 뜨는데, 사실 help.autocorrect 설정에 사용하는 값은 1/10초 단위의 숫자를 나타낸다. 만약 50이라는 값으로 설정한다면 자동으로 고친 명령을 실행할 때 Git은 5초간 명령을 실행하지 않고 기다려줄 수 있다.

## git diff, 두 트리 개체 차이 보여주기
- 워킹 디렉터리와 Staging Area 비교
  - 그래서 작업한 내용을 `git add` 명령어로 Staging Area로 옮기면 당연히 아무내용도 안 뜸
```
git diff
```
- Staging Area와 마지막 커밋 비교
```
git diff --staged
```
- 두 커밋 비교
```
git diff master branchB
```

## 파일 삭제
- git 에서 파일을 지우려면 `git rm`
  - 삭제한 파일은 staged 상태가 바로 되고, 커밋하면 파일은 삭제 & Git 이 추적하지 않음.
```
git rm test.txt
```
- 만약 그냥 `rm 파일명` 으로 지우게 된다면?
  - 지운 후 git status로 확인해보면 Unstaged 상태임
  ```
  rm test.txt
  git status
  ```
- `--cached` 옵션. Staging Area에서만 제거하고 워킹 디렉터리에 있는 파일은 지우지 않고 남겨두기
  - 실수했을 경우 매우 유용
    - `git add .` 명령어로 모든 컴파일된 .class 파일들을 staging area에 올렸을 경우.
  - 하드디스크에 있는 파일은 그대로 두고 Git만 추적하지 않게 한다.
  - 이것은 .gitignore 파일에 추가하는 것을 빼먹었거나, 대용량 로그 파일이나 컴파일된 .a 파일 같은 것을 실수로 추가했을 때 아주 유용.
  ```
  git rm --cached test.txt
  ```
  - 실수로 추가한 .class 파일들 staging area에서 내리기
  ```
  git rm --cached .class
  ```
  
- 여러 개의 파일이나 디렉터리를 한꺼번에 삭제하기. file-glob 패턴
  - log/ 디렉터리에 있는 .log 파일을 모두 삭제 
    - `*` 앞에는 \ 있음. 파일명 확장 기능은 셸에만 있는 것이 아니라 Git 자체에도 있으므로 필요함.
  ```
  git rm log/\a.log
  ```
  - ~로 끝나는 파일들 모두 삭제
  ```
  git rm \*~
  ```
  - 현재 밑의 testfolder 폴더에 있는 .class 파일들 staging area에서 내리기
  ```
  git rm --cached testfolder/\*.class
  ```

## 브런치 조작
- remote 서버(ex. origin)와 동기화
  - fetch는 서버로부터 데이터를 가져와서 저장해두고 사용자가 merge하도록 준비만 해둔다. 워킹 디렉토리의 파일 내용은 변경하지 않고 그대로 남는다. 
  - fetch 만 후에 git log 로 어떤 작업들을 했는지 확인해볼 수 있음. 그 후에 merge 
```
git fetch origin --all
git fetch --all 도 마찬가지
```
- 동기화 후 실제 merge (ex. origin 리모트 서버의 develop 브런치와 develop). 로컬에서 develop 으로 체크아웃한 후 실행하기.
```
git merge origin/develop
```
- fetch와 merge를 동시에
```
git pull origin master
```
- 리모트 트래킹 브랜치에서 시작하는 새 브랜치를 만들 때. 새로운 기능추가 작업을 할 때 release 브런치를 기반으로 생성한 후 다 작업이 끝나면 develop에 merge할 브런치 생성하는 것
```
git checkout -b feature/PROD-11111 origin/release
```

- 현재 추적 브런치가 어떻게 설정되어 있는가 (git fetch --all 명령어로 서버로부터 최신 데이터를 받아온 후에 추적 상황 확인하기)
```
git fetch --all
git branch -vv
```

- 리모트 브런치 삭제
  - 서버에서 브랜치(즉 커밋을 가리키는 포인터) 
```
git push origin --delete hotfixbranch
```

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

## 충돌해결
- (곧 정리..)

## 히스토리 단장하기
- 모든 것은 다른 사람과 코드를 공유하기 전에 커밋 히스토리를 예쁘게 단장해야 한다.
- 마지막 커밋 메세지 수정하기
```
git commit --amend
```
- 커밋 후에 새로 만든 파일이나 수정한 파일을 가장 최근 커밋에 집어 넣기.
  - 파일 수정후 git add로 staging area에 넣는다. 
  - 그 후에 git commit --amend 로 커밋 하면 된다. 이 명령은 Staging Area의 내용을 이용해서 수정한다.
  - 이 때 SHA-1 값이 바뀌므로 과거의 커밋을 변경할 때 주의해야 함. Rebase와 같이 이미 Push한 커밋은 수정하면 안된다.

- 커밋 메시지 여러 개 수정하기
  - `-i`옵션으로 대화형실행. HEAD~3 은 어느 시점부터 HEAD까지의 커밋 3개를 수정.
  ```
  git rebase -i HEAD~3
  ```
  - 그러면 각 커밋 앞에 'pick' 이 붙어 있는데 이것을 edit으로 수정후에.
  ```
  git commit --amend // 후에 커밋 메시지 수정후 텍스트 편집기 종료.
  git rebase --continue
  ```
  - 후에 나머지 두 개의 커밋에 적용하면 끝. 다른 것도 pick을 edit으로 수정해서 하면 된다. 

- 커밋 순서 바꾸기
  - 아래 명령어 실행 후, 한 라인을 지우거나 각각의 라인의 순서를 변경해도 된다.
  ```
  git rebase -i HEAD~3
  ```
- 커밋 합치기
  - 아래 명령어 실행 후 pick이나 edit 말고 squash 를 입력하면 git은 해당 커밋과 바로 이전 커밋을 합칠 것이고 커밋 메시지도 수정한다. 
  ```
  git rebase -i HEAD~3
  ```
  - 만약 3개의 커밋을 모두 합치려면 스크립트를 아래와 같이 수정한다.
  ```
  pick f7f3f6d changed my name a bit
  squash 310154e upated README formatting and added blame
  squash a5f4a0d added cat-file
  ```
  - 저장 후 편집기 종료 하면 git은 3개의 커밋 메시지를 merge할 수 있도록 에디터를 바로 실행해주고, 메시지 저장하면 3개의 커밋이 합쳐진 하나의 커밋만 남는다.

- 커밋 분리하기
  - 아래 명령어 실행시.
  ```
  git rebase -i HEAD~3
  ```
  - 아래와 같이 나왔을 때, 두 번째 커밋인 "updated README formatting and added blame"을 "updated README formatting"과 "added blame"으로 분리하는 것이다. 
  ```
  pick f7f3f6d changed my name a bit
  squash 310154e upated README formatting and added blame
  squash a5f4a0d added cat-file
  ```
  - 그러면 아래와 같이 변경
  ```
  pick f7f3f6d changed my name a bit
  edit 310154e upated README formatting and added blame
  pick a5f4a0d added cat-file
  ```
  - 저장후에, 명령 프롬프트로 넘어간 후 그 커밋을 해제하고 그 내용을 다시 두 개로 나눠서 커밋하면 된다. 저장하고 편집기를 종료하면 Git은 제일 오래된 커밋의 부모로 이동하고서 f7f3f6d과 310154e을 처리하고 콘솔 프롬프트를 보여준다. 여기서 커밋을 해제하는 `git reset HEAD^` 라는 명령으로 커밋을 해제 하면 수정했던 파일은 Unstaged 상태가 된다. 그 후에 파일을 Stage 한 후 커밋하는 일을 원하는 만큼 반복하고 나서 `git rebase --continue` 라는 명령을 실행하면 남은 Rebase 작업이 끝난다.
  ```
  git reset HEAD^
  git add README
  git commit -m "updated README formatting"
  git add newfile4.txt
  git commit -m "updated newfile4.txt
  git rebase --continue
  ```
## reset
- `HEAD`
  - 현재 브랜치를 가리키는 포인터.
  - 브랜치는 브랜치에 담긴 커밋 중 가장 마지막 커밋을 가리킨다. 
  - 지금의 HEAD가 가리키는 커밋은 바로 다음 커밋의 부모가 된다.
  - 즉, 마지막 커밋의 스냅샷이라고 할 수 있다.
- `index(Staging area)`
  - 바로 다음에 커밋할 것들.
  - Staging area 말하는 것. git commit 명령을 실행했을 때 Git이 처리할 것들이 있는 곳
  - 먼저 Index는 워킹 디렉터리에서 마지막으로 Checkout한 브랜치의 파일 목록과 피일 내용으로 채워짐. 이후 파일을 변경하고 변경한 내용으로 index를 업데이트할 수 있다. 이후 git commit 명령으로 index는 새 커밋으로 변환된다.
  - git commit 명령으로 index의 내용을 스냅샷으로 영구히 저장하고 그 스냅샷을 가리키는 커밋 객체를 만든다. 그리고 master가(현재 HEAD가 가리키고 있는 브런치를 master라고 가정) 그 커밋 객체를 가리키도록 한다.
- `워킹 디렉토리`
  - 실제 파일로 존재하는 곳
  - git add 명령어로 워킹 디렉토리의 내용을 index로 복사.
> 브랜치를 바꾸거나 Clone 명령도 내부에서는 비슷한 절차. 브랜치를 checkout하면, HEAD가 새로운 브랜치를 가리키도록 바구고, 새로운 커밋의 스냅샷을 Index에 놓는다. 그리고 Index의 내용을 워킹 디렉터리로 복사한다. 

- Reset은 크게 3가지 역할
1. HEAD 이동(--soft)
2. Index 업데이트 (--mixed)
3. 워킹 디렉토리 업데이트(--hard)
- HEAD 이동(--soft)
  - HEAD는 계속 현재 브랜치를 가리키고 있고, 현재 브랜치가 가리키는 커밋을 바꾼다.
  - `--soft` 옵션을 사용하면 Index나 워킹 디렉토리는 그대로 놔두고 브랜치가 가리키는 커밋만 이전으로 되돌린다. (HEAD~ 의 의미는 HEAD의 부모 커밋)
  ```
  git reset --soft HEAD~
  ```
- Index 업데이트(--mixed)
  - `--mixed` 옵션을 주면, HEAD를 이동시킨 후(1단계) index를 현재 HEAD가 가리키는 스냅샷으로 업데이트(2단계)를 한다.
  - 그리고 아무 옵션도 주지 않으면 --mixed 옵션으로 동작한다. (아래 두 명령어는 동일)
  ```
  git reset --mixed HEAD~
  git reset HEAD~
  ```

- 워킹 디렉터리 업데이트(--hard)
  - 매우 위험한 옵션.
  - `--hard` 옵션은 HEAD를 이동시킨 후(1단계) index를 현재 HEAD가 가리키는 스냅샷으로 업데이트(2단계)를 하고, 워킹 디렉토리까지 업데이트(3단계) 한다.
  ```
  git reset --hard HEAD~
  ```
  - git add와 git commit 명령으로 생성한 마지막 커밋을 되돌리고, 워킹 디렉터리의 내용까지도 되돌리는 것.
  - git 에서는 데이터를 실제로 삭제하는 방법이 없는데 이것은 그 중에 하나.

## checkout
- reset과 거의 동일. 
- 하지만 2가지가 다르다.
  1. 워킹 디렉토리를 안전하게 다룬다. 
    - 저장하지 않은 것이 있는지 확인해서 날려버리지 않는다는 것을 보장.
    - 워킹 디렉토리에서 작업을 한번 시도해보고 변경하지 않은 파일만 업데이트한다.
  2. HEAD 자체를 다른 브랜치로 옮긴다.