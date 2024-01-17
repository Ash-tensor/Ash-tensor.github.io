---
layout: post
comments: true
title: "[백준][PS] 회사에 있는 사람 :: 실버 5(자료구조)"
subtitle: 
description: 
date: 2024-01-17
categories: PS 백준 JAVA 자료구조
background: '/img/port.jpg'
---

# [백준][PS] 회사에 있는 사람 :: 실버 5(자료구조)

## 문제 설명

상근이는 세계적인 소프트웨어 회사 기글에서 일한다. 이 회사의 가장 큰 특징은 자유로운 출퇴근 시간이다. 따라서, 직원들은 반드시 9시부터 6시까지 회사에 있지 않아도 된다.

각 직원은 자기가 원할 때 출근할 수 있고, 아무때나 퇴근할 수 있다.

상근이는 모든 사람의 출입카드 시스템의 로그를 가지고 있다. 이 로그는 어떤 사람이 회사에 들어왔는지, 나갔는지가 기록되어져 있다. 로그가 주어졌을 때, 현재 회사에 있는 모든 사람을 구하는 프로그램을 작성하시오.

**입력**

첫째 줄에 로그에 기록된 출입 기록의 수 n이 주어진다. (2 ≤ n ≤ 106) 다음 n개의 줄에는 출입 기록이 순서대로 주어지며, 각 사람의 이름이 주어지고 "enter"나 "leave"가 주어진다. "enter"인 경우는 출근, "leave"인 경우는 퇴근이다.

회사에는 동명이인이 없으며, 대소문자가 다른 경우에는 다른 이름이다. 사람들의 이름은 알파벳 대소문자로 구성된 5글자 이하의 문자열이다.

**출력**

현재 회사에 있는 사람의 이름을 사전 순의 역순으로 한 줄에 한 명씩 출력한다.

## 제한시간

1초, 메모리제한 256MB


### 접근 방법

솔직히 엄청 쉬운 문제다. 누구나 구현할 수 있고, 전공자가 아니더라도 누구나 이 문제를 구현하는데는 문제가 없을 것이다. 맵 자료형을 써도 되고, 파이썬에서는 딕셔너리를 쓰거나, 해시 맵을 쓰거나, 집합 자료구조를 사용해도 문제는 무리 없이 풀린다. 하지만 자료구조에 대한 이해가 없이 배열과 같은 선형 자료구조를 사용하면 **시간 초과**를 마주치는 문제다. 개인적으로 이 문제는 좋은 문제라고 생각해서 가져왔다.

#### 잘못된 접근

~~~ java
package boj;

import java.lang.reflect.Array;
import java.util.*;

public class boj7785 {

    public static void main(String[] args) {
        List<String> coworkerList = new ArrayList<>();
        Scanner scanner = new Scanner(System.in);
        int coworkerNumber = scanner.nextInt();

        for(int i = 0; i < coworkerNumber; i++) {
            String coworkerName = scanner.next();
            String sign = scanner.next();
            if (sign.equals("enter")){
                coworkerList.add(coworkerName);
            }
            else {
               coworkerList.remove(coworkerName);
            }
        }

        Collections.sort(coworkerList, Collections.reverseOrder());
        for(String coworker : coworkerList) {
            System.out.println(coworker);
        }
    }
}
~~~

이렇게 풀어도 답은 문제없이 나온다. 하지만 시간초과로 문제 해결이 실패하게 된다. 구현방법을 보면 enter이면 ArrayList<String>에 동료의 이름을 집어 넣고, 만약에 leave라는 문자열을 마주치면 ArrayList에서 해당 문자열을 삭제한다.

여기서 문제가 발생하는데, 배열과 같은 선형 자료구조는 중간 노드의 삽입과 삭제가 빈번하게 일어날 때, 성능이 매우 나빠진다. 

왜냐하면 연속된 메모리 공간에 요소들을 저장하고 있기 때문에 중간에 요소를 추가하거나, 삭제하면 해당 위치 이후의 모든 요소를 이동시켜야만 한다. 이런 경우에 노드의 추가와 삭제에 소요되는 시간 복잡도는 O(n)이 된다. 이렇게 추가적으로 시간이 매우 많이 소요되기 때문에 1초라는 시간으로는 문제를 풀이할 수 없다. 

이런 경우에는 순서가 상관없는 자료구조로 문제를 풀이해야 한다. 대표적으로는 set 자료형이 있다. 다음은 HashSet 자료형으로 풀이한 자바 코드다.


### 자바 풀이

~~~ java
public class boj7785 {
    public static void main(String[] args) {
        Set<String> coworkerSet = new HashSet<>();

        Scanner scanner = new Scanner(System.in);
        int coworkerNumber = scanner.nextInt();

        for (int i = 0; i < coworkerNumber; i++) {
            String coworkerName = scanner.next();
            String sign = scanner.next();

            if (sign.equals("enter")) {
                coworkerSet.add(coworkerName);
            } else {
                coworkerSet.remove(coworkerName);
            }
        }

        List<String> coworkerList = new ArrayList<>(coworkerSet);
        Collections.sort(coworkerList, Collections.reverseOrder());

        for (String coworker : coworkerList) {
            System.out.println(coworker);
        }
    }
}
~~~ 