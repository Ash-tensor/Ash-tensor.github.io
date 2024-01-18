---
layout: post
comments: true
title: "[JAVA] 인터페이스와 추상클래스의 차이"
subtitle: 
description: 
date: 2024-01-07
categories: JAVA
background: '/img/port.jpg'
---

추상 클래스와 인터페이스는 모두 자바에서 객체지향의 원칙 중 추상화를 구현하기 위해 사용되는 개념이지만 몇가지 중요한 차이가 존재한다. 솔직히 내 생각에는, 자바라는 언어가 어쩌면 극단적으로 객체지향을 중시한다는 것을 보여주는 예시라고 생각한다. 파이썬에서는 굳이 인터페이스를 사용하지 않는 경우가 많은 것 같다. "__call__" 처럼 간단한 인터페이스라면 객체를 그냥 파이썬 함수처럼 호출해서 사용하는 경우도 있고. 

인터페이스를 배울 때 (적어도 나는) 추상 클래스와는 다르게 다중 상속이 가능하다는 것이 기억나는데 이는 객체지향에 익숙하지 않으면 추상 클래스와 인터페이스가 어떤 일을 하는지 정확히 이해하기 힘들기 때문이라고 생각한다.

## 인터페이스

인터페이스는 추상 클래스와는 별도로 **구현 객체가 같은 동작을 한다는 것을 보장하기 위해 초점을 두는 것이 추상 클래스와는 다르다.** 예를 들어서, 리모컨이라는 추상 클래스가 존재한다고 할 때, TV리모컨과 에어컨 리모컨은 하는 역할이 다를 것이다. 

    abstract class RemoteControl {
        abstract void turnOn();
        abstract void turnOff();
        abstract void volumeUp(); << 에어컨 리모컨에서는 구현할 수 없다.
    }

    class TVRemote extends RemoteControl {
        void turnOn();
        void turnOff();
        void volumeUp();
    }

    class AirconditionerRemote extends RemoteControl {
        void turnOn();
        void turnOff();
        void volumeUp(); << 구현할 수 없음
    }

이런 경우에 인터페이스를 사용하는 것이다. 공통적인 기능의 보장! TurnOn과 TurnOff는 공통적인 기능이므로, 이런 식으로 구현이 될 수 있겠다.

    Interface Volume {
        void volumeUp();
        void volumeDown();
    }

    class TVRemote extends RemoteControl implements Volume {
        void turnOn();
        void turnOff();
        void volumeUp();
    }

이렇게 동작의 메서드를 각각 인터페이스마다 분리하여 설계함으로서 조금 더 구조적이고 더욱 추상적인 객체를 설계할 수 있다. 

### 구현 예제

#### IRemoteControlable 인터페이스

    public interface IRemoteControlable {
    void turnOn();
    void turnOff();
    void channelUp();
    void channelDown();
    void volumnUp();
    void volumnDown();
    void putChannel(int value);
    }

#### Controller 클래스

    package jan5th;
    import java.util.Scanner;

    public class Controller implements IRemoteControlable {
        Television model;

        public Controller(Television model){
            this.model = model;
        }

        public boolean analyzer(int value, Scanner hand){

    //"1. TV 켜기, 2. TV 끄기, 3. 채널 올리기, 4. 채널 내리기, " +
    //"5. 볼륨 올리기, 6. 볼륨 내리기, 7. 특정 채널로 이동, 0. 종료";
            
            switch (value){

                case 1:{
                    this.turnOn();
                    break;
                }
                case 2: {
                    this.turnOff();
                    break;
                }
                case 3 : {
                    this.channelUp();
                    break;
                }
                case 4 : {
                    this.channelDown();
                    break;
                }
                case 5 : {
                    this.volumnUp();
                    break;
                }
                case 6 : {
                    this.volumnDown();
                    break;
                }
                case 7 : {
                    System.out.println("입력할 채널을 선택해주세요");
                    int inputedChannel = hand.nextInt();
                    this.putChannel(inputedChannel);
                    break;
                }
                case 0 : {
                    System.out.println("TV를 그만 봅니다.");
                    return false;
                }
            }
            return true;
        }

        public void turnOn() {
            model.on = true;
            System.out.println("TV 전원 : ON");
        }

        @Override
        public void turnOff() {
            model.on = false;
            System.out.println("TV 전원 : OFF");
        }

        @Override
        public void channelUp() {
            if(model.on == false) {
                System.out.println("TV STATE : OFF");
            }
            else {
                model.channel += 1;
                System.out.printf("현재 채널 : %d\n", model.channel);
            }
        }

        @Override
        public void channelDown() {
            if(model.on == false) {
                System.out.println("TV STATE : OFF");
            }
            else {
                if (model.channel == 1) {
                    System.out.println("0번 채널은 없습니다.");
                } else {
                    model.channel -= 1;
                    System.out.printf("현재 채널 : %d\n", model.channel);
                }
            }
        }
        @Override
        public void volumnUp() {
            if(model.on == false) {
                System.out.println("TV STATE : OFF");
            }
            else {
                model.volume += 1;
                System.out.printf("현재 볼륨 : %d\n", model.volume);
            }
        }

        @Override
        public void volumnDown() {
            if(model.on == false) {
                System.out.println("TV STATE : OFF");
            }
            else {
                if (model.volume == 0) {
                    System.out.println("볼륨을 0 아래로 내릴 수 없습니다.");
                } else {
                    model.volume -= 1;
                    System.out.printf("현재 볼륨 : %d\n", model.volume);
                }
            }
        }
        @Override
        public void putChannel(int value) {
            if(model.on == false) {
                System.out.println("TV STATE : OFF");
            }
            else {
                if (value <= 0) {
                    System.out.println("그런 채널은 없습니다.");
                } else {
                    model.channel = value;
                    System.out.printf("현재 채널 : %d\n", model.channel);
                }
            }
        }
    }


#### Human 클래스

    package jan5th;

    import java.util.Scanner;

    public class Human {
        Controller model;

        public Human(Controller controller){
            this.model = controller;
        }
        void input(Scanner Hand){
            boolean condition = true;
            while(condition){
                System.out.println("1. TV 켜기, 2. TV 끄기, 3. 채널 올리기, 4. 채널 내리기, " +
                        "5. 볼륨 올리기, 6. 볼륨 내리기, 7. 특정 채널로 이동, 0. 종료");
                int value = Hand.nextInt();
                condition = this.model.analyzer(value, Hand);
            }
        }
    }


#### 메인 클래스

    package jan5th;

    import java.util.Scanner;

    public class Main {
        public static void main(String[] args) {
            Television appleTV = new Television();
            Controller appleTVController = new Controller(appleTV);
            Human me = new Human(appleTVController);
            me.input(Hand);
        }
    }

