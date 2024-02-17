---
layout: post
comments: true
title: "[WEB]GCP(google cloud platform) 프리 티어 VM 생성 / MYSQL 서버 연결"
subtitle: mysql 서버 설정 및 연결
description: 
date: 2024-02-17
categories: WEB
background: '/img/port.jpg'
---


## [WEB]GCP(google cloud platform) 프리 티어 VM 생성

클라우드 서비스로 가장 우선적으로 선택하는 것 중에 하나가 바로 AWS일 것이다. 하지만 AWS 같은 경우에는 프리티어 VM의 사용 기한이 1년으로, 
1년이 지나면 자동으로 등록해 놓은 신용카드로 사용 요금이 빠져나간다. 그래서 나 같은 경우에는 프리티어로 사용하던 VM의 사용 기한이 끝나 매달 2만원 가량의 요금이
 지불되고 있는데... 솔직히 돈이 너무 아까웠다. 그래서 이에 대한 대책으로 다른 클라우드 서비스 프리 티어를 살펴보고 최종적으로 GCP에 MYSQL 서버를 설치하고 접속되게 변경했다.

### 오라클은 가입이 안됨...

사실 이런 프리티어 클라우드 서비스 중에 가장 성능이 좋은 서비스는 오라클이다. 그런데 이 오라클 클라우드의 가장 큰 단점은 회원가입이 어럽다는 점이다...
농담이 아니라 실제 많은 사람들이 호소하는 불편함으로, 이 마지막 화면에서 "트랜잭션 오류가 발생하였습니다." 라는 오류를 뱉으며 회원가입이 안 되는 오류가 있다.

<img src="/img/posts/gcp/oraclefail.png" style="width: 80%">

바로 이 화면이다. 해결 방법으로는 오라클 고객센터 라이브 챗으로 풀어달라고 하면 해결된다고는 하는데, 나 같은 경우에는 응답이 없어서, 그냥 GCP로 바꾸기로 했다.

### GCP

GCP와 오라클이 프리티어에서 가장 큰 강점을 가지는 것은 바로 영구 무료 유지라는 점이다. AWS는 물론이고 MS 애저도 1년이 지나면 얄짤없이 유료화 시키는데 비해서
프리 티어 성능 안에서는 1년이 지나건 몇년이 지나건 무료로 서비스를 사용할 수 있다는 점이 큰 매력이다.

<img src="/img/posts/gcp/gcp1.png" style="width: 80%">

이 사진에서 알수있듯, 오리건, 아이오와, 사우스캐롤라이나의 VM이라면 30GB 용량까지 평생 무료로 사용할 수 있다. 하지만 스펙이 저열한데, 프리티어의 e2-micro
성능은 vcpu 1개와 0.5GB 메모리, 그리고 1GB 트래픽만 허용된다.

vcpu 1개의 경우에는 대략 0.2Ghz 정도의 성능을 가지는데, 아무리 무료라고는 하지만 좋은 성능은 아니다. 0.2Ghz다. 0.2Ghz!! 아마 피쳐폰보다 성능이 안좋을지도 모른다.
게다가 램도 겨우 500메가 정도로, 정말 20달러짜리 라즈베리파이보다 성능이 안좋을 것이다. 라즈베리파이는 단 돈 5만원에 2Ghz CPU 8개가 달려있는데...

아무튼 프리티어는 프리티어인 이유가 있다. 간단한 실증 정도의 테스트만 가능한 환경이니 너무 큰 프로젝트는 올리지 않는 편이 좋다.

#### VM생성

[이 페이지(GCP)](https://cloud.google.com/free/?utm_source=google&utm_medium=cpc&utm_campaign=japac-KR-all-en-dr-BKWS-all-core-trial-EXA-dr-1605216&utm_content=text-ad-none-none-DEV_c-CRE_644033776725-ADGP_Hybrid+%7C+BKWS+-+EXA+%7C+Txt+~+GCP_General_core+brand_main-KWID_43700074755444107-kwd-87853815&userloc_1009871-network_g&utm_term=KW_gcp&gad_source=1&gclid=CjwKCAiArLyuBhA7EiwA-qo80PzDfq52iWQuvaP7LEccHJPst6zFmWL2s0Fu31ddUOkLIflDbWUbBxoCogEQAvD_BwE&gclsrc=aw.ds&hl=ko)에서 회원가입을 한 후에,

<img src="/img/posts/gcp/gcp2.png" style="width: 80%"> 

"사용" 버튼을 누른후 인스턴스 만들기를 선택하면 된다. 그리고 난 뒤에

<img src="/img/posts/gcp/gcp3.png" style="width: 80%"> 

아까 언급했던 것처럼, 오리건, 아이오와, 사우스캐롤라이나와 같이 프리티어를 제공하는 리전을 선택한 뒤에(아마도 서부가 아주 조금이라도 더 빠를 것이다) 머신 구성에서는
 E2를 선택하고

<img src="/img/posts/gcp/gcp6.png" style="width: 80%"> 

엑세스 범위: 모든 클라우드 API에 대해 전체 액세스 허용, HTTP, HTTPS 트래픽 허용을 선택해 준다.

<img src="/img/posts/gcp/gcp4.png" style="width: 80%"> 

머신 유형에서는 프리티어를 제공해주는 e2-micro를 선택하자. 이후 부팅 디스크에서 새로운 SSD 영구 디스크, 크기는 30GB내에서 원하는 만큼 
선택하면 된다.

<img src="/img/posts/gcp/gcp5.png" style="width: 80%"> 

이후 인스턴스를 생성한 뒤 SSH로 접속을 선택하면 흔히 사용했던 bash가 새로운 브라우저 창으로 등장할 것이다.

### MYSQL 설치 및 설정

~~~
apt-get install mariadb-server mariadb-client
/usr/bin/mysql_secure_installation (db초기화 작업)
~~~

나는 우분투 환경이기 때문에 apt-get install mariadb-server mariadb-client로 mysql을 설치했다. 그리고 이 이후에 방화벽을 설정해야한다.

VCP 네트워크의 방화벽 - 방화벽 규칙 만들기를 통해서 새로운 방화벽 규칙을 만들어야 한다 (프로젝트 창 옆에 있는 검색창에 방화벽이라고 검색해도 찾을 수 있다.)

<img src="/img/posts/gcp/mysql1.png" style="width: 80%">

8080포트, tcp/ip 프로토콜에 대해서 0.0.0.0/0 (모든 ip 입력에 대해서 허용, 자신의 IP범위를 적어 넣는편이 당연히 보안에 무조건 좋다)

<img src="/img/posts/gcp/mysql2.png" style="width: 80%">

3306포트에 대해서 (mysql) 접근 허용 방화벽 규칙을 설정해 준다. 이 이후에, 로컬에서만 접속할 수 있도록 바인딩 해 놓은 mysql 설정 파일도 수정해야한다.

~~~
nano /etc/mysql/mariadb.conf.d/50-server.cnf 
~~~

룰 한 뒤에, bind-address= 127.0.0.1 을 지우던지, 주석처리 해야 한다.

<img src="/img/posts/gcp/mysql3.png" style="width: 80%">

이러면 mysql 서버를 사용할 기본적인 설정은 모두 끝났다!

