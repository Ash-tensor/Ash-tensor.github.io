---
layout: post
comments: true
title: "[WEB] GCP App Engine을 이용한 프론트엔드 서비스 배포"
subtitle: 
description: 
date: 2024-08-05
categories: WEB
background: '/img/port.jpg'
---

## [WEB] GCP App Engine을 이용한 리액트 프론트엔드 서비스 배포

### GCP App Engine

와, 솔직히 너무 오랜만에 돌아왔다. 거의 세달 만인데 솔직히 이야기해서 너무 일이 많았다. 정보처리기사, 빅데이터분석기사와 같은 자격증들을 준비하느라 고생했고,
그리고 이제는 취업준비를 하느라 너무 바쁜 일상을 보내고 있다. 6월부터 7월 내내 팀 프로젝트를 진행하고 기간에 맞추느라 매번 배운 것들을 정리해야지, 정리해야지 하다가
프로젝트 배포가 끝나고 나서야 돌아오게 됐다. 프로젝트 배포를 하면서 GCP의 App Engine을 이용해서 프론트엔드 서비스를 배포했는데, 이번 기회에 그 과정을 정리해보려고 한다.
솔직히 어려운 것도 없지만... 그래도 정리해보자.

### 프론트엔드

우선 프론트엔드는 React로 만들어진 프로젝트였다. 실제 배포한 프로젝트는 다음 링크로 들어가서 확인해보면 된다. 비밀번호는 1111, 1111로 설정되어 있다.

[Kiosk](http://white-faculty-427513-f2.de.r.appspot.com/users/login)

<img src="/img/posts/gcp/gcp_appengine/img.png" width="80%">

이런 식의, PG사의 결제 모듈을 결합한 음성인식 지능형 키오스크를 제작하는 것이 목표인데, 백엔드는 스프링으로 제작했고 프론트엔드는 리액트로 작성했다.
일단 백엔드는 AWS의 EC2 프리티어를 이용해서 배포했는데, 팀원 중 한명이 프론트엔드를 배포하는데 어려움이 있어해서, 이번에 한번 처음으로 GCP의 App Engine을 이용해 보았다!

### CORS 문제

우선 프론트엔드를 배포하기 전에 CORS 문제를 해결해야 했다. 팀원이 마주했던 문제가 바로 CORS 문제였다.

<img src="/img/posts/gcp/gcp_appengine/img2.png" width="70%">

지금은 문제를 해결해서 나오지 않지만, 위 화면의 개발자 도구 창의 콘솔을 보면 CORS 문제가 발생했다는 것을 알 수 있었다.

* Access to XMLHttpRequest at '주소A' from origin '주소B' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

아무래도 스프링 콘솔이나 AWS의 EC2에서는 아무런 로그도 나오지 않기 때문에 이런 문제를 해결하기가 어려웠을 것이라고 생각한다.

백엔드는 AWS의 EC2 프리티어를 이용해서 배포했는데, 프론트엔드는 GCP의 App Engine을 이용해서 배포했기 때문에
서로 다른 도메인에서 서비스를 제공하게 되었다. 이런 경우에는 CORS 문제가 발생하는데, 이를 해결하기 위해서는 백엔드에서 CORS를 허용해주는 설정을 해주어야 했다.

~~~ java

package ac.su.kiosk.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
                        .allowedOrigins("http://white-faculty-427513-f2.de.r.appspot.com") // React 앱의 URL
                        .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                        .allowedHeaders("*") // 모든 헤더를 허용
                        .allowCredentials(true); // 쿠키 및 인증 정보를 허용
            }
        };
    }
}
~~~

### npm run build

CORS 문제를 해결한 다음에는 리액트 앱을 빌드해 주어야 한다. npm run build로 프로젝트를 빌드해 주고, GCP의 APP Engine이 빌드하고 배포하는데 
필요한 환경설정을 해 주어야 한다.

일단 프로젝트 디렉토리에서 serve 패키지를 전역으로 설치한다.

~~~ bash

npm install -g serve

~~~

