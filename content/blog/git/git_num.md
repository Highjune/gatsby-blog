---
title: 'git num'
date: 2023-07-15 15:04:00
category: 'git'
draft: false
---

> GUI가 아닌 CLI 로 git을 다룰 때 더 편리한 기능 추가

# 현재의 불편함
- `git status` 명령어 후에 `git add`와  `git restore` 명령어로 `staging area` 에 추가, 제외할 때에는 해당 파일의 전체 경로를 입력해줘야 한다. 꽤 불편하다.
- 물론 `git add .` 명령어로 모든 파일을 한번에 다 추가할 수는 있지만 개별파일들을 찍어서 올리고 싶을 수 있기 때문.

```
git status
```

<img width="455" alt="image" src="https://github.com/Highjune/TIL/assets/57219160/17ccbb16-3a6c-42a8-8f20-e1e0afdf0e92">

```
git add content/blog/git/git_num.md
git status
```

<img width="458" alt="image" src="https://github.com/Highjune/TIL/assets/57219160/a79b7572-9b57-40bf-99d2-1ac88653473c">

# git num의 편리함
- [깃허브 링크](https://github.com/schreifels/git-num)
- 위에서 누가 `git num` 명령어를 통해 숫자로 간단히 파일을 `staging area` 에 추가할 수 있도록 만들었다.

<img width="453" alt="image" src="https://github.com/Highjune/TIL/assets/57219160/eb29d491-ccb5-42aa-a480-e1517ba663f7">

- 심지어 여러 파일들도 한번에 가능하다

<img width="458" alt="image" src="https://github.com/Highjune/TIL/assets/57219160/ed01b26a-27d6-4a6f-9ef8-1a536afb11d9">


# 설치방법

## 1. 다운로드
[다운로드 링크](https://github.com/schreifels/git-num/releases) 에서 다운로드
    - 필자는 Mac OS 이고 `2.0.1 Source code(tar.gz)` 다운로드 후 압축 해제

## 2. 파일 실행 및 경로에 넣기
- `git-num` 실행파일 실행.(안해도 되는듯)
- `git-num`실행파일을 $path 안에 넣기
- 터미널에서 `echo $path` 명령어로 나오는 디렉토리에 넣으면 된다. 나는 `/usr/local/bin` 에 git-num 실행 파일 넣음

<img width="403" alt="image" src="https://github.com/Highjune/TIL/assets/57219160/22e11fd0-a5e0-4604-bd7d-074b7f2877ff">


## 3. git-num 파일 실행 권한 추가
- /usr/local/bin 에서 권한 추가.
```
chmod +x git-num
```
