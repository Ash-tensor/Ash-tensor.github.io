---
layout: post
comments: true
title: "[HTML/JAVASCRIPT] 아주 간단한 버켓리스트 페이지 제작"
subtitle: 
description: 
date: 2024-01-10
categories: WEB
background: '/img/port.jpg'
---

2024년이 시작한지도 벌써 10일이 지나긴 했지만, 새해를 맞아 새해의 목표를 세우는 것은 필요한 것 같다. 막연하게 생각했었던 새해 목표도 구체적으로 정리하고, 일일 목표를 세우기 위해서라도.

그래서 간단한 2024년 버켓리스트 페이지를 제작했다! 제작하면서 기초적인 HTML 태그, javascript, css 지식을 정리할 수 있는 좋은 기회였다.

# 완성된 버켓리스트


<iframe src="https://ash-tensor.github.io/bucketlist/" width="100%" height="800"></iframe>

기능이라고 할 것도 없이 간단한데, 클릭을 하면 완료 도장이 찍히는 것이다. 

## 코드


    <!DOCTYPE html>
    <html>

    <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width">
    <title>2024 버킷리스트</title>
    <link href="style.css" rel="stylesheet" type="text/css">
    <style>
        @import url("https://fonts.googleapis.com/css2?family=Hahmlet:wght@300&display=swap");
        * {
        font-family: "Hahmlet", serif;";
        }
        .bg {
        background-image: url('https://s3.ap-northeast-2.amazonaws.com/materials.spartacodingclub.kr/bucketList/bg-grid.png');
        background-position: center;
        background-size: cover;
        margin: auto;
        width: 360px;
        }
        .title {
        background-color: tomato;
        border-radius: 10px;
        color: white;
        text-align: center;
        padding: 4px 14px;
        }

        .msg {
        margin: -5px 0 15px;
        }

        .bucket {
        width : 160px;
        height: 160px;
        background-position: center;
        background-size: cover;
        }

        .img1 {
        background-image: url('https://s3.ap-northeast-2.amazonaws.com/materials.spartacodingclub.kr/bucketList/bucket-red.png');
        }
        .img2 {
        background-image: url('https://s3.ap-northeast-2.amazonaws.com/materials.spartacodingclub.kr/bucketList/bucket-lightred.png');
        }

        .class = 'msg center' {
        color: black !important;
        }

    </style>
    <link rel="stylesheet" type="text/css" href="https://s3.ap-northeast-2.amazonaws.com/materials.spartacodingclub.kr/bucketList/sparta-bucket2.css" />

    </head>

    <body class="bg center">
    <h1 class="title">2024년 신년계획</h1>
    <p class="msg center">올해 안에 모두 달성하는게</p>
    <p class="msg center">목표!</p>
    <div class="flex-row wrap">
        <div class='bucket img1 center'>자바 마스터하기</div>
        <div class='bucket img2 center'>빅데이터 분석기사 합격하기</div>
        <div class='bucket img2 center'>취업 뽀개기</div>
        <div class='bucket img1 center'>운동하기!</div>
        <div class='bucket img1 center'>알고리즘 통달하기</div>
        <div class='bucket img2 center'>여유를 가지고 여행하기!</div>
    </div>
    
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

## 코드 설명

    <meta name="viewport" content="width=device-width">
        // 뷰포트(viewport)의 너비를 기기의 너비로 설정하여 
        모바일 화면에 최적화된 보기 경험을 제공한다. 
        반응형 웹 디자인을 위해 필요하다.

뷰포트(viewport) : 사용자가 웹사이트나 애플리케이션을 통해 볼 수 있는 영역을 의미한다. 쉽게 말해, 사용자의 화면에 실제로 보이는 부분. 예를 들어, 스마트폰이나 태블릿, 랩탑 등 다양한 기기에서 웹사이트를 방문했을 때, 그 기기의 화면 크기에 맞게 보이는 웹페이지의 영역이 뷰포트에 해당한다.

    @import url("https://fonts.googleapis.com/css2?family=Hahmlet:wght@300&display=swap");
    * {
    font-family: "Hahmlet", serif;";
    }
    //구글 폰트를 적용하고 * 즉 모든 요소에 대해서 "Hahmlet" 폰트를 적용함

    .bg {
        /* 배경 이미지 및 스타일 설정 */
    }
    .title {
        /* 타이틀 영역의 배경색, 글자색, 텍스트 정렬 등의 스타일을 정의 */
    }
    .msg {
        /* 메시지 영역의 스타일을 정의 */
    }
    .bucket {
        /* 버킷리스트 아이템의 크기 및 배경 이미지 관련 스타일을 정의 */
    }
    .img1, .img2 {
        /* 특정 버킷리스트 항목에 대한 배경 이미지를 정의 */
    }

    <body class="bg center">
        .bg에서 정의한 CSS 스타일과 center 스타일 속성을 사용함. 
        // center 속성은 외부 css 파일에 정의되어 있음

        /* 외부 CSS 시트에 적용된 .center 클래스
        
        .center {
            display: flex; << 플랙스박스 레이아웃을 적용함
            flex-direction: column; << 주 축을 column으로 설정
            align-items: center; << 교차 축에서 중앙 정렬
            justify-content: center; << 주 축에서 중앙 정렬
        } */


* 플렉스박스(Flexbox) 레이아웃은 CSS3에서 도입된 강력한 레이아웃 도구로, 복잡한 레이아웃을 쉽게 만들 수 있게 해주는 방식. 주로 1차원 레이아웃을 위해 사용되며, 즉, 한 번에 한 축(가로 또는 세로)을 따라 요소를 배치하는 데 최적화되어 있음

플렉스박스를 사용하면 요소들 간의 간격과 정렬을 유연하게 조정할 수 있어서, 다양한 화면 크기와 해상도에 대응하는 반응형 웹 디자인을 구현하기 쉽다. 플렉스박스는 특히 요소의 크기가 불명확하거나 동적일 때 유용하며, 복잡한 정렬이나 분배를 단순한 코드로 해결할 수 있게 해 준다.

플렉스박스의 주요 특징:

1. **컨테이너와 아이템**: 플렉스 컨테이너(Flex Container) 내부에 플렉스 아이템(Flex Items)들이 배치됨. 컨테이너에 `display: flex;` 또는 `display: inline-flex;` 속성을 적용함으로써 플렉스박스 레이아웃이 활성화된다.

2. **주 축과 교차 축**: 플렉스박스 레이아웃에서는 주 축(Main Axis)과 교차 축(Cross Axis)이라는 두 개의 축이 있다. `flex-direction` 속성을 통해 주 축을 설정할 수 있으며, 아이템들은 이 주 축을 따라 배치됨.

3. **정렬**: `justify-content`, `align-items`, `align-self` 등의 속성을 통해 아이템들의 수평, 수직 정렬을 쉽게 조절할 수 있음.

4. **유연성**: `flex-grow`, `flex-shrink`, `flex-basis` 속성을 사용해 아이템들이 차지하는 공간의 비율을 조절할 수 있으며, 컨테이너의 남은 공간을 아이템들이 어떻게 분배할지 결정할 수 있다.

플렉스박스 레이아웃은 웹 페이지의 다양한 구성 요소에 유용하게 적용될 수 있으며, 특히 **메뉴, 카드 레이아웃, 그리드 시스템** 등을 만들 때 자주 사용된다.

    <h1 class="title">2024년 신년계획</h1>

h1 태그 : 가장 중요한 제목을 나타냄


    <p class="msg center">올해 안에 모두 달성하는게</p>
    <p class="msg center">목표!</p>


p 태그 : 문단을 나타냄, msg, center 클래스가 적용되어 있음.

    <div class="flex-row wrap">
    flex-row, wrap 클래스를 적용한다. flex-row와 wrap은 외부 css 시트에 적용되어 있음

    */ 외부 css 시트에 적용된 .flex-row와 .wrap 클래스
    
    .flex-row {
        display: flex;
        flex-direction: row;
        align-items: center;
        justify-content: center;
    }

    .wrap {
        flex-wrap: wrap;
    } */


div 태그 : <div> 태그는 블록 레벨 컨테이너로, 여기서는 여러 버킷리스트 항목들을 감싸고 있음. class="flex-row wrap" 속성은 CSS 플렉스박스(flexbox) 레이아웃을 적용하여 항목들을 행(row)으로 정렬하고, 필요한 경우 여러 줄로 감싸(wrap)서 표시한다.

.flex-row

1. display: flex;: 이 속성은 해당 요소를 플렉스 컨테이너로 만듭니다. 플렉스 아이템들이 플렉스박스 레이아웃을 사용하여 배치됩니다.
2. flex-direction: row;: 플렉스 아이템들이 수평 방향(가로)으로 나열되도록 설정합니다.
align-items: center;: 플렉스 아이템들이 교차 축(이 경우에는 세로 축)에서 중앙에 위치하도록 설정합니다.
3. justify-content: center;: 플렉스 아이템들이 주 축(이 경우에는 가로 축)에서 중앙에 위치하도록 설정합니다.

.wrap

.wrap 클래스는 플렉스 아이템들이 플렉스 컨테이너 내에서 여유 공간이 부족할 때, 다음 줄로 넘어가도록 설정합니다:

flex-wrap: wrap; 이 속성은 플렉스 아이템이 한 줄에 모두 표시될 수 없을 때, 다음 줄로 감싸지도록(줄바꿈) 설정

이 두 클래스가 적용되어 버킷리스트가 한줄에 두 개씩 표시되고, 플렉스 아이템이 한 줄에 표시될 수 없으므로 다음 줄로 감싸지도록(줄바꿈)이 됨

자바스크립트 코드는 다음 포스트에 자세히 정리하겠다.