이후에, package.json 파일을 열어서 scripts 부분을 수정해준다. scripts의 start 부분을 "serve -s build"로 수정해주면 된다.
자세한 예는 다음과 같다.

~~~ json

"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "serve": "serve -s build"
  },

~~~

### GCP CLI 설치

이 이후에 GCP CLI를 설치해야 하는데, 이 과정이 조금은 번거롭다. 일단 GCP의 콘솔에 들어가서 프로젝트를 생성해주고, 프로젝트 ID를 확인해야 한다.
일단 https://cloud.google.com/sdk/docs/install?hl=ko#mac 를 들어가서 GCP CLI를 다운로드 받아, 압축을 푼 다음
google-cloud-sdk 폴더 내의 install.sh 파일을 실행하면 되는데, 구글 공식 홈페이지에는 특정 파이썬 버전을 권장한다고 쓰여 있지만
맥 기준 설치하면 알아서 해당 파이썬 버전을 설치해주기 때문에 그냥 넘어가도 된다.

또한 구글에서는 딱히 홈 루트에서 실행해야 된다는 말은 없는데, 다른 설치글에서는 홈 루트에서 실행을 권장하니까, 혹시 불안하거나 문제가 생긴다면 홈 루트에서 실행해보자.
그리고 환경변수 추가 및 다양한 설정에 대해 안내가 나오는데 환경변수를 나중에 추가하고 싶지 않다면 지금 추가하면 된다.

이 이후, 터미널을 열어서 gcloud init을 실행하면 로그인 창이 뜨는데, 로그인을 하고 프로젝트 ID를 입력하면 된다.


### app.yaml 파일 생성

이후에는 프로젝트 디렉토리에 app.yaml 파일을 생성해야 한다. 이 파일은 GCP의 App Engine이 프로젝트를 배포할 때 필요한 설정 파일이다.
이 파일은 프로젝트 디렉토리에 생성해야 한다. 이 부분에서 살짝 트러블 슈팅이 필요했는데, GPT가 안내한대로 진행했을때는 배포가 안되고 오로지 
흰색 화면만 뜨는 문제가 발생했다. 이 문제를 해결하기 위해서는 app.yaml 파일을 다음과 같이 수정해주어야 한다.

~~~ bash

runtime: nodejs20

instance_class: F1

handlers:
  - url: /static
    static_dir: build/static

  - url: /(.*)
    static_files: build/index.html
    upload: build/index.html

~~~

handlers 부분에서 url: /static과 static_dir: build/static은 리액트 앱에서 사용하는 정적 파일들을 가리키는 부분이다.
url: /(.\*)과 static_files: build/index.html은 리액트 앱의 라우팅을 위한 설정이다. 이 부분을 추가해주지 않으면 라우팅이 제대로 되지 않는다.
GPT는 이부분을 url: /.\*과 static_files: build/index.html로 설정했는데, 이렇게 설정하면 라우팅이 제대로 되지 않는다. 이 점 주의하자.

또한 instance_class 부분은 F1로 설정해주면 된다. 이 부분은 GCP의 App Engine의 인스턴스 클래스를 설정하는 부분인데, F1은 무료 티어를 의미한다.

### 배포

이제 배포를 해주면 된다. 배포는 다음과 같이 진행하면 된다.

~~~ bash

gcloud app deploy

~~~

이렇게 입력하면 배포가 진행되는데, 배포가 완료되면 다음과 같은 화면이 뜨는데, 이 화면에서 배포된 프로젝트의 주소를 확인할 수 있다!!

<img src="/img/posts/gcp/gcp_appengine/img3.png" width="70%">


### 마치며

이렇게 GCP의 App Engine을 이용해서 프론트엔드 서비스를 배포하는 방법을 정리해보았다. 이번 프로젝트에서는 우연히도 GCP API를 많이 사용하게 되었는데,
아무래도 처음 시작할 때 300$의 크레딧을 제공해주는 것이 매력적이었던 것 같다. 또한 GCP 클라우드 버킷을 이용해서 이미지를 업로드하는 부분에서 
조금 고생했는데 이 부분도 나중에 정리해 보도록 해야겠다.