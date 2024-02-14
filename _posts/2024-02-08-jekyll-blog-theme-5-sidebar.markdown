---
layout: post
comments: true
title: "[깃허브 블로그]지킬 블로그 카테고리 / SIDEBAR 기능 추가(clean blog theme)"
subtitle: 사이드바 기능 추가
description: 
date: 2024-02-08
categories: 깃허브 
background: '/img/port.jpg'
---


## 깃허브 블로그 SIDEBAR 기능 추가(clean blog theme)

블로그에 사이드바 기능을 추가할 지 말지 고민했는데, 그래도 기능은 많으면 많을수록 좋다고 느껴서 사이드바 기능을 추가했다. 
최근 HTML과 CSS를 공부하는 중이라서 공부하는 겸 sidebar 기능을 개발해 보기로 했다. 리퀴드를 이용해서 구현했고, 추가적으로 카테고리별로 축소시켰다, 늘렸다 하는 기능을 추가할까도 생각중이다.
CLEAN BLOG THEME은 자체적으로 사이드바를 지원하는 테마는 아니기 때문에 여타 다른 테마의 수려한 기능과는 거리가 멀긴 하지만 일단 작동은 하게 만들었으니, 이정도로 마무리 지을까 한다. 

### 카테고리 레이아웃

이를 구현하기 위해서는 사이드바와 컨텐츠를 수평으로 담기 위해서 최상의 컨테이너의 display 속성을 flex로 변경해야 한다. _layouts의 post.markdown 파일이다.

<img src="/img/posts/sidebar/sidebar-1.png" style="width: 80%">

그리고 너무 작은 화면에서는 사이드바가 나오지 않게 하기 위해(너무 작은 화면에서는 사이드바가 있어봐야 소용이 없기 때문에 / 그리고 화면이 너무 지저분해지기도 하고) general 스타일 시트를 변경하려고 했는데, general.scss 파일은 물론이고 asset의 main css, post.scss 파일을 아무리 수정해도 스타일이 먹지를 않아서 어쩔 수 없이 header에 직접 스타일을 때려넣어야만 했다. 

<img src="/img/posts/sidebar/sidebar-2.png" style="width: 80%">

이런식으로, 기본적으로 사이드바의 display 설정은 none으로 하고, 일정 크기 이상으로 화면이 커져야만 display 되도록 하는 방식이다. 솔직히 아직도 너무 지저분한 방식이라고 생각은 하지만, 왜 적용이 안되는지 방법을 모르겠어서 어쩔 수 없었다... 아무튼, 이 이후에 .post문서의 세번째 컨테이너에 이 코드를 추가하면 된다. 첫번쨰 사진을 확인하면 어떻게 추가하는지 이해가 쉬울 것이다.


    {% raw %}
    <!-- sidebar를 추가하는 코드 -->
    <sidebar id="sidebar" 
                style="width: 260px; 
                border-right: gray 1px solid; 
                padding-left: 0; 
                padding-right: 20px;"
                >
        <ul>
            <p style="font-weight: bold; border-top: black 1px solid; border-bottom: black 1px solid; text-align: center; margin-top: 2px; padding: 2px; padding-top: 4px">
                TENSOR STUDIO
            </p>
            {% for category in site.categories %}
                <p style="margin-bottom: 0; margin-top: 10px">
                <a style= "margin-bottom: 0px; font-size: 15px; color: black; font-weight: bold; font-family: 'Noto Sans KR', sans-serif;" href= "{{ site.url }}/category/{{ category[0] }}.html">
                    {{ category[0] }}
                </a>
                <a> 🔽
                </a>
            </p>
            {% assign posts = category[1] | sort: 'date' | reverse | limit: 5 %}
            {% assign counter = 0 %}
            {% for post in posts %}
            {% if counter < 3 %}
            <li style="margin: 0px">
                <a href="{{ site.baseurl }}{{ post.url }}" style= "color: black; font-size: 12px; font-family: 'Noto Sans KR';">
                    {{ post.title }}
                </a>
                <small style= "margin-bottom: 0px; font-size: 10px;">{{ post.date | date_to_string }}</small>
            </li>
                {% assign counter = counter | plus: 1 %}
                {% endif %}
                {% endfor %}
            {% endfor %}
        </ul>
    </sidebar>
    {% endraw %}

### 완성된 모습

<img src="/img/posts/sidebar/sidebar-3.png" style="width: 80%">

현재는 최근 세 개의 포스트만 나오게 되어 있는데, 이는 내부 코드의 카운터 부분을 바꾸면 된다. 추가적으로 레이아웃에서 건드릴 것이 많기는 하지만(display 속성을 이용해서 완성도를 높일 수 있을 것 같다) 아무튼 여기서 마무리 지을 것이다!


