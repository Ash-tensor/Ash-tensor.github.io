---
layout: post
comments: true
title: "[깃허브 블로그]지킬 블로그 카테고리 / NAVBAR 기능 추가(clean blog theme) - 1"
subtitle: 카테고리 기능 추가 + navbar 커스터마이징의 첫번째 포스트 (1 / 2)
description: troubleshooting
date: 2023-12-15
categories: 깃허브
background: '/img/posts/06.jpg'
---


# 깃허브 블로그 카테고리 기능 추가(clean blog theme)

네이버 블로그나 티스토리 블로그를 확인해 보면 당연히 카테고리 섹션이 있다. 이것도 역시, 댓글창처럼 당연하다면 당연하다고 생각한다. 그럴게, 블로그나 인터넷에서 글을 볼 때, 검색해서 들어가게 되면 그 주제의 글을 계속 봐야 할 때가 생긴다. 웹툰도, 만화도 1화만 보고 그 다음 화를 보지 못하면 답답하잖아?

<img src="/img/posts/category/01.png" style="width: 75%">

위에 볼 수 있는 것처럼 Clean blog theme에는 전체 post를 볼 수 있는 기능은 있지만 카테고리 기능이 없어서 불편하다고 생각했다.

필요하다면 만든다. 만드는데 조금 시간이 걸렸다. 

## 카테고리 레이아웃

<img src="/img/posts/category/02.png" style="width: 75%">

일단 내가 구현한 방법으로는, 위 사진처럼 전체 카테고리를 나열하는 전체 카테고리와

<img src="/img/posts/category/03.png" style="width: 75%">

그리고 그 세부 카테고리의 모든 글을 표시해주는 페이지를 만드는 방법으로 구현한 뒤에, 이 페이지에 존재하는 것처럼 navigation bar에 카테고리를 추가하는 것이다. 

첫번째로 카테고리 레이아웃을 생성해야 한다. _layouts 디렉토리에 category.markdown 파일을 생성한다. 그리고

<img src="/img/posts/category/04.png" style="width: 75%">

    {% raw %}
    ---
    layout: page
    ---
    <ul class="posts-list">
    
    {% assign category = page.category | default: page.title %}
    {% for post in site.categories[category] %}
        <li>
        <h3>
            <a href="{{ site.baseurl }}{{ post.url }}">
            {{ post.title }}   
            </a>
            <small>{{ post.date | date_to_string }}</small>
        </h3>
        </li>
    {% endfor %}
    
    </ul>
    {% endraw %}

위 코드를 복사, 붙여넣기 한다.

위 코드는 이미 존재하는 페이지 레이아웃을 사용하면서, 모든 페이지를 순회하며 YAML 프론트매터의 categories 부분을 이용하여 카테고리 링크를 생성하는 코드다. 

그리고 _layouts 디렉토리의 post.markdown을 수정한다. 

    {% raw %}
    <!-- 카테고리를 추가하는 코드 -->

            <div>
                {% for category in page.categories %}
                <a href="{{ site.url }}/category/{{ category }}.html" style="color: white;">카테고리📁: {{ category }}</a>
                {% endfor %}
            </div>

    <!-- 카테고리를 추가하는 코드 -->
    {% endraw %}

위 코드를 붙여 넣으면 되는데, 이는 YAML 프론트매터를 이용해서 현재 페이지의 category를 이용해서 해당하는 링크를 만들어주는 코드다.

<img src="/img/posts/category/05.png" style="width: 75%"> 

나는 위 사진처럼 제목 부분 헤더 컨테이너 안에 하나, 그리고 본문 제목 위에 하나, 이렇게 삽입하고 싶기 때문에 두 번 삽입했다. 자신이 삽입하기 윈하는 위치에 삽입하면 된다.

만약 위 사진처럼 삽입하고 싶다면

<img src="/img/posts/category/06.png" style="width: 75%"> 

여기에 한 번

<img src="/img/posts/category/07.png" style="width: 75%"> 

여기에 한 번 복사해 넣으면 된다.

## 카테고리 페이지 추가

이게 끝이 아니다. 지킬 페이지는 개별 카테고리의 페이지를 직접 생성해 줘야 한다.

<img src="/img/posts/category/03.png" style="width: 75%">

바로 이 페이지를 만드는 과정이다. 만약 웹 페이지의 카테고리를 추가하기 위해서는 앞으로도 이 과정을 계속 수행해 줘야 한다! 그렇지 않으면 당연히 오류가 난다. 예를 들어서 당신이 지금은 C 카테고리만 있지만, java 카테고리를 만들고 싶으면 java 페이지를 추가해야 하는 것이다.

귀찮다. 하지만 어쩔 수 없다. 

일단 루트 디렉토리에 category라는 디렉토리를 생성해야 한다. 어떤 방법으로 생성해도 상관없다.

그리고 난 뒤, 자신의 원하는 카테고리 이름으로 markdown 파일을 생성한다. 예를 들어서 이 페이지의 카테고리인 깃허브 카테고리를 만드려면

    깃허브.markdown 
    이런 식으로 이름을 생성하면 된다.

그리고 난 뒤 그 파일에 코드를 작성해 줄 텐데

<img src="/img/posts/category/08.png" style="width: 75%">

    ---
    layout: category
    title: 자격증
    description: 자격증 📁카테고리의 모든 글.
    background: '/img/bg-category.jpg'
    ---

이런 식으로 코드를 작성해 주면 된다. layout은 category를 사용하지만(방금 전 작성한) category layout은 page 레이아웃을 상속받아 사용하기 때문에 descripion 및 background 프론트매터를 사용 가능하다!

descripion은 카테고리 페이지의 설명에 사용되니 자기가 원하는대로 사용하면 된다.

이러면 카테고리 페이지는 완성했다!

사실 한번에 쓰고자 했었는데, 쓰다 보니까 내용이 너무 길어져서 전체 카테고리 포스트 생성과 navigation bar를 생성하는 법은 다음 포스트에 작성하도록 하겠다.