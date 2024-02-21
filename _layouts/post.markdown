---
layout: default
---

<!-- Page Header -->

{% if page.background %}
<header class="masthead" style="background-image: url('{{ page.background | prepend: site.baseurl | replace: '//', '/' }}')">
{% elsif page.background_color %}
<header class="masthead" style="background: {{page.background_color }}">
{% else %}
<header class="masthead">
{% endif %}
  <div class="overlay"></div>
  <div class="container">
    <div class="row">
      <div class="col-lg-8 col-md-10 mx-auto">
        <div class="post-heading">
          <h1>{{ page.title }}</h1>
          {% if page.subtitle %}
          <h2 class="subheading">{{ page.subtitle }}</h2>
          {% endif %}
          <span class="meta">Posted by
            <a href="#">{% if page.author %}{{ page.author }}{% else %}{{ site.author }}{% endif %}</a>
            on {{ page.date | date: '%B %d, %Y' }} &middot; {% include read_time.html
            content=page.content %}
          </span>
<!-- 카테고리를 추가하는 코드 -->
            <a>
              카테고리
            <a>
          <div style="font-family: 'Noto Sans KR', sans-serif;">
            {% for category in page.categories %}
              <a href="{{ site.url }}/category/{{ category }}.html" style="color: white;">📁{{ category }}</a>
            {% endfor %}
          </div>
<!-- 카테고리를 추가하는 코드 -->
        </div>
      </div>
    </div>
  </div>
</header>
<div class="container" style="display: flex">
  <div class="row">
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
                <a style= "margin-bottom: 0px; font-size: 15px; color: black; font-weight: bold;" href= "{{ site.url }}/category/{{ category[0] }}.html">
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
                        <a href="{{ site.baseurl }}{{ post.url }}" style= "color: black; font-size: 12px;">
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

    <div class="col-lg-8 col-md-10 mx-auto">
<!-- 본문 카테고리를 추가하는 코드 -->
            <a style="font-family: 'Noto Sans KR', sans-serif;">
              카테고리 링크
            <a>

          <div style="font-family: 'Noto Sans KR', sans-serif;">
            {% for category in page.categories %}
              <a href="{{ site.url }}/category/{{ category }}.html" style="color: black;">📁 {{ category }}</a>
            {% endfor %}
          </div>
          
<!-- 본문 카테고리를 추가하는 코드 -->

      {{ content }}

      <hr>
      {% include thanks.html %}
      <div class="clearfix" >

<!-- 수정중 -->
<!-- 가장 첫번쨰 카테고리의 최근 글 불러오는 글 기능 추가  -->
<!-- <ul class="posts-list"> -->

<ul class="mb-5 ">
  {% assign firstCategory = page.categories | first %}
  <p>
    <a style= "font-size: 20px; font-weight: bold; font-family: 'Noto Sans KR', sans-serif;" href= "{{ site.url }}/category/{{ firstCategory }}.html">
      {{ firstCategory }} 카테고리의 다른 글
    </a>
  </p>
  {% assign posts = site.categories[firstCategory] | sort: 'date' | reverse | limit: 5 %}
  {% assign counter = 0 %}
  {% for post in posts %}
    {% if counter < 5 %}
      <li>
        <h3>
          <a href="{{ site.baseurl }}{{ post.url }}" style= "font-size: 17px;">
            {{ post.title }}
          </a>
          <small style= "font-size: 14px;">{{ post.date | date_to_string }}</small>
        </h3>
      </li>
      {% assign counter = counter | plus: 1 %}
    {% endif %}
  {% endfor %}
</ul>


<!-- 수정중 -->
<!-- 가장 첫번쨰 카테고리의 최근 글 불러오는 글 기능 추가  -->


<!-- 코멘트 란을 추가하는 코드 -->

{% if page.comments %}
  <div id="disqus_thread"></div>
  <script>
      /**
      *  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
      *  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables    */
      /*
      var disqus_config = function () {
      this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
      this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
      };
      */
      (function() { // DON'T EDIT BELOW THIS LINE
      var d = document, s = d.createElement('script');
      s.src = 'https://tensorstudio.disqus.com/embed.js';
      s.setAttribute('data-timestamp', +new Date());
      (d.head || d.body).appendChild(s);
      })();
  </script>
  <noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
{% endif %}

<!-- 코멘트 란을 추가하는 코드 -->

        {% if page.previous.url %}
        <a class="btn btn-primary float-left" href="{{ page.previous.url | prepend: site.baseurl | replace: '//', '/' }}" data-toggle="tooltip" data-placement="top" title="{{ page.previous.title }}">&larr; Previous<span class="d-none d-md-inline">
            Post</span></a>
        {% endif %}
        {% if page.next.url %}
        <a class="btn btn-primary float-right" href="{{ page.next.url | prepend: site.baseurl | replace: '//', '/' }}" data-toggle="tooltip" data-placement="top" title="{{ page.next.title }}">Next<span class="d-none d-md-inline">
            Post</span> &rarr;</a>
        {% endif %}

      </div>

    </div>
  </div>
</div>
