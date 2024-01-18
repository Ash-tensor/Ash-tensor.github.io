---
layout: post
comments: true
title: "[자바][자료구조] T 메모리구조"
subtitle: 
description: 
date: 2024-01-17
categories: JAVA 자료구조
background: '/img/port.jpg'
---

## [자바][자료구조] T 메모리구조

<img src="/img/posts/JAVA/TMemorySample.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%">


### Main.java

~~~ java
package tmemorysample;
import java.util.ArrayList;
import java.util.List;

public class Main {
    static final int SAMPLE_INTEGER_A = 5;
    static int SAMPLE_INTEGER_B = 3;

    public static int fact(int target) {
        if (target == 0) {
            return 1;
        }
        else {
            int answer = target * fact(target-1);
            return answer;
        }
    }
    public static void main(String[] args) {

        List<Integer> sample = new ArrayList<>();
        sample.add(SAMPLE_INTEGER_A);
        sample.add(SAMPLE_INTEGER_B);

        int a = SolveA.add(sample);

        SolveB solveB = new SolveB();
        int b = solveB.add(sample);
        if (a + b >= SAMPLE_INTEGER_A) {
            System.out.println(fact(a+b));
        }
    }
}
class SolveA {
    public static int add(List<Integer> a){
        int answer = 0;
        for(int i = 0; i < a.size(); i++) {
            answer += a.get(i);
        }
        return answer;
    }
}
~~~ 

### SolveB.java

~~~ java

package tmemorysample;

import java.util.List;

public class SolveB {
    public int add(List<Integer> a){
        int answer = 0;
        for(int i = 0; i < a.size(); i++) {
            answer += a.get(i);
        }
        return answer;
    }
}

~~~