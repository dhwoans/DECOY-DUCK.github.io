---
layout: post
title: (블로그) githubpage로 블로그 제작
subtitle: 단 하루 만에 블로그를 제작합니다.
banner:
  image: https://i.pinimg.com/564x/27/f2/3b/27f23b058bdde17abf2b190eb7e14e9f.jpg
  opacity: 0.618
  background: "#000"
  heading_style: "font-size: 4.25em; font-weight: bold; "
author: 오재문
categories: web
tags: [blog,docker,Jekyll,github page]
comments: true

---

## 로컬 환경에서 jekyll 설치

![Untitled](https://user-images.githubusercontent.com/51963264/221788404-6f5b5f9a-9915-4d76-9aac-5b6332819c92.png)

웹페이지 구축이 완료된 후 블로그를 관리하면서 로컬 환경에서 먼저 수정사항을 확인한 후 깃 페이지에 적용하는 것이 관리 측면에서 유리할 것으로 판단되기 때문에  `jekyll`을 설치해 보겠습니다. 설치과정은 [공식 문서](https://jekyllrb-ko.github.io/docs/installation/)를 참고했습니다. 빠른 설치 방법도 존재하지만 `MacOS`의 경우 아래의 오류가 출력될 수 있으니 링크의 설치가이드를 따라 설치하는 것을 추천드립니다.

```bash
gem install bundler jekyll                         

Fetching bundler-2.4.7.gem
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /Library/Ruby/Gems/2.6.0 directory.
```

설치가 정상적으로 완료 했을 경우 url에 [http://localhost:4000](http://localhost:4000/) 입력시 아래와 같은 화면이 출력됩니다.

![Untitled](https://user-images.githubusercontent.com/51963264/221788675-3fb0602c-c865-475e-9253-668c68f0aaf2.png)

---

## 💡발생할수 있는 오류

이곳에서는 발생할 수 있는 오류에 대한 해결책을 다뤄 보겠습니다 `jekyll` 명령어가 성공적으로 실행된다면 이부분은 생략 가능합니다.

### 1.`rbenv global` 명령을 실행 후 ruby version이 바뀌지 않는 경우

```bash
$ which ruby
/usr/bin/ruby

$ rbenv global 3.2.1

$ ruby -v                                           
ruby 2.6.8p205 (2021-07-07 revision 67951) [universal.x86_64-darwin21]

$ rbenv versions
system
* 3.2.1 (set by /Users/decoy/.rbenv/version)
```

이 경우 환경변수 설정이 필요합니다.`rebenv init` 명령어를 실행하면 아래와 같이 출력됩니다. 문서편집도구를 이용해 `~/.zshrc` 파일에 `eval "$(rbenv init - zsh)"` 붙여 넣습니다.

```bash
$ rbenv init
# Load rbenv automatically by appending
# the following to ~/.zshrc:

eval "$(rbenv init - zsh)"

$ vim ~/.zshrc
```

---

### 2.`gem env` 가 정상적으로 출력됨에도 jekyll 명령 실행시 오류 출력

```bash
$ jekyll new myblog                                  
zsh: command not found: jekyll
```

아래 명령어를 통해 해결할 수 있습니다.

```bash
$ gem install -n /usr/local/bin jekyll
```

---

## 웹 페이지 구성

이제까지 블로그를 운영하기 위한 기반을 다졌다면 이제 실제 블로그 페이지를 만들어 볼 차례 입니다. 정확히 말하면 템플릿을 활용해 웹페이지를 만드는 것입니다. 직접 만들어도 좋지만 시간과 노력은 한정적이기 때문에 이미 잘 만들어진 웹 페이지를 잘 활용하는 것이 더 효과적일 수 있습니다. 

`jekyll` 테마를 적용할 수 있는 방법은 다양합니다.  저는 로컬환경에 소스코드를 다운받는 방법으로 진행하겠습니다.  테마는  [이곳](https://jekyll-themes.com/free/)을 통해 구할수 있습니다. 그 외 `jekyll theme`로 검색하면 다양한 테마를 무료로 사용할 수 있습니다. 

![Untitled](https://user-images.githubusercontent.com/51963264/221790282-0dd82015-618c-41ab-96bd-75b02d82b91f.png)

대부분 깃허브를 통해 소스코드를 제공하고 있습니다. 마음에 드는 테마의 코드를 다운받습니다. 적당한 위치에 압축을 풀고 아래 명령어를 실행합니다. 

```bash
bundle

bundle exec jekyll serve
```

정상적으로 실행 되었다면 다음과 같이 테마가 적용된 웹페이지를 보실 수 있습니다. 

![Untitled](https://user-images.githubusercontent.com/51963264/221790553-1f59777f-8929-4596-a1d1-7162cdb66042.png)

로컬환경 설정은 이로써 마무리 되었습니다.  이제 호스트 서버를 만들고 로컬 환경에서 완료한 작업을 실제 서버에 배포하는 작업을 진행해 보겠습니다.

---

## github page 웹호스팅


![Untitled](https://user-images.githubusercontent.com/51963264/221825227-14868773-fcc8-4f84-ba93-263b7da2b287.png)


로컬 환경을 마쳤다면 이제 배포서버를 만들 차례입니다. 깃페이지의 경우 jekyll이 설치되어 있기때문에 별도의 설치작업이 필요하지 않습니다. 그럼 이제 원격 서버를 만들고 로컬의 웹페이지를 원격서버에 띄워보겠습니다. 이번 작업을 원활히 진행하기 위해서는 git 명령어 및 github에 대한 전반적인 지식이 요구됩니다.


작업 순서는 [이곳](https://docs.github.com/ko/pages/getting-started-with-github-pages/creating-a-github-pages-site)을 참고하여 진행하였습니다.

github Page를 생성하기 위해 자신의 계정에 새로운 repo를 생성합니다. 이때 새로운 repo의 이름은 `{user-name}.github.io`  또는 `{organization}.github.io` 형식으로 만들어야 합니다.

![Untitled](https://user-images.githubusercontent.com/51963264/221826324-b325aef8-f19e-4d8b-8508-5539e02fa8e1.png)

repo 생성후 실제로 웹 페이지가 출력되는지 확인하기 위해 방금 생성한 저장소에 ‘Hello, World!’가 출력되는 index.html 파일을 올린 뒤 url 주소창에 {user-name}.github.io를 입력해 보겠습니다.

![Untitled](https://user-images.githubusercontent.com/51963264/221826844-f29b09d7-d90e-43be-bd14-65aae71c8b2a.png)

위와 같이 정상적으로 ‘Hello,world!’가 출력된 것을 확인했다면 다음 단계인 커스텀 도메인 등록로 넘어가겠습니다.


---

## 커스텀 도메인 등록

다음으로 커스텀 도메인을 등록해 보겠습니다. 커스텀 도메인은 이름처럼 기본으로 제공되는 {user-name}.github.io가 아닌 도메인 공급업체로부터 구매한  주소를 사용하는 것을 말합니다. 거의 유일하게 과금요소가 있는 부분이지만 공급업체에 따라 무료이거나 합리적인 가격으로 구매가 가능합니다. 

커스텀 도메인 등록 순서는 다음과 같습니다.

1. 커스텀 도메인을 구매합니다.

1. 도메인 공급자 페이지에서 깃허브 페이지를 연결한다.
2. github page 설정에서 커스텀 도메인을 등록한다.

하나씩 순서대로 살펴보겠습니다. 우선 사용하고 싶은 도메인을 도메인 공급업체로 부터 구매를 해야 합니다. 많은 도메인 업체가 있고 가격을 비교한 후 구매하는 것은 어렵지 않을 것입니다. 중요한 것은 구매후 DNS 레코드를 설정해야한다는 것입니다. 

이 작업은 도메인 공급업체에 github page의 주소를 알려주고 github page에 구입한 도메인 주소를 알려주는 것입니다. Github에서는 Github Pages에서 Custom Domain을 사용할 경우에 DNS 설정을 아래와 같이 반영할 것을 요구하고 있습니다.

![Untitled](https://user-images.githubusercontent.com/51963264/221827069-f3b4502c-8a1e-47b9-b1ba-d50fecb24518.png)

`CNAME` 은 별칭을 지정하는 레코드입니다. {user-name}.github.io 입력해도 구입한 도메인 주소로 접속할 수 있도록 합니다.

`A` 는 IP 주소와 연결시켜 줍니다. 위의 IP는 Github Page의 Static Page를 관리하는 IP입니다.

![Untitled](https://user-images.githubusercontent.com/51963264/221831920-224cae02-d372-464b-b8d8-d367834ad9a8.png)

마지막으로 github에 구입한 도메인을 커스텀 도메인에 등록하면 커스텀 도메인 등록이 끝납니다. github settings>pages>custom domain 항목에 구입한 도메인을 적고 save를 누르면 작업이 끝납니다. 시간이 10분 정도 요소될 수 있으니 만약 정상적으로 작동되지 않더라도 조금만 기다리면 해결될 수 있습니다.

![Untitled](https://user-images.githubusercontent.com/51963264/221827214-4e81b891-16df-4422-bec0-703f80412523.png)

웹페이지를 띄울 서버가 완성되었습니다. 이제 웹페이지를 서버로 옮기면 블로그를 운영할 사이트가 완성됩니다.

---

## 로컬 동기화

원격 서버가 적상적으로 작동되는 것을 확인 했으니 로컬환경에서 다운 받은 웹 페이지를 서버로 보내면 블로그를 운영하기 위한 준비는 끝났습니다. 

웹페이지를 서버로 보내기 위해 우리가 해야하는 것은 정말 단순합니다. 바로 git을 이용하는 것 뿐입니다. 그 외에 어떠한 설치나 설정작업도 필요하지 않습니다.

로컬환경에 {user-name}.github.io 저장소와 연결할 디렉토리를 만든후 원격저장소와 연결합니다.

```Bash

git clone https://github.com/{user-name}/{user-name}.github.io.git

```

그 후 원격저장소에 저장한 index.html가 있다면 지운 후 다운받아 놓은 jekyll 테마를 디렉토리에 가져와 압축을 풉니다. git commit 및 push 명령어를 실행하여 원격 깃 저장소로 코드를 보냅니다. 그리고 브라우저를 통해 테마가 적용 되었는지 확인합니다.

![Untitled](https://user-images.githubusercontent.com/51963264/221827379-26463192-b4de-4290-86d4-258c4cd45fd6.png)

## docker를 이용한 개발 환경 구축

만약 다른  컴퓨터에서 위와 같은 웹페이지를 보기 위해서 개발 환경을 구성해야 한다면 당연히 위와 동일한 작업을 다시 진행 해야 합니다.심지어  오류를 최대한 발생시키지 않게 하기 위해 특정 버전을 맞춰야 할지도 모릅니다.

하지만 완전 동일한 프로세스로 진행했다하더라도 오류가 발생하지 않는다는 것을 보장할 수 없습니다. 이런 처리를 위해  또 시간과 노력이 할애된다면  시작하기도 전에 의지가 사라질 것입니다. 

![Untitled](https://user-images.githubusercontent.com/51963264/221789268-244603af-dbf5-41a2-a311-26d58c27ddf2.png)

 그래서 이 문제를 도커로 해결해보고자 합니다. 도커를 통해 어떤 환경에서라도 [localhost:4000](http://localhost:4000)에 웹페이지가 나오도록 하는 것이 목표입니다. 

![Untitled](https://user-images.githubusercontent.com/51963264/221789349-960936f0-4936-4727-8b8e-5c144bc39a1a.png)

먼저 도커,도커컴포즈가 설치된 환경이 필요 합니다. 이번에는 window 환경에서 개발 환경을 구성해보겠습니다.  꼭 window가 아닌 다른 OS환경이라도 좋습니다.

```bash
.
├── 404.html
├── CNAME
├── Gemfile
├── Gemfile.lock
├── LICENSE.txt
├── README.md
├── _config.yml
├── _data
├── _includes
├── _layouts
├── _posts
├── _sass
├── _site
├── about.html
├── archives.html
├── assets
├── categories.html
├── docker-compose.yml <----------
├── index.html

…
```

최상위 경로에 docker-compose.yml 파일을 생성합니다. 그리고 아래와 같이 저장합니다.

```bash
version: '3'

services:
  jekyll:
    image: jekyll/jekyll:latest
    command: jekyll serve 
    ports:
      - 4000:4000
    volumes:
      - .:/srv/jekyll
```

다음  터미널을 열어 명령어를 실행합니다.

```bash
docker compose up
```

정상적으로 실행된다면 성공입니다. 이제 도커가 설치된 환경이라면 다른 별도의 설치작업 없이 로컬 환경에서 웹페이지를 확인할 수 있게 되었습니다.

![Untitled](https://user-images.githubusercontent.com/51963264/221789433-8e2b6ffa-24fc-4c09-b0d5-b06950c9e0ae.png)

---
## 맺음말

짧은 시간에 웹사이트를 제작해 보았습니다. Jekyll을 이용해 웹페이지를 구성하였지만 Jekyll말고도 다양한 정적 웹페이지 생성기(SSG)가 있습니다. 블로그 관리에 익숙해진다면  저에게 있어 생소한 언어인 ruby보다 go나 JavaScript로 만들어진 SSG로 마이그레이션해보는 것도 좋은 경험이 될것 같습니다.