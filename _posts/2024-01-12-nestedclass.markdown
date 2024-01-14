---
layout: post
comments: true
title: "[JAVA] 중첩 클래스"
subtitle: 
description: 
date: 2024-01-12
categories: JAVA
background: '/img/port.jpg'
---

자바의 중첩 클래스(Nested Class)는 다른 클래스 내부에 정의된 클래스를 말한다. 중첩 클래스는 외부 클래스의 멤버로 간주되고, 외부 클래스의 멤버 변수 및 메서드에 쉽게 접근할 수 있다. 보통 논리적으로 관련된 클래스를 그룹화하는데 사용된다.

**상속과 같은 개념의 확장 관계는 아니지만 어떤 개념의 상하관계를 개념적으로 구현해야 하는 경우에 사용되는데** 예를 들어서 국적 또는 국민은 국가라는 개념의 확장이 아니므로 상속 관계로 표현이 불가능하기 때문이다.

구현 포인트는 다음과 같다.

1. 예시 코드에서 Korean 및 American, NorthKorean 은 각 국가의 중첩 클래스로 작성하였다. Korean.Korean, US.American은 인터페이스 **ILibertyToMove**의 구현체이기 때문에 **tourAbroad(Nation nation) 및 submittingPassport 메소드**가 존재하기 때문에 여행이 가능하고, NorthKorea.NorthKorean은 구현체가 아니라 해외 여행 및 여권 제출이 불가능하다.

2. 마찬가지로 Korea, US, NorthKorea 모두 추상 클래스 Nation을 상속받았지만, Korea, US는 인터페이스 **IVisitable**의 구현체이여서 Korea.Korean, US.American이 서로의 국가를 방문 가능하다. 하지만 NorthKorean은 IVisitable의 구현체가 아니기 때문에 타 국가가 해당 국가의 방문이 불가능하다.

3. 각 국가는 중첩 클래스로 **.administraionBureau, .ImmigrationBureau**를 가지고(싱글톤이다), 각 Bureau는 추상 클래스인 Bureau를 상속하고, **ImmigrationBureau는 IImmigrationBureau라는 인터페이스**를 상속받아 **visa 메소드와 immigrationScreening 메소드**와 같은 일반적인 Bureau가 아닌 이민청의 일을 할 수 있게 구현했다.


이번 구현을 통해서 타입 캐스팅을 하며 인터페이스가 왜 필요한지 절실히 깨닫는 계기가 되었다. 구현이 부족한 부분이 많지만 일단 이 정도로 마무리할 생각이다.

