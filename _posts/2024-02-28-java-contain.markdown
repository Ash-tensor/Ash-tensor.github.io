---
layout: post
comments: true
title: "[JAVA] 배열에서는 왜 contains() 메소드가 제대로 작동하지 않을까?"
subtitle: equals()와 hashCode() 메소드의 작동 방식.
description: 
date: 2024-02-28
categories: JAVA
background: '/img/port.jpg'
---

## [JAVA] 배열에서는 왜 contains() 메소드가 제대로 작동하지 않을까?

### 문제 설명

자바의 contain() 메소드는 배열에서 제대로 작동하지 않는다. 다시 말하면, 예를 들어서 ArrayList에 int[] 타입의 배열을 넣고, contains() 메소드를 사용하여 같은 값을 가진 배열을 찾아내는 것이 불가능하다.
하지만 ArrayList에 Integer 타입의 배열을 넣고, contains() 메소드를 사용하여 같은 값을 가진 배열을 찾아내는 것은 가능하다.
분명 contains() 메소드도 eqauls() 메소드를 사용하여 값을 비교하도록 되어있는데, ArrayList<Integer>의 경우에는 가능하고, int[]의 경우에는 불가능하다.

왜 가능할까? 

#### 배열의 경우

~~~ java

package boj;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class equal_test {
    public static void main(String[] args) {
    
    // 테스트용 ArrayList 생성
        ArrayList<int[]> testArrayList = new ArrayList<>();
        int[] targetIntArray = {1, 2};

        testArrayList.add(targetIntArray);
        boolean question1 = testArrayList.contains(targetIntArray);
        System.out.println(question1);
        // true 출력

        // 같은 값을 가진 배열을 생성하여 contains() 메소드를 사용
        int[] sameValueArray = {1, 2};
        boolean question2 = testArrayList.contains(sameValueArray);
        
        // false 출력
        System.out.println(question2);
    }
}

~~~


<img src="/img/posts/JAVA/equals/2.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%">

아마 감이 좋다면 String에서 == 연산자와 equals() 메소드의 차이점을 떠올리며 감을 잡을 수 있을 것이다. String의 경우에는 == 연산자를 사용하여 두 String의 주소를 비교하고, equals() 메소드를 사용하여 두 String의 값을 비교했었다.
contain() 역시 equal() 메소드를 사용한다. 그렇다면 왜 같은 값을 가진 배열을 찾아내지 못할까? 

위 코드의 예시를 통해 확인할 수 있듯, 배열의 경우 contains() 메소드를 사용하여 같은 값을 가진 배열을 찾아내는 것이 불가능하다. 디버거를 이용해서 주소를 확인해 보면 다음과 같은데
배열 변수의 주소가 다른 것을 확인할 수 있다. 하지만 배열 변수의 주소가 다르기 때문에 false를 반환하는 것이 아니다. ArrayList의 equals() 메소드는 단순히 배열 변수의 주소만을 비교하지는 않는다. 이는 다음 코드를 확인하면 더욱 명확해진다.

#### ArrayList<Integer> 의 경우

~~~ java

package boj;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class equal_test {
public static void main(String[] args) {
    
    // 테스트용 ArrayList 생성
        ArrayList<ArrayList<Integer>> testArrayList = new ArrayList<>();

    // ArrayList에 Integer 타입의 ArrayList를 넣음
        ArrayList<Integer> targetIntArrayList = new ArrayList<>();
        targetIntArrayList.add(1);
        targetIntArrayList.add(2);
        testArrayList.add(targetIntArrayList);
        
        boolean question1 = testArrayList.contains(targetIntArrayList);
        System.out.println(question1);
    // true 출력

        ArrayList<Integer> sameValueArray = new ArrayList<>();
        
    // 같은 값을 가진 ArrayList를 생성하여 contains() 메소드를 사용
        sameValueArray.add(1);
        sameValueArray.add(2);
        boolean question2 = testArrayList.contains(sameValueArray);
        System.out.println(question2);
    // true 출력
    }
}

~~~ 

위 코드를 실행해 보면 알 수 있듯, 배열과는 달리, 같은 값을 가진 ArrayList<Integer> 의 경우에는 둘 다 true를 반환하는 것을 확인할 수 있다. 하지만 이를 디버거로 확인해 보면, 두 ArrayList<Integer>의 
주소는 다르다. 그런데도 불구하고, contains() 메소드는 두 ArrayList<Integer>의 값을 비교하여 같은 값을 가진 것으로 판단한다.


<img src="/img/posts/JAVA/equals/4.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%">

이는 ArrayList<Integer>의 경우에, int 값이 아닌 Integer 객체를 비교하기 때문이다. 위 사진을 보면 코드상에서는 분명, 둘 다 각각 int(1, 2)를 집어넣었는데도 불구하고
Integer 객체의 경우에 두 ArrayList 모두 똑같은 객체의 Integer 객체를 가리키고 있는 것을 확인할 수 있다. 이 이전의 배열 사진을 확인해 보면 각각의 int는 다른 객체를 가리키고 있는 것을 확인할 수 있다.
자바에서 기본 자료형 (int, char 등)은 객체로 취급받지 않고 특정한 값으로 취급받는다. 하지만 Integer, Character 등은 객체로 취급받고, java의 Heap 영역에 적재되어 있다. 
따라서 아무리 많은 Integer, Character, String 등의 객체를 사용하더라도, 같은 값을 가지고 있으면 같은 주소를 가지는 동일한 객체를 가리키고 있는 것이다.

또한 ArrayList<Integer>의 경우에는 equals() 메소드를 사용하여 두 객체의 값을 비교한다. equals() 메소드는 객체의 값을 비교하기 위해 equals() 메소드를 오버라이딩하여 사용한다.

그래서 ArrayList<Integer>의 경우에는 contains() 메소드를 사용하여 같은 값을 가진 객체를 찾아낼 수 있다!!
