---
layout: post
comments: true
title: "[깃허브 블로그]지킬 블로그 코멘트 기능 추가(clean blog theme)"
subtitle: disqus를 이용한 clean blog theme 댓글 코멘트 기능 추가
description: troubleshooting
date: 2023-12-15
categories: 
background: '/img/posts/06.jpg'
---


# 깃허브 블로그 코멘트 기능 추가(clean blog theme)

블로그에는 기본적으로 댓글 란은 무조건 있어야 한다고 생각한다. 추가하는게 어렵지 않다. 터미널을 두드릴 필요도 없다. 오히려 카테고리 란을 추가하는게 조금 더 어려운 것 같다. 그러면 한번 추가해 보자고!


## DISQUS

깃허브 페이지를 사용하는 데 있어서 가장 많이 사용하는 덧글/코멘트 기능이라고 하면 DISQUS다. STACK OVERFLOW나 다른 곳에서도 가장 먼저 추천하는 기능이다. 게다가 무료다! 사용하지 않을 이유가 없다.

<img src="/img/posts/clean-blog-setup/09.png" style="width: 75%">

지금 이 포스트 아래에도 붙어있는 댓글란인데, 많은 깃허브 페이지들을 보다 보면 대부분의 댓글이 이렇게 생긴 것을 확인할 수 있을 것이다.

[일단 여기](https://disqus.com/profile/signup/intent/)서 회원가입을 해 주면 된다.

<img src="/img/posts/clean-blog-setup/10.png" style="width: 75%">

회원가입을 하고 나면 이런 창이 보일 텐데, I want to install Disqus on my site를 눌러 주면 된다. 그리고 사이트 이름을 정해주고 난 뒤, 요금이 들지 않는 BASIC 요금제를 선택해 주고 select platform에서 jekyll을 선택해 주면 된다. 

<img src="/img/posts/clean-blog-setup/12.png" style="width: 75%">

그리고 나면 

<img src="/img/posts/clean-blog-setup/13.png" style="width: 75%">

위 사진에서도 나와있듯이 Universial Embed code 링크를 클릭해서 들어간 뒤 

    <div id="disqus_thread"></div>
    <script>
        /**
        *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM
        *  ...
        *  ...
        *  생략함

위 꼴의 universial embed code를 복사한다. 그리고 난 뒤 자신의 지킬 블로그가 설치되어 있는 로컬 폴더로 들어간다.

그리고 난 뒤, _layout 디렉토리의 post.markdown 파일로 들어간 뒤

<img src="/img/posts/clean-blog-setup/15.png" style="width: 75%">

이 부분에 아까 복사한 자신의 코드를 붙여넣기 하면 된다! 이제 다 끝났다!

## YAML 프론트매터

위 코드는 자신의 disqus 댓글창을 생성하는 코드로, 당연하겠지만 항상 코멘트 섹션을 생성한다. 만약 그게 싫고, 포스트별로 댓글 창을 설정하고 싶은 경우에는 추가적으로 코드를 적어 넣어야 한다.

<img src="/img/posts/clean-blog-setup/14.png" style="width: 75%">

이 포스트의 프론트매터인데, 댓글을 달고싶은 포스트의 프론트매터에 위 사진처럼

    comments: true

속성을 넣어주면 된다. 그러면 지킬이 페이지를 구성할 때, disquss를 참조해서 포스트를 구성하게 된다. 그리고 난 뒤, 아까 수정했던 post.markdown에 들어가서 아까 작성했던 코드 스니펫의 앞 뒤에

{% if page.comments %}, {% endif %} 를 작성해 주면 된다. 

    {% if page.comments %}


    <div id="disqus_thread"></div>
    <script>
        /**
         *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
         *  LEARN WHY DEFINING THESE VARIABLES 

        ...생략함...


    {% endif %}

이런식으로!

충분히 이해 가능하겠지만, 이는 페이지의 yaml 프론트메터의 comments 부분이 참이면 아래의 코드 스니펫을 실행하라는 의미이다. 
아, 그런데 주의해야 할 점은 이 리퀴드라는 언어는 띄어쓰기에 엄청 민감하다. 만약 {% if page.comments% } 이런 식으로 의미없는 띄어쓰기가 있어도 바로 오류를 뱉어내니까, 주의해야 한다.