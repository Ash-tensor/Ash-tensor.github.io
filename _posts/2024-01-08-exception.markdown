---
layout: post
comments: true
title: "[JAVA] 예외처리 - throw, throws, throwable"
subtitle: Checked Error - Unchecked Error
description: 
date: 2024-01-08
categories: JAVA
background: '/img/port.jpg'
---

## 예외처리

자바는 솔직히 예외처리를 아주 빡빡하게 처리하는 언어다. 다른 언어들이 특정 예외가 일어났을 때를 열거해서 (divideByZero error, Stack overflow error 등) 제외하는 것에 비해서 조금은 특이하다.

물론 C 처럼 예외 처리를 아예 강제하지 않는 언어와 비교하려는 것은 아니다. Python이나 C++, C# 모두 try - catch - finally와 같은 예외 처리를 지원하고, 자바와 사용되는 예약어만 다들 뿐 그 궤는 비슷하다.

하지만 널리 사용되는 언어인 Javascript와 Python에 비교해 봐도, 자바처럼 Checked 에러와 Unchecked 에러를 나눠서 구현을 강제해서 구분하는 언어는 충분히 드물다고 할 수 있다.

### Checked Error - Unchecked Error

    class ExceptionExample {
        // 예외를 던질 수 있는 메서드
        public void methodWithThrows() throws CustomException { << 에러 열거
            System.out.println("메서드에서 예외를 던집니다.");
            throw new CustomException("Custom 예외 발생");
        }

        // 호출하는 코드
        public static void main(String[] args) {
            ExceptionExample example = new ExceptionExample();

            try {
                example.methodWithThrows();
            } catch (CustomException e) {   << 열거한 에러 처리
                System.out.println("예외를 처리합니다: " + e.getMessage());
            }
        }
    }

    // CustomException 정의
    class CustomException extends Exception {
        public CustomException(String message) {
            super(message);
        }
    }

여기서 세번째 라인의 throws가 바로 Checked Error를 열거하는 예약어다. 보통 throws 키워드를 설명할 때

>> [Throws 키워드는 메소드 선언부 끝에 작성되어 메소드에서 처리하지 않은 예외를 호출한 곳으로 떠넘기는 역할을 합니다.]

라고 설명하는데, 이 말을 처음 들을 때 이해가 쉽지는 않다(고 생각한다).

throws 키워드는 Checked Error를 열거하는 예약어로, 만약 이 메소드를 호출하는 메소드가 있다면 무조건 예외처리를 해야 컴파일이 가능하다는 뜻이다.

즉 위 예시 코드로 설명해 보자면, 예외를 던질 수 있는 메서드에서 customException을 throws 예약어로 열거했으므로 //호출하는 코드 부분에서 무조건적으로 

    catch(CustomException e)

부분이 없으면 실행을 아예 거부해버린다.

<img src="/img/posts/JAVA/throws.PNG" style="width: 80%">

이런식으로. 

### throws가 없는 프로그램

    class ExceptionExample {
        public void methodWithoutThrows() {
            System.out.println("예외 테스트");
            ExceptionExample example = new ExceptionExample();
            example.methodWithoutThrows();
        }

        // 호출하는 코드
        public static void main(String[] args) {
            ExceptionExample example = new ExceptionExample();
            example.methodWithoutThrows();
            }
        }


throws를 제거한 코드다. try - catch 문이 없어도 문제 없이 컴파일 가능한 모습을 볼 수 있다. methodWithoutThrows는 예외를 던지지 않으므로 호출하는 코드에서는 예외 처리를 할 필요가 없다. 

이렇게 throws를 사용하면 메서드를 사용하는 측에게 어떤 예외가 발생할 수 있는지 명시함으로써 명확한 예외 처리를 유도할 수 있다... 곤 하지만, 솔직히 아무리 문법적으로 강제해도 제대로 하지 않을 사람은 안 한다. 아예 JVM으로 에러를 던져버리는 것이다. 

    
    // main 메소드 throws

    public class Example {
        public static void main(String[] args) throws Exception {
            // 예외가 발생할 수 있는 코드
        }
    }

이런 식으로, main 메소드에서 throws를 사용하는 것은 에러를 JVM에게 전가한다는 뜻이다. Main 메소드보다 더 상위의 메소드는 JVM밖에는 없기에, 에러를 던져도 JVM은 에러 메시지만 출력하고 프로그램을 종료한다.

main 메서드에서 throws를 사용할 때는 주로 런타임 예외인 RuntimeException 또는 Exception과 같은 일반적인 예외를 사용하며, 이를 통해 예외 처리를 뒷단으로 미루는 경우가 있다.


>> [throws <unchecked exception(runtime exception)> 키워드가 붙어있는 메서드는 꼭 try 블록 내부에서 호출되지 않더라도 컴파일이 됩니다. 하지만 매우 문제가 있는 코드입니다.]


    // RuntimeException throws

    public class Example {
        // 예외를 던질 수 있는 메서드
        public static void methodWithThrows() throws RuntimeException {
            System.out.println("메서드에서 예외를 던집니다.");
            throw new RuntimeException("런타임 예외 발생");
        }

        // 호출하는 코드
        public static void main(String[] args) {
            // 메서드 호출 시 try-catch 블록 없음
            methodWithThrows();
            
            // 아래 코드는 실행되지 않음 (Unreachable code 에러 발생)
            System.out.println("메서드 호출 후의 코드");
        }
    }

위의 코드에서 methodWithThrows는 RuntimeException을 던질 수 있는데, 호출하는 main 메서드에서는 해당 예외를 처리하는 try-catch 블록이 없다. 그러나 컴파일은 문제없이 진행된다.

하지만 실행 시에는 methodWithThrows에서 발생한 예외가 main 메서드로 전파되어, "Unreachable code" 에러가 발생하면서 프로그램이 비정상적으로 종료된다. 