자세한 코드는 [깃허브 주소](https://github.com/Ash-tensor/studyingJava.git)에 업로드했다.


## 구현 예시

### Main

~~~java
package jan11st;

public class Main {
    public static void main(String[] args) {

        // 싱글톤이기 때문에 Korea, US, NorthKorea 객체는 생성하지 않아도 됨

        System.out.println("객체 생성");
        Korea.Korean A = new Korea.Korean("한국인", "Male", "KOREA");
        US.American B = new US.American("미국인", "Female", "US");
        NorthKorea.NorthKorean C = new NorthKorea.NorthKorean("북한인", "Male", "NorthKorea");

        A.birthRegistration();
        B.birthRegistration();
        C.birthRegistration();


        System.out.printf("현재 %s의 위치 : %s\n", A.name, A.region);
        System.out.printf("현재 %s의 위치 : %s\n", B.name, B.region);

        System.out.println("\n* 한국인의 미국 방문 상황 :");

        A.tourAbroad(US.getInstance());

        System.out.println("\n* 미국인의 한국 방문 상황 :");

        B.tourAbroad(Korea.getInstance());
        // C는 ILibertyToMove(이동할 자유)를 구현하지 않기 때문에 .tourAbroad 할 수 없다.
        System.out.println("\n* 북한 방문 상황 :");

        // NorthKorea는 IVisitable interface를 구현하지 않기 때문에 방문할 수 없다.
        A.tourAbroad(NorthKorea.getInstance());
        B.tourAbroad(NorthKorea.getInstance());


        System.out.printf("* 현재 %s의 위치 : %s\n", A.name, A.region);
        System.out.printf("* 현재 %s의 위치 : %s\n", B.name, B.region);

        System.out.println("\n* Situation Immigration :");

        A.Immigrate(US.getInstance());

    }
}
~~~

### 실행 결과


<img src="/img/posts/nestedclass/1.PNG" style="width: 75%">

<img src="/img/posts/nestedclass/2.PNG" style="width: 75%">


### IVisitable 인터페이스

~~~java

package jan11st;

public interface IVisitable {
    public Bureau visitImmigrationBureau();
}

~~~

### Korea 클래스

~~~java

package jan11st;

import java.lang.reflect.Array;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

// IVisitable 인터페이스를 구현해야만 Human 객체가 해당 Nation을 방문할 수 있음

public class Korea extends Nation implements Republic, IVisitable{

    // 싱글톤
    private static final Korea INSTANCE = new Korea();
    public static String NAME = "Republic Of Korea";
    public static String CAPITAL_CITY = "SEOUL";
    public static String REGION = "ASIA";
    public static String LANGUAGE = "KOREAN";

    //국가 리스트
    private List<Korean> civilianList = new ArrayList<>();

    private Korea() {
    }

    public static Korea getInstance() {
        return INSTANCE;
    }
    private void immigrationRegistration(Human human) {
        if (ImmigrationBureau.getInstance().immigrationScreening(human)) {
            System.out.println("한국 정부 : 이민 최종 승인");
        }
        else {
            System.out.println("한국 정부 : 이민 최종 거절");
        }
    }

    @Override
    public ImmigrationBureau visitImmigrationBureau() {
        return ImmigrationBureau.getInstance();
    }
    @Override
    public Bureau visitAdministartionBureau() {
        return AdministrationBureau.getInstance();
    }
    @Override
    public String getNAME() {
        return Korea.NAME;
    }

//정부 구현
    public static class AdministrationBureau extends Bureau {
        private static final AdministrationBureau INSTANCE = new AdministrationBureau();
        private AdministrationBureau() {}
        public static AdministrationBureau getInstance() {
            return INSTANCE;
        }
        public void birthRegistration(Korean korean) {
            System.out.printf("한국 정부 : %s 출생신고 접수 완료\n", korean.name);
            Korea.getInstance().civilianList.add(korean);
        }
        public void deathRegistration(Korean korean) {
            Korea.getInstance().civilianList.remove(korean);
        }
    }

    //이민청 구현, Bureau 추상클래스를 상속하고, IImmigrationBureau 인터페이스를 구현해 visa 발급,
    // 이민 신청등의 일(일반 Bureau가 아닌 이민청이 해야하는 일을 구현)

    public static class ImmigrationBureau extends Bureau implements IImmigrationBureau {

        //수교국 리스트를 생성, List는 Nation 추상클래스 받아 모든 Nation이 ListArray에 들어갈 수 있게함

        private List<Nation> dipolomaticNations = Arrays.asList(US.getInstance(), Korea.getInstance());

        private static final ImmigrationBureau INSTANCE = new ImmigrationBureau();

        private ImmigrationBureau() {
        }

        public static ImmigrationBureau getInstance() {
            return INSTANCE;
        }
        @Override
        public boolean visa(Human human) {
            System.out.println("visa");
            // 수교국 명단에 있으면 비자주기
//            human이 IlibertyToMove 구현체이면
            if(human instanceof ILibertyToMove) {
                System.out.printf("%s : 여권 있음\n", human.name);
                ILibertyToMove tourist = (ILibertyToMove) human;
                Nation passport = tourist.passportSubmitting();

            for(Nation nation : this.dipolomaticNations) {
                if (passport.equals(nation)) {
                    System.out.println("이민청 : 수교국 명단에 있음");
                    System.out.println("이민청 : 비자 발급 가능");
                    return true;
                }
            }
            }
            System.out.println("이민청 : 수교국 명단에 없음");
            System.out.println("이민청 : 비자 발급 불가");
            return false;
        }
        @Override
        public boolean immigrationScreening(Human human) {
            if(this.visa(human)) {
                if(human instanceof IImmigrant) {
                    System.out.println("이민청 :이민심사 승인");
                    return true;
                }
            }
            System.out.println("이민청 :이민심사 거절");
            return false;
        }
    }

//    Korean은 Human이고 이동할 수 있는 자유가 있음

    public static class Korean extends Human implements ILibertyToMove{
        Boolean died;
        Nation nationality;

        public Korean(String name, String sex, String region) {
            super(name, sex, region);
            System.out.printf("%s : %s 출생\n", this.region, this.name);
        }
        public void birthRegistration() {
            Bureau administrationBureau = Korea.getInstance().visitAdministartionBureau();
            System.out.printf("%s : %s 한국 행정센터 방문\n", this.name, this.region);
            AdministrationBureau koreaAdministartionBureau = (AdministrationBureau) administrationBureau;
            koreaAdministartionBureau.birthRegistration(this);
            this.nationality = Korea.getInstance();
        }
        @Override
        public void death() {
            this.died = true;
        }

        @Override
        public void tourAbroad(Nation nation) {
            if(nation instanceof IVisitable) {
                System.out.printf("%s : %s 방문 가능.\n", this.name, nation.getNAME());
                System.out.printf("%s : %s 도착\n", this.name, nation.getNAME());
                IVisitable travelDestination = (IVisitable) nation;

                Bureau immigrationBureau = travelDestination.visitImmigrationBureau();

                if (immigrationBureau instanceof IImmigrationBureau) {
                    IImmigrationBureau IBureau = (IImmigrationBureau) immigrationBureau;

                    if(IBureau.visa(this)) {
                        this.region = nation.getNAME();
                    }
                }
            }
            else{
                System.out.printf("%s : 목적지는 방문할 수 없다.\n", this.name);
            }
        }
        @Override
        public Nation passportSubmitting() {
            System.out.printf("%s : 여권 제출.\n", this.name);
            return this.nationality;
        }

        public void Immigrate(Nation nation) {
            if (nation instanceof IVisitable) {
                IVisitable targetNation = (IVisitable) nation;

                Bureau targetNationBureau = targetNation.visitImmigrationBureau();
                IImmigrationBureau targetNationImmigrationBureau = (IImmigrationBureau) targetNationBureau;
                targetNationImmigrationBureau.immigrationScreening(this);

            }
        }
    }
}

~~~

### IImmigrationBureau 인터페이스

~~~java

package jan11st;

public interface IImmigrationBureau {
    boolean visa(Human human);
    boolean immigrationScreening(Human human);
}
~~~

### NorthKorea 클래스

~~~java

package jan11st;

import java.lang.reflect.Array;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

//NorthKorea는 IVisitable의 구현체가 아니기 때문에 방문할 수 없음.
public class NorthKorea extends Nation implements Dictatorship{

    private static final NorthKorea INSTANCE = new NorthKorea();
    public static String NAME = "Democratic People\'s Republic of Korea";
    public static String CAPITAL_CITY = "PYEONGYANG";
    public static String REGION = "ASIA";
    public static String LANGUAGE = "KOREAN";
    private List<NorthKorean> civilianList = new ArrayList<>();

    private NorthKorea() {
    }

    // NorthKorean은 ILibertyToMove의 구현체가 아니기 때문에
    // tourAborad와 submittingPassport 메소드가 없음
    public static NorthKorea getInstance() {
        return INSTANCE;
    }
    private void immigrationRegistration(Human human) {
        if (ImmigrationBureau.getInstance().immigrationScreening(human)) {
            System.out.println("NorthKorean immigration Bureau : immigration Accepted");
        }
        else {
            System.out.println("NorthKorean immigratoin Bureau : immigration Disaccepted");
        }
    }

    @Override
    public String getNAME() {
        return NorthKorea.NAME;
    }

    @Override
    public Bureau visitAdministartionBureau() {
        return AdministrationBureau.getInstance();
    }

    public static class AdministrationBureau extends Bureau {
        private static final AdministrationBureau INSTANCE = new AdministrationBureau();
        private AdministrationBureau() {}
        public static AdministrationBureau getInstance() {
            return INSTANCE;
        }
        public void birthRegistration(NorthKorean northKorean) {
            System.out.printf("북한 정부 : %s 출생신고 접수 완료\n", northKorean.name);
            NorthKorea.getInstance().civilianList.add(northKorean);
        }
        public void deathRegistration(NorthKorean northKorean) {
            NorthKorea.getInstance().civilianList.remove(northKorean);
        }
    }

    public static class ImmigrationBureau extends Bureau implements IImmigrationBureau {
        private List<Nation> dipolomaticNations = Arrays.asList();

        private static final ImmigrationBureau INSTANCE = new ImmigrationBureau();

        private ImmigrationBureau() {
        }

        public static ImmigrationBureau getInstance() {
            return INSTANCE;
        }
        public boolean visa(Human human) {
            System.out.println("visa");
            // 수교국 명단에 있으면 비자주기

            if(human instanceof ILibertyToMove) {
                System.out.printf("%s : 여권 있음\n", human.name);
                ILibertyToMove tourist = (ILibertyToMove) human;
                Nation passport = tourist.passportSubmitting();

                for(Nation nation : this.dipolomaticNations) {
//                if(human instanceof ILibertyToMove) {
//                    System.out.printf("%s : 여권 있음\n", human.name);
//                    ILibertyToMove tourist = (ILibertyToMove) human;
//                    Nation passport = tourist.passportSubmitting();

                    if (passport.equals(nation)) {
                        System.out.println("이민청 : 수교국 명단에 있음");
                        System.out.println("이민청 : 비자 발급 가능");
                        return true;
                    }
                }
            }
            System.out.println("이민청 : 수교국 명단에 없음");
            System.out.println("이민청 : 비자 발급 불가");
            return false;
        }

        public boolean immigrationScreening(Human human) {
            if(this.visa(human)) {
                if(human instanceof IImmigrant) {
                    System.out.println("이민청 :이민심사 승인");
                    return true;
                }
            }
            System.out.println("이민청 :이민심사 거절");
            return false;
        }
    }

    public static class NorthKorean extends Human{
        Boolean died;
        Nation nationality;

        public NorthKorean(String name, String sex, String region) {
            super(name, sex, region);
            System.out.printf("%s : %s Born\n", this.region, this.name);
        }
        public void birthRegistration() {
            Bureau administrationBureau = NorthKorea.getInstance().visitAdministartionBureau();
            System.out.printf("%s : visit %s Administration Bureau\n", this.name, this.region);
            AdministrationBureau northKoreaAdministartionBureau = (AdministrationBureau) administrationBureau;
            northKoreaAdministartionBureau.birthRegistration(this);
            this.nationality = NorthKorea.getInstance();
        }
        @Override
        public void death() {
            this.died = true;
        }
    }
}
~~~

### Human 클래스

~~~java
package jan11st;

public abstract class Human {
    String name;
    String sex;
    String region;
    public Human(String name, String sex, String region){
        this.name = name;
        this.sex = sex;
        this.region = region;
    }
    void death() {
    };

    void eat() {};
}
~~~


### US 클래스

~~~java

package jan11st;

import java.lang.reflect.Array;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class US extends Nation implements Republic, IVisitable{

    private static final US INSTANCE = new US();
    public static String NAME = "United States Of America";
    public static String CAPITAL_CITY = "WASHINTON";
    public static String REGION = "NA";
    public static String LANGUAGE = "ENGLISH";
    private List<American> civilianList = new ArrayList<>();

    private US() {
    }

    public static US getInstance() {
        return INSTANCE;
    }
    private void immigrationRegistration(Human human) {
        if (ImmigrationBureau.getInstance().immigrationScreening(human)) {
            System.out.println("US Government : Immigration Finally Accepted");
        }
        else {
            System.out.println("US Government : Immigration Finally Disaccepted");
        }
    }
    public String getNAME() {
        return US.NAME;
    }

    @Override
    public ImmigrationBureau visitImmigrationBureau() {
        return ImmigrationBureau.getInstance();
    }

    @Override
    public Bureau visitAdministartionBureau() {
        return AdministrationBureau.getInstance();
    }

    public static class AdministrationBureau extends Bureau {
        private static final AdministrationBureau INSTANCE = new AdministrationBureau();
        private AdministrationBureau() {}
        public static AdministrationBureau getInstance() {
            return INSTANCE;
        }
        public void birthRegistration(American american) {
            System.out.printf("US Administration Bureau : %s \'s birth registration Accepted\n", american.name);

            US.getInstance().civilianList.add(american);
        }
        public void deathRegistration(American american) {
            US.getInstance().civilianList.remove(american);
        }
    }

    public static class ImmigrationBureau extends Bureau implements IImmigrationBureau{
        private List<Nation> dipolomaticNations = Arrays.asList(US.getInstance(), Korea.getInstance());

        private static final ImmigrationBureau INSTANCE = new ImmigrationBureau();

        private ImmigrationBureau() {
        }

        public static ImmigrationBureau getInstance() {
            return INSTANCE;
        }
        @Override
        public boolean visa(Human human) {
            System.out.println("visa");
            // 수교국 명단에 있으면 비자주기

            if (human instanceof ILibertyToMove) {
                System.out.printf("%s : Having Passport \n", human.name);
                ILibertyToMove tourist = (ILibertyToMove) human;
                Nation passport = tourist.passportSubmitting();

            for(Nation nation : this.dipolomaticNations) {
//                if(human instanceof ILibertyToMove) {
//                    System.out.printf("%s : 여권 있음\n", human.name);
//                    ILibertyToMove tourist = (ILibertyToMove) human;
//                    Nation passport = tourist.passportSubmitting();

                    if (passport.equals(nation)) {
                        System.out.println("Immigration Bureau : Visa Waiver Program Partner");
                        System.out.println("Immigration Bureau : Visa Granted");
                        return true;
                    }
                }
            }
            System.out.println("Immigration Bureau : Not Visa Waiver Program Partner");
            System.out.println("Immigration Bureau : Visa not Granted");
            return false;
        }
        @Override
        public boolean immigrationScreening(Human human) {
            if(this.visa(human)) {
                if(human instanceof IImmigrant) {
                    System.out.println("Immigration Bureau : Immigration Accepted");
                    return true;
                }
            }
            System.out.println("Immigration Bureau : Immigration Disaccepted");
            return false;
        }
    }

    public static class American extends Human implements ILibertyToMove{
        Boolean died;
        Nation nationality;

        public American(String name, String sex, String region) {
            super(name, sex, region);
            System.out.printf("%s : %s Born\n", this.region, this.name);
        }
        public void birthRegistration() {
            Bureau administrationBureau = US.getInstance().visitAdministartionBureau();
            System.out.printf("%s : visit %s Administration Bureau\n", this.name, this.region);
            AdministrationBureau USAdministartionBureau = (AdministrationBureau) administrationBureau;
            USAdministartionBureau.birthRegistration(this);
            this.nationality = Korea.getInstance();
        }
        @Override
        public void death() {
            this.died = true;
        }

        @Override
        public void tourAbroad(Nation nation) {
            if(nation instanceof IVisitable) {
                System.out.printf("%s : %s can be visited.\n", this.name, nation.getNAME());
                System.out.printf("%s : Arrived at %s\n", this.name, nation.getNAME());
                IVisitable travelDestination = (IVisitable) nation;

                Bureau immigrationBureau = travelDestination.visitImmigrationBureau();


                if (immigrationBureau instanceof IImmigrationBureau) {
                    IImmigrationBureau IBureau = (IImmigrationBureau) immigrationBureau;

                    if(IBureau.visa(this)) {
                        this.region = nation.getNAME();
                    }
                }
            }
            else{
                System.out.printf("%s :Destination is not available to visit.\n", this.name);
            }
        }
        @Override
        public Nation passportSubmitting() {
            System.out.printf("%s : Submit Passport\n", this.name);
            return this.nationality;
        }
    }
}
~~~