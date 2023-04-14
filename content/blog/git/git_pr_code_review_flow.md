---
title: 'pull request 통한 리뷰 과정'
date: 2023-04-14 21:14:00
category: 'git'
draft: false
---

- NextStep 강의를 들으며 오픈소스에 기여하는 방식의 git flow 를 공부했고, 간단하지만 정리해본다.
- [NextStep git docs](https://github.com/next-step/nextstep-docs/tree/master/codereview)

## 1단계
> 미션을 시작, 개발 환경을 구축, 1단계 미션 완료, push를 보내는 단계.
- [github docs](https://github.com/next-step/nextstep-docs/blob/master/codereview/review-step1.md)
- [영상](https://www.youtube.com/watch?v=YkgBUt7zG5k)

1. 미션 저장소에 나의 github 계정(highjune) 으로 된 highjune 브랜치가 생성. (next step 에서 초기 단계에 자동생성)
2. 프로젝트를 자신의 계정으로 fork 한다. 
- next-step 저장소는 권한이 없으므로 미션을 진행한 코드를 추가할 수 없다.
- fork는 next-step 의 저장소를 자신의 계정으로 복사하는 기능. 모든 미션은 자신의 계정 아래에 있는 저장소를 활용해 진행. 

3. fork한 저자소를 자신의 컴퓨터로 clone한 후 디렉터리로 이동한다.
- fork 한 저장소는 github.com에 존재하기 때문에 바로 작업할 수 있다.
- clone 명령은 github.com에 존쟇나는 저장소를 자신의 노트북 또는 pc로 복사하는 과정.
- 터미널에 아래 명령어 입력
```
git clone -b {본인_아이디} --single-branch https://github.com/{본인_아이디}/{저장소 아이디}
ex) git clone -b highjune --single-branch https://github.com/highjune/java-racingcar
```
- clone한 폴더로 이동하는 방법
```
cd {저장소 아이디}
ex) cd java-racingcar
```
- 만약 fork시 `copy the main branch only` 를 체크했다면 main브랜치만 받아오게 된다. 이 경우에는 github 저장소에서 나의 아이디(highjune)로 branch를 생성해준다.
- 현재 터미널에서 `git remote -v` 라고 치면 아래와 같이 뜸
```
origin  https://github.com/highjune/java-racingcar (fetch)
origin  https://github.com/highjune/java-racingcar (push)
```

4. 기능 구현을 위한 브랜치 생성
- git은 서로 다른 작업을 하기 위한 별도의 공간을 생성할 때 브랜치를 생성할 수 있다.(실무 연습용 & 리뷰 받기 위한 것)
- 터미널 명령
```
git checkout -b 브랜치이름
ex) git checkout -b step1
```

5. IDE(intelliJ) 로 import

6. 해당 브런치(step1)로 기능구현

7. 기능 구현 후 add, commit
- 기능 구현을 완료한 후 로컬 저장소에 변경된 부분을 반영하기 위해 add, commit 명령을 사용한다.
```
git status // 변경된 파일 확인
git add -A(또는 .) // 변경된 전체 파일을 한번에 반영
git commit -m "메시지" // 작업한 내용을 메시지에 기록
```

8. 본인 원격 저장소에 올리기
- 로컬에서 commit 명령을 실행하면 로컬 저장소에만 반영되고, 원격 github.com의 저장소에는 반영되지 않는다.
```
git push origin 브랜치이름
ex) git push origin step1
```

## 2단계
> pull request를 통해 코드 리뷰 요청을 한 후 피드백을 받고, 피드백을 반영하는 과정
- [github docs](https://github.com/next-step/nextstep-docs/blob/master/codereview/review-step2.md)
- [영상](https://www.youtube.com/watch?v=HnTdFJd0PtU)
1. github 서비스에서 pull request를 보낸다.
- pull request는 github에서 제공하는 기능으로 코드리뷰 요청을 보낼 때 사용한다.
- pull request는 original 저장소(next-step의 저장소)의 브랜치(자신의 아이디 = highjune)와 앞 단계에서 생성한 브랜치 이름(앞 단계의 예에서는 step1)을 기준으로 한다.
```
ex) 미션을 진행한 javajigi/java-racingcar step1 브랜치 => nextstep/java-racingcar javajigi 브랜치로 pull request를 보낸다.
```
- 브라우저에서 github 저장소에 접근한다.
- 브랜치를 작업 브랜치로 변경한다(앞 단계의 예에서는 step1).
- 브랜치 오른쪽에 있는 "New pull request" 버튼을 클릭한다.
- 왼쪽 next-step 저장소의 브랜치를 자신의 github 계정 브랜치로 변경한다.
- 현재 미션에서 작업한 내용을 입력하고 "Create pull request" 버튼을 클릭해 pull request를 보낸다.

2. pull request를 보낸 후 리뷰어에게 리뷰 요청
- nextstep에서는 특정 버튼이지만(slack 알림도 감), slack 같은 협업툴이라면 해당 링크 보내면서 `리뷰 부탁드립니다.` 요청

3. pull request에 대해 승인이 되지 않고 수정 요청 피드백을 받으면 피드백 받은 내용을 반영한다. 만약, pull request가 승인이 되어 next-step 저장소에 통합(merge)이 된다면 코드리뷰 요청 3단계를 진행한다.

4. 피드백을 반영한 후 add, commit, push 명령을 실행한다.
- pull request를 보내 피드백을 받은 후 add, commit, push를 한 후 새로운 pull request를 보내지 않아도 된다.
- 앞서 보낸 pull request가 통합(merge)되지 않은 상태이기 때문에 같은 pull request를 재활용한다.
```
git status // 변경된 파일 확인
git add -A(또는 .) // 변경된 전체 파일을 한번에 반영
git commit -m "메시지" // 작업한 내용을 메시지에 기록
```
```
git push origin 브랜치이름
ex) git push origin step1
```
5. 몇 번의 피드백을 주고 받은 후 승인이 되어 next-step 저장소에 통합(merge)이 된다면 코드리뷰 요청 3단계를 진행한다.


## 3단계
> 리뷰 요청을 보낸 후 pull request가 next-step으로 통합(merge)된 이후의 과정
- [github docs](https://github.com/next-step/nextstep-docs/blob/master/codereview/review-step3.md)
1. merge를 완료했다는 통보를 받으면 브랜치 변경 및 작업 브랜치 삭제(option)한다.
- 강사에게 승인 후 merge를 완료했다는 통보를 받으면 해당 미션은 완료한 상태가 된다.
- 현재 미션을 완료했기 때문에 미션을 진행한 브랜치를 삭제하고 다음 미션을 위한 새로운 브랜치를 생성해야 한다.
```
git checkout 본인_아이디
git branch -D 삭제할_브랜치이름
ex) git checkout highjune
ex) git branch -D step1
```

2. 통합(merge)한 next-step 저장소와 동기화하기 위해 next-step 저장소 추가(최초 한번만)
- 계정 브랜치에서 다음 미션을 이어서 진행하기 위해 브랜치를 생성하려고 한다.
- 그런데 로컬 PC의 현재 상태는 최신 코드가 아니기 때문에 미션을 이어서 진행할 수 없다. 따라서 next-step에 통합(merge)된 코드와 동기화하는 작업을 진행해야 한다.
- `git remote add-` 명령은 최초 1회만 진행하면 된다.
```
git remote add {저장소_별칭} base_저장소_url
ex) git remote add upstream https://github.com/next-step/java-racingcar.git
// 위와 같이 next-step 저장소를 추가한 후 전체 remote 저장소 목록을 본다.
git remote -v
```
- `git remote -v` 의 결과는.
```
origin  https://github.com/highjune/java-racingcar (fetch)
origin  https://github.com/highjune/java-racingcar (push)
upstream        https://github.com/next-step/java-racingcar.git (fetch)
upstream        https://github.com/next-step/java-racingcar.git (push)
```

3. next-step 저장소에서 자기 브랜치 가져오기(또는 갱신하기)
- 앞 단계의 remote add 명령은 로컬 PC에서 next-step 저장소에 접근할 수 있도록 이름을 부여한 것이다. 앞 단계의 예제는 upstream이라는 이름을 부여했다.
- 앞 단계에서 next-step 저장소에 이름을 부여했다면 이번 단계는 fetch 명령으로 동기화하고 싶은 next-step 저장소의 브랜치를 가져오기 해야 한다.
```
git fetch upstream {본인 아이디}
ex) git fetch upstream highjune
```
- fetch 명령을 실행한 후 git branch -a 명령을 실행하면 remotes/upstream/javajigi와 같은 브랜치가 생성된 것을 확인할 수 있다.

4. next-step 저장소 브랜치와 동기화하기
- 현재 상태는 next-step 저장소 브랜치를 가져오기는 했지만 아직까지 로컬 저장소에 최신 버전의 코드가 반영된 것은 아니다.
- rebase 명령을 실행해 next-step 저장소와 로컬 저장소의 브랜치를 동기화한다.
```
git rebase upstream/본인_아이디
ex) git rebase upstream/highjune
```

5. 새로운 미션을 진행하기 위한 브랜치 생성
- 지금까지 과정을 통해 새로운 작업을 시작하기 위한 준비 작업을 마쳤다.
- 다음 단계는 코드리뷰 요청 1단계의 4번 항목의 checkout 명령으로 새로운 브랜치를 생성한다
```
git checkout -b 브랜치이름
ex) git checkout -b step2
```
- 이렇게 다음 단계(step2) 또한 끝나면 코드리뷰 요청 1단계의 7번 이후의 add, commit, push를 진행하고, 다시 pull request를 보내면 된다.




## 질문
- PR에서 충돌이 발생한다면 어떻게 해결하면 좋을까?
- Q) next-step 저장소와 rebase를 하지 않은 상태에서 브랜치를 생성했더니 충돌이 발생하고 있어요. 충돌을 해결하려면 어떻게 해야 하나요?
```
git checkout 본인_아이디(예: git checkout javajigi) 명령을 실행해 계정 브랜치로 이동한다.
git reset --hard upstream/본인_아이디(예: git reset --hard upstream/javajigi)
git checkout 기능_브랜치(예: git checkout step2)
git merge 본인_아이디(예: git merge javajigi)
```
- 위 명령어를 실행하면 충돌이 발생할 것. 충돌을 해결한 후 add, commit, push 진행하면 pr 충돌이 해결되어 리뷰 요청 가능.

