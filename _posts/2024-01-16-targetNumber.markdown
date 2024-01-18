---
layout: post
comments: true
title: "[프로그래머스][PS] 타겟 넘버 :: LEVEL 2 (DP 풀이)"
subtitle: 
description: 
date: 2024-01-16
categories: PS DP 파이썬 프로그래머스 JAVA
background: '/img/port.jpg'
---


## [프로그래머스][PS] 타겟 넘버 :: LEVEL 2 (DP 풀이)

### 문제 설명

문제 설명
n개의 음이 아닌 정수들이 있습니다. 이 정수들을 순서를 바꾸지 않고 적절히 더하거나 빼서 타겟 넘버를 만들려고 합니다. 예를 들어 [1, 1, 1, 1, 1]로 숫자 3을 만들려면 다음 다섯 방법을 쓸 수 있습니다.

-1+1+1+1+1 = 3

+1-1+1+1+1 = 3

+1+1-1+1+1 = 3

+1+1+1-1+1 = 3

+1+1+1+1-1 = 3

사용할 수 있는 숫자가 담긴 배열 numbers, 타겟 넘버 target이 매개변수로 주어질 때 숫자를 적절히 더하고 빼서 타겟 넘버를 만드는 방법의 수를 return 하도록 solution 함수를 작성해주세요.

**제한사항**

1. 주어지는 숫자의 개수는 2개 이상 20개 이하입니다.
2. 각 숫자는 1 이상 50 이하인 자연수입니다.
3. 타겟 넘버는 1 이상 1000 이하인 자연수입니다.

### 서론

일단 프로그래머스에서는 이 문제를 DFS / BFS 문제로 설정해 놓았다. 당연히, 이렇게 풀 수 있다. 
조합을 만들어서, 선택된 값들을 음수 또는 양수로 바꿔서 Sum을 리턴하면 되는 문제다.
하지만 굳이 풀어보지 않더라도 아마 이 방법은 분명 최적해가 아닐 것이다. (DFS, BFS로 풀지 못한다는 말이 아니다!!) 
내 경험상 많은 경우에, 이런 문제의 최적해는 DP로 풀어야 하는 경우가 많다.

### 접근 방법

이 문제는 **동전 거스름돈 문제** 와 유사하다.

일단 각 노드는 양수와 음수만을 가질 수 있으므로 

    양수 부분집합의 합 - 음수 부분집합의 합 = 타겟 넘버
    양수 부분집합의 합 + 음수 부분집합의 합 = 주어진 숫자들의 합

이렇게 부분집합을 나눌 수 있다.

이 방정식을 풀면

    2 * 양수 부분집합이 합 + (음수 부분집합의 합은 소거된다) = 타겟 넘버 + 주어진 숫자들의 합  

이고, 따라서

    양수 부분집합의 합 = (타겟 넘버 + 주어진 숫자들의 합) / 2 라는 결과를 얻을 수 있다.

또한 만약에 조건을 quest = [1,2,3] 이고 targetNumber = 1 인 조건을 생각해 보자.
이를 위 식에 대입해 보면

    양수 부분집합의 합 = ( 1 + 6 ) / 2 로 3.5

3.5가 나오는데 이는 문제의 조건에 위배된다. 당연히 int끼리 더하고 빼는데 어떻게 float이 나오겠는가? 정수를 더하고 빼는데 유리수가 나올 수 없다.
따라서 만약 이런 경우에는 해가 없어서 0을 리턴해야 한다. 이렇게 문제의 조건을 간단하게 양수 부분집합의 합으로만 한정지었다.

이제 DP를 사용할 수 있다.


### 파이썬 코드

~~~ python

def solution(quest, target):
    sum_quest = sum(quest)
    if sum_quest < target or (sum_quest + target) % 2 == 1:
        return 0

    subset_sum = (sum_quest + target) // 2
    dp = [0] * (subset_sum + 1)
    dp[0] = 1

    for num in quest:
        for j in range(subset_sum, num - 1, -1):
            dp[j] += dp[j - num]

    return dp[subset_sum]

~~~

dp 배열은 각 부분집합의 합이 index가 되는 경우이다. 아까의 예시로 돌아가보자. 즉 quest가 [1, 2, 3]이고 targetNumber가 2일때
dp[0] 은 부분집합의 합이 0이 되는 경우이고, dp[1]은 부분집합의 합이 1이 되는 경우이다. 이런 경우에 8번 라인을 보면 dp[0] = 1이라고 초기화 해 놨는데 이는 당연하다

왜냐하면 부분집합의 합이 0이 되는 경우는 단 한가지, 아무것도 선택하지 않는 경우 하나밖에는 없기 떄문이다.

이제 for 루프로 넘어가보면 quest 배열, 즉 조건 [1, 2, 3] 일때를 계속 설명해보자.

num이 1일때 j는 subset_sum(양수 부분집합의 합) 부터 num - 1 까지 역순회를 한다. j는 4이므로
dp[4] = dp[4- num (즉 3)] ..... dp[1] = dp[1] + dp[1 - num(즉 0)] 이 된다. 이는 부분집합의 합이 1인 경우는 부분집합의 합이 [0]이 되는 경우에 1을 더하는 것으로. num이 1일때만 가능하다. 이는 동전 거스르기 문제와 비슷하다고 할 수 있다.

dp[j]를 "j원을 거슬러주는 방법의 수"라고 생각해보자. 그리고 주어진 숫자들을 각각 "동전"이라고 생각했을때 이제 이 "동전"들을 사용하여 "j원"을 만드는 방법의 수를 찾는 것이다.

dp[j]에 dp[j - num]을 더하는 것은, "j원"을 만드는 방법에 "j - num원"을 만드는 방법을 추가하는 것으로 이는 "num원 짜리 동전"을 추가로 사용하여 "j원"을 만드는 새로운 방법을 찾는 것을 의미한다.


### 자바 풀이

~~~ java
class Solution {
    public int solution(int[] quest, int target) {
        int sumQuest = 0;

        for(int i : quest) {
            sumQuest = sumQuest + i;
        }

        int subsetSum = (sumQuest + target) / 2;
        int dp[] = new int[subsetSum + 1];
        dp[0] = 1;

        for(int num : quest) {
            for(int j = subsetSum; j >= num; j--) { // j >= num으로 변경
                dp[j] = dp[j] + dp[j - num];
            }
        }
        return dp[subsetSum];
    }

    public static void main(String[] args) {
        int[] quest = {4,2,1,2};
        int targetNumber = 4;
        Solution sol = new Solution();
        int answer = sol.solution(quest,targetNumber); 
        System.out.println(answer);
    }
}
~~~