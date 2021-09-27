# Gatsby 로 정적 블로그 생성(+Netlify로 배포)

- (윈도우 버전임)
- 참고 글
  - https://www.gatsbyjs.com/docs/tutorial/part-0/
  - https://velog.io/@hwang-eunji/create-github-blog-feat.-gatsby
  - https://appear.github.io/2019/03/10/ETC/gatsby-01/
  - https://delivan.dev/web/start-gatsby-blog/
  - https://blog.outsider.ne.kr/1417
  - https://blueshw.github.io/2018/10/07/hexo_to_gatsby/

## Gatsby 프로젝트 생성

- node.js 설치

  - Gatsby 는 node.js로 만들었기 때문에 설치 필요
  - https://nodejs.org/ko/
  - .msi 파일 다운 받아서 설치(기본으로 진행)
  - 설치 후 cmd창에서 확인

  ```
  node --version
  npm --version
  ```

- Git 설치

  - 이미 설치되어있으면 생략

- gatsby-cli 설치
  - gatsby 프로젝트 생성을 하기 위해서는 gatsby-cli 를 설치해야 한다
  ```
  npm install -g gatsby-cli
  ```
- 스타터 연결 및 dev-server 켜기

  - 프로젝트를 생성할 폴더를 먼저 만든다
    - C:/gatsby
  - 해당 프로젝트(gatsby)에 이동해서 blog(임의 이름) 폴더로 github 스타터 주소 연결해서 clone하기

    ```
    (C:\gatsby)로 이동 후
    gatsby new blog https://github.com/Highjune/gatsby-starter-bee
    ```

    - gatsby new - 명령어 게츠비 프로젝트생성을 위한 명령어
    - blog - 내컴퓨터 디렉토리명을 지정
    - https://github.com/Highjune/gatsby-starter-bee 는 내가 선택한 템플릿(해당하는 스타터) 주소를 fork해온 나의 repository 주소

  - C:\gatsby\blog로 이동 후 서버 on

  ```
  cd blog (현재 위치 C:\gatsby\blog)
  gatsby develop
  ```

  - 확인

  ```
  http://localhost:8080/  또는
  http://localhost:8000/___graphql
  ```

- 깃허브에 repository 만들고 push
  - 만든 gatsby 프로젝트를 올릴 "gatsby-blog" 라는 repository 생성
  - C:\gatsby\blog 위치에서 아래 명령어 실행
  ```
  $ git init
  $ git add .
  $ git commit -m "gatsby blog setup"
  $ git remote add origin [나의 "gatsby-blog" repository url]
  $ git push origin master
  ```

## Netlify 로 배포

- https://app.netlify.com/teams/highjune/sites
- 위 접속
- new site from git 클릭 → githup 클릭
- only select repository 클릭 → 해당 repository 선택(gatsby-blog) → deploy site
- 배포 완료되면 도메인 주소를 할당해준다.
