---
layout: post
comments: true
title: "[HTML/JAVASCRIPT] 아주 간단한 버켓리스트 페이지 제작 - 도장찍기 부분 제작 (2/2)"
subtitle: 
description: 
date: 2024-01-12
categories: WEB
background: '/img/port.jpg'
---

javascript를 이용해 도장을 찍는 기능을 구현했다. javascript는 동적 웹 페이지를 만들기 위한 프로그래밍 언어로, 웹 브라우저에서 실행된다. 깃허브 페이지는 정적 웹 페이지를 호스팅 하는데 쓰이지만, 동적 서버 코드를 사용하는 것이 아니기 때문에 상관 없다.

## 완성된 버켓리스트

<iframe src="https://ash-tensor.github.io/bucketlist/" width="100%" height="800"></iframe>

간단한 완료 도장이 찍히는 코드.

### 코드
    
    <script>
        // 페이지 로드 시 이전 상태 복원
        document.addEventListener("DOMContentLoaded", (event) => {
        const buckets = document.querySelectorAll(".bucket");
        buckets.forEach((bucket, index) => {
            // 로컬 스토리지에서 상태 읽기
            const isDone = localStorage.getItem("bucket" + index) === "done";
            if (isDone) {
            bucket.classList.add("done");
            }
        });
        });

        // 버킷 리스트 클릭 이벤트
        const buckets = document.querySelectorAll(".bucket");
        buckets.forEach((bucket, index) => {
        bucket.addEventListener("click", function () {
            // 클래스 토글
            bucket.classList.toggle("done");

            // 로컬 스토리지에 상태 저장
            if (bucket.classList.contains("done")) {
            localStorage.setItem("bucket" + index, "done");
            } else {
            localStorage.setItem("bucket" + index, "");
            }
        });
        });
    </script>
    </body>

    </html>

### 코드 설명

    document.addEventListener("DOMContentLoaded", (event) => { ... }):

이 코드는 웹 페이지의 HTML이 완전히 로드되고 나서 실행된다. 이를 통해 페이지가 로드되자마자 필요한 작업을 수행할 수 있는데, 여기서는 로드되고 난 후, 이전에 저장된 상태를 불러오는 것을 맡고 있다.

    const buckets = document.querySelectorAll(".bucket");

웹 페이지에서 클래스 이름이 "bucket"인 모든 요소를 선택하고, 그 결과를 "buckets"라는 상수에 저장함.

    buckets.forEach((bucket, index) => { ... })

"buckets"라는 변수에 저장된 모든 요소에 대해 작업을 수행함.

    const isDone = localStorage.getItem("bucket" + index) === "done";:

로컬 스토리지에서 "bucket" + index(인덱스 번호)라는 키를 가진 아이템을 가져와서, 그 값이 "done"인지 확인함.

로컬 스토리지는 웹 브라우저에서 제공하는 간단한 키-값 형태의 클라이언트 측 웹 스토리지 메커니즘이다. 웹 앱에서 데이터를 로컬에 저장하고 나중에 사용자가 페이지를 새로고침하거나 다시 방문할 때도 그 데이터를 유지할 수 있게 한다.

    if (isDone) { bucket.classList.add("done"); }:

만약 위에서 확인한 값이 "done"이라면, 해당 버킷에 "done" 클래스를 추가함.

    bucket.addEventListener("click", function () { ... }):

이 코드는 각 버킷에 클릭 이벤트 리스너를 추가한다. 이를 통해 사용자가 버킷을 클릭할 때마다 특정 함수가 실행된다.

    bucket.classList.toggle("done");:

클릭 이벤트가 발생하면, 해당 버킷의 클래스 리스트에서done" 클래스를 토글(있으면 제거, 없으면 추가)

if (bucket.classList.contains("done")) { localStorage.setItem("bucket" + index, "done"); } else { localStorage.setItem("bucket" + index, ""); }:

버킷에 "done" 클래스가 있으면 로컬 스토리지에 "done"이라는 값을 저장하고, 없으면 빈 문자열을 저장한다.


이렇게 페이지 로드 시 웹 브라우저의 로컬 스토리지를 확인하여 버킷 리스트의 상태를 복원하고, 사용자가 버킷을 클릭하여 상태를 변경할 때마다 그 상태를 로컬 스토리지에 저장하는 역할을 해 도장을 찍는 간단한 기능을 구현할 수 있다!

### 마무리

2024년이 시작한지 얼마 안 지났는데, 하루하루가 굉장히 빠르다는 느낌이 든다. 아침에 일어나고 일과가 끝나고 집에 돌아가면 여섯시 - 일곱시 가량인데, 하루하루 목표를 위해서 열심히 달려야 간단하게 제작한 버켓리스트 페이지에 도장을 찍을 수 있을 것 같다.

열심히 해 보자고!
