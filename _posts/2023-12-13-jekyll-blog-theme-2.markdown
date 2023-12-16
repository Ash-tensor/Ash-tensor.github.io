---
layout: post
comments: true
title: "[깃허브 블로그]지킬 블로그 테마 설치 및 오류 수정(clean blog theme)"
subtitle: Your SCSS file assets/main.scss has an error on line 2 File to import not found or unreadable 오류 해결
description: troubleshooting
date: 2023-12-13
categories: 깃허브
background: '/img/posts/06.jpg'
---


# 깃허브 블로그 지킬 테마 설지법(clean blog theme)

루비 및 gem, bundle은 이미 설치되어 있다고 가정한다. 만약 설치되어 있지 않다면 

    gem install jekyll bundler

를 터미널에서 실행해 주면 된다. macOs에는 이미 기본적으로 루비가 설치되어 있기 때문에 루비를 따로 설치할 필요는 없다. 혹시라도 윈도우인 경우에는 루비 공식 홈페이지에서 루비를 설치한 뒤, 번들러와 지킬을 설치해 주면 된다.

설치 방법은 두 가지가 있다. 

[Clean Blog 깃허브 주소](https://github.com/StartBootstrap/startbootstrap-clean-blog-jekyll)

저 깃허브 레포지토리의 readme.md의 방법대로 실행하는 법과, 깃허브 레포지토리를 클론해서 사용하는 방법 두 가지가 존재한다.

<img src="/img/posts/clean-blog-setup/01.png" style="width: 75%">

위 방법이 readme.md 방법대로 실행하는 방법인데, 위 방법은 순수한 테마를 다운받는 방법으로, 기본적인 설정이나 최초 이미지 같은 설정이 아무것도 되어 있지 않은 테마를 다운받는 방법이다. 
깔끔해서 나쁘지는 않지만, 결국 기초 뼈대가 되는 레이아웃 마크다운 파일을 직접 만들어 줘야 되고 커스터마이징 해줘야 되는건 똑같기 때문에 그냥 깃허브 레포지토리를 클론해서 사용하는 것을 추천한다. 그렇기에 여기서도 레포지토리를 클론해서 사용하는 방법으로 설명할 것이다.

## 설치 방법

1. 깃허브 페이지를 이용해 지킬 블로그를 배포할 정도라면 적어도 최소한 컴퓨터에 친숙한 사람이라고 생각은 하지만 설명하자면

    <img src="/img/posts/clean-blog-setup/02.png" style="width: 75%">

    여기서 New 버튼을 눌러서 새로운 깃허브 레포지토리를 만들고 위 사진처럼 자신의 깃허브 이름과 동일하도록 레포지토리를 만들면 된다.

### 깃허브 이름과 레포지토리 이름을 동일하게 만들어야 하는 이유

왜 깃허브 이름과 동일하도록 레포지토리를 만들어야 되냐고 물어볼 수도 있다. 여기에는 이유가 있는데 만약 자신의 이름과 다르게 레포지토리를 만들게 되면 설정이 조금 귀찮게 된다.

예를 들어서 레포지토리의 이름을 [tensor-studio] 라고 설정한다면

* "tensor-studio.github.io"

인터넷 상에 올라갈 주소가 이런 식으로 만들어 지는 것이 아니라

* "Ash-tensor/tensor-studio.github.io"

이런식으로 구성되게 된다. 이렇게 되면 문제가 생기는데 지킬 테마는 기본적으로 깃허브 이름과 레포지토리 이름이 동일할 것이라고 생각하고 제작된 경우가 있다. 그런 경우에는 당연하게도 오류가 생기게 되는데 pamalink와 baseurl등을 추가적으로 _config.yml에서 설정해 줘야 할 필요가 있다. 이런 경우에는 css 스타일이 다 날라가 버리거나, 테마의 기능이 제대로 기능하지 않는 경우가 있다. 그런 설정을 추가적으로 하고 싶지 않다면, 그냥 레포지토리의 이름과 깃허브 이름을 동일하게 만들어 주는 편이 낫다.


<img src="/img/posts/clean-blog-setup/03.png" style="width: 75%">

2. 설정을 Public으로 구성해 주고, 터미널로 들어가서 로컬에서 설치하고 싶은 위치에 당신의 레포지토리를 git clone 해준 뒤,
[여기서 Clean Blog를 git clone 하면 된다.](https://github.com/StartBootstrap/startbootstrap-clean-blog-jekyll) 혹시라도 처음인 사람을 위해 쓰자면

<img src="/img/posts/clean-blog-setup/04.png" style="width: 75%">

여기에서 download.zip을 하거나, 원하는 위치에

    git clone "https://github.com/StartBootstrap/startbootstrap-clean-blog-jekyll.git"

이렇게 클론하면 된다. 

clean-blog-setup에 있는 모든 파일을 복사한 뒤, 아까 만든 자신의 레포지토리에 복사해 넣고, 중복되는 파일은 '대치'를 선택한다.

3. 터미널에
        
        bundle install
        bundle exec jekyll serve

를 입력해 주면 지킬이 번들러를 통해 페이지를 빌드하기 시작한다(당연히 터미널의 위치는 당신의 레포지토리 위치여야 한다). 그 뒤, 자신의 github 레포지토리로 푸쉬하면 된다.

## _config.yml 및 깃허브 deploy 오류 해결

1. _config.yml 수정

<img src="/img/posts/clean-blog-setup/06.png" style="width: 75%">

_config.yml 파일을 자신에게 맞게 올바르게 수정하고(title, email, username 등) url을 자신 깃허브 주소에 맞게 수정한다. 예를들어서

    url: https://Ash-tensor.github.io
    https://자신 깃허브 아이디.github.io 포맷이다.

이런 식으로 수정하면 된다. https:// 부분을 빼놓고 작성하면 나중에 google search consol에 sitemap이 제대로 등록되지 않는 문제가 생긴다. 그러니 https:// 부분을 꼭 작성해 주도록 하자!

그리고 위 사진처럼 baseurl을

    baseurl: ""

이런식으로 지워주면 된다. 앞서 설명했듯, baseurl은 레포지토리의 이름을 다른 이름으로 설정했을때 필요한 설정이기 때문에 지워야 한다. 그리고

    sass:
    sass_dir: _sass
    style: compressed

부분을 추가해 주면 된다. 이 부분은 만약 bundle exec jekyll serve를 실행했을때 아무런 상관이 없거나, github page 상에서 아무 문제 없이 deploy되면 추가할 필요는 없다. 하지만 내 경우에는 

    Your SCSS file assets/main.scss has an error on line 2: File to import not found or unreadable

에러가 생겼기 때문에 추가했다.
만약 구글 애널리틱스를 사용하지 않는다면 주석 처리해놔도 상관 없다.

2. webrick 문제

bundle exec jekyll serve를 터미널에서 실행했을 때,
webrick 관련 오류가  설치되지 않았다는 오류가 뜨는 경우가 있다. 이런 경우에는 자신의 .gemfile을 열어서

<img src="/img/posts/clean-blog-setup/07.png" style="width: 75%">

이런식으로, 

    gem "webrick"

을 추가해 준 뒤

    bundle install
    buncle exec jekyll serve

를 터미널에서 실행하면 된다.

3. Your SCSS file assets/main.scss has an error on line 2: File to import not found or unreadable 문제

이 경우에는 로컬에서는 문제 없이 빌드되는데, 깃허브 상에서 올바르게 deploy 되지 않는 문제였다. 위 경우에는 

[위 링크](https://github.com/StartBootstrap/startbootstrap-clean-blog-jekyll/issues/219)를 많이 참고했는데, 해결 방법으로는 

    sass:
    sass_dir: _sass
    style: compressed

를 _config.yml 파일에 추가하고 .gitignore 파일에서 vendor와 vendor/bundle을 삭제하면 정상적으로 깃허브 상에서 빌드된다!!

수정하는 방법으로는 터미널에서

    nano .gitignore

를 한 뒤에, vendor, vendor/bundle을 삭제하고, contrl + x를 누른 뒤 저장하거나, 숨긴파일을 모두 보도록 설정한 뒤, .gitignore 파일을 직접 수정해도 된다.

<img src="/img/posts/clean-blog-setup/08.png" style="width: 75%">

이런식으로!

4. 혹시라도 index.md, index.html, 또는 home.md, about.md, 404.html, 404.md등이 이미 존재한다는 오류

이는

    jekyll new 

등으로 이미 디렉터리에 지킬 파일을 만들어 놓은 경우나, 다른 테마를 설치하거나 한 뒤 남아있는 파일이 중복되는 경우에 발생한다. index.md, index.html 등은 jekyll에서 같은 파일로 보기 때문에 빌드함에 있어서 충돌이 나는 경우다.

이런 경우에는 간단하게, 자신이 사용하고 싶은 페이지를 선택하고 다른 파일을 삭제하면 된다.(jekyll은 html이나 md 모두 빌드할 수 있다)

## 마치면서

깃허브 및 기본적인 지식은 알 것이라고 생각하고 넘어간 부분이 있었는데, 그리고 내가 이미 빌드해 놨기 때문에 처움부터 모든 오류를 서술할 수는 없었지만, 대부분은 gpt가 해결할 수 있거나, 사소한 오류인 경우가 많기 때문에 따로 적지는 않았다.

Your SCSS file assets/main.scss has an error on line 2: File to import not found or unreadable 문제가 가장 귀찮았는데 혹시라도 문제가 있다면 위 방법으로 해결할 수 있었으면 한다. 

끝!