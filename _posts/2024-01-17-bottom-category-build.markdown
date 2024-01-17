---
layout: post
comments: true
title: "[깃허브 블로그] 동일 카테고리의 최근 글 기능 개발(clean blog theme)"
subtitle: 
description: 
date: 2024-01-17
categories: 깃허브
background: '/img/port.jpg'
---

## 동일 카테고리의 최근 글 기능

<img src="/img/posts/clean-blog-setup/18.png" style="display: block; margin-left: auto; margin-right: auto; width: 80%">

사진을 보면 이해가 아주 쉬울텐데
티스토리 블로그나 네이버 블로그 모두, 블로그 하단부에는 최근 동일 카테고리의 글 몇개를 보여주는 기능이 존재한다(정확히 무슨 이름인지는 모르겠다). 상황에 따라 전부 보여주기도 하고, 최근 글 몇개만 보여주기도 한다. 아마 잘은 모르겠지만 blogger나 wordpress 모두 동일할 것이다. 나도 그런 블로그에 익숙해서인지 모르겠지만 NavBar에 카테고리를 추가하기도 하고, 헤더 부분과 본문에 카테고리 링크를 표시하는 기능을 추가하기도 했는데 아무리 그래도 블로그에 이 기능이 없으면 허전하다고 느꼈고, 리퀴드 / html에 익숙해지기도 했고 해서, 이렇게 추가하게 되었다.

### 코드

post.markdown 파일을 수정하면 된다. 나는 댓글창 위에 있는게 편해서 if page.comments 부분 위에 추가했는데 본인이 불편하지 않은 곳 어디든 추가하면 된다. 당연히 기존 코드 구조를 해치지 않는 선에서.

~~~ 
{% raw %}
<ul class="row mb-5 ">
{% assign firstCategory = page.categories | first %}
<p>
    <a style= "font-size: 20px; font-weight: bold;" href= "{{ site.url }}/category/{{ firstCategory }}.html">
    {{ firstCategory }} 카테고리의 다른 글
    </a>
</p>
{% assign posts = site.categories[firstCategory] | sort: 'date' | reverse | limit: 5 %}
{% assign counter = 0 %}
{% for post in posts %}
    {% if counter < 5 %}
    
    <li>
        <h3>
        <a href="{{ site.baseurl }}{{ post.url }}" style= "font-size: 20px;">
            {{ post.title }}
        </a>
        <small style= "font-size: 20px;">{{ post.date | date_to_string }}</small>
        </h3>
    </li>
    {% assign counter = counter | plus: 1 %}
    {% endif %}
{% endfor %}
</ul>

{% endraw %}
~~~

### 설명

    {% assign firstCategory = page.categories | first %}

지킬에서 사용하는 리퀴드 언어에서는 assign을 이용해서 변수를 할당한다. 이건 자바스크립트와 조금 비슷한 것 같다. 그래서 코드를 살펴보면  firstCategory라는 변수에 page의 categories의 첫번째 카테고리를 할당한다. 여기서 page.categories라는 리스트는 프론트매터의 categoris: xxx에 해당하는 부분이다. 

<img src="/img/posts/clean-blog-setup/19.png" style="display: block; margin-left: auto; margin-right: auto; width: 80%">

바로 이 부분.

당연하겠지만 이 기능은 첫번째 카테고리의 내용만을 표시한다. 카테고리에는 여러개의 카테고리를 할당할 수는 있지만 보여주는건 하나뿐이다. 아무래도 여러개를 보여주면 너무 헷갈리지 않을까 싶어서 이렇게 설정했다. 리퀴드에서는 파이썬의 for문과 똑같이 for x in xlist 구조의 향상된 순회가 가능하다.


    {% raw %}
    {% assign posts = site.categories[firstCategory] | sort: 'date' | reverse | limit: 5 %}
    {% assign counter = 0 %}
    {% for post in posts %}
        {% if counter < 5 %}
        
        <li>
            <h3>
            <a href="{{ site.baseurl }}{{ post.url }}" style= "font-size: 20px;">
                {{ post.title }}
            </a>
            <small style= "font-size: 20px;">{{ post.date | date_to_string }}</small>
            </h3>
        </li>
        {% assign counter = counter | plus: 1 %}
        {% endif %}
    {% endfor %}
    {% endraw %}


그리고 위 부분에서는 예전에 카테고리를 만들때 사용했던 코드를 응용했는데, 이해가 가지 않았던 점은 분명 **limit: 5**를 설정했는데 어떻게 해도 5개 이상으로 표시가 되어서 GPT에게도 물어봤지만 "코드는 정확해, 아무런 문제도 없어." 라는 대답만 들을 수 있었다. 왜 그런지 잘 모르겠는데, 정렬 문제인가 싶어서 여러번 설정을 바꿔봤지만 그 문제는 아닌 것 같다. 어쩔 수 없이 그냥 for 문에 카운터를 설정해서 5개만 표시되도록 코드를 수정했다. << 이유 아는 사람 있으면 알려주시길

아무튼 수정한 결과 이 문단의 바로 아랫부분에 나오는 페이지를 완성할 수 있었다! CSS 스타일을 내가 처음부터 직접 만든 게 아니라 상당히 투박하기도 하고 모바일에서 어떻게 나올지는 잘 모르겠지만 일단 작동은 하긴 하니까, 여기에서 마무리 지어야겠다.