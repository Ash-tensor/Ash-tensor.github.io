---
layout: post
comments: true
title: "[Android Studio]Bottom Navigation Bar 구현 - OnItemSelectedListener 이용"
subtitle: setOnNavigationView Depricated 문제 해결
description: android studio
date: 2023-12-29
categories: 안드로이드
background: '/img/port.jpg'
---

## Bottom Navigation Bar

어떤 앱이든 Bottom Naviagation Bar가 없는 앱은 상상하기 힘들다. 토스, 에브리타임, 카카오톡은 아닌가? 아무튼, 과장 좀 보태서 거의 모든 앱의 하단 부분에는 이 Bottom Navigation Bar가 존재한다.

<img src="/img/posts/android_studio/3.jpeg" style="width: 80%">

이런거 말이다.

그런데 흔히 온라인에서 돌아다니는 BottomNavigationBar 구현방법 대부분은 사용중지가 될 예정인 setOnNavigationItemSelectedListener와 onNavigationItemSelectedListener를 사용하고 있어서, 사용이 비권장된다. 안드로이드 호환성 때문에 나중에 사용하다가 에러가 뜰 가능성도 높고, Deprecate될 기능을 사용하는 건 당연히 좋지 못하다. 사용중지가 될 기능을 유지보수하고 있는건 웃기기도 하고.

그래서 android 팀이 권장하는 OnItemSelectedListener를 이용해서 BottomNavigationBar룰 구현해보았다.

### menu.xml

일단 먼저 menu.xml을 추가해 주어야 한다. 그러기 위해서는 res 디렉토리에 menu라는 android resources directory를 추가해야 하는데

<img src="/img/posts/android_studio/menu1.png" style="width: 80%">

이렇게 추가한 뒤에, 똑같게 File > New > Menu Resources File을 추가하면 된다. 그 뒤에 item을 추가해 준다. item은 BottomNavigationBar의 각 메뉴로 구현된다.

        <item
            android:id="@+id/menu_1"
            android:title="MENU1"
                ##android:title은 화면에 표시될 이름이다
                ##나중에 bottomNavigationView와 연결할 때 android:id를 이용해서 연결한다.
            #android:icon="" 에서 icon에 아이콘 파일을 연결해 주면 아이콘이 나오는데 에셋을 만드는 중이라 현재는 추가하지 않았다
            />
         <item
            android:id="@+id/menu_2"
            android:title="MENU2"
            />
         <item
            android:id="@+id/menu_3"
            android:title="MENU3"
            />
        
        
        >


### activity_main.xml

    <?xml version="1.0" encoding="utf-8"?>
    <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <FrameLayout
                android:layout_width="match_parent"
                android:layout_height="0dp"  ## 레이아웃의 높이를 가중치를 이용하여 조정
                android:layout_weight="1"    ## 부모 레이아웃의 남은 공간을 모두 차지
                android:id="@+id/FragmentContainer" >
                <TextView
                    android:id="@+id/testTextView"   ## 프래그먼트가 바뀌었는지 확인용
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="firstView"
                    android:gravity="center"
                />
            </FrameLayout>

            <com.google.android.material.bottomnavigation.BottomNavigationView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_gravity="bottom"
                android:id="@+id/bottomNavigationView"
                app:menu="@menu/menu" />    ## 아까 만든 메뉴를 적으면 됨
        </LinearLayout>

    </androidx.constraintlayout.widget.ConstraintLayout>


기본적으로 리니어 레이아웃과 프레임 레이아웃을 사용해서 레이아웃을 설정했다. android:layout_height="0dp", android:layout_weight="1"을 이용해서 바텀 네비게이션 뷰를 차지한 공간 이외의 공간을 모두 LinearLayout이 차지하기로 하고, 이 리니어 레이아웃에 이후에 바텀 네비게이션 뷰의 메뉴를 누른 경우에 Fragment가 표시될 예정이다. android:id="@+id/FragmentContaner id를 이용해서 연결한다!


### fragment1 추가


<img src="/img/posts/android_studio/fragment2.png" style="width: 80%">

File > New > New Fragment를 추가해주고, Fragment를 작성한다. 이번에는 일단 BottomNavigationView를 작성하는게 목표기 때문에 따로 수정은 하지 않았다. 


### MainActivity.java

    public class MainActivity extends AppCompatActivity  {

        public FragmentManager fragmentManager = getSupportFragmentManager();
        BottomNavigationView bottomNavigationView;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);

            BottomNavigationView bottomNavigationView = findViewById(R.id.bottomNavigationView);
            bottomNavigationView.setOnItemSelectedListener(navListener);

        }

        private NavigationBarView.OnItemSelectedListener navListener = new NavigationBarView.OnItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                Fragment selectedFragment = null;

                switch (item.getItemId()) {    ## 누른 해당 메뉴의 id가 menu_1인 경우에
                    case R.id.menu_1:
                        selectedFragment = new fragment1();     ## fragment를 할당
                        break;
                }
                TextView testTextView = findViewById(R.id.testTextView);
                ##테스트용 TextView를 숨김
                testTextView.setVisibility(View.GONE);
                getSupportFragmentManager().beginTransaction().replace(R.id.FragmentContainer, selectedFragment).commit();

                return true;
            }
        } ;
    }

FrameLayout은 레이아웃을 겹쳐서 보여주기 때문에 testTextView와 Fragment1의 TextView가 겹쳐서 보여지게 된다. 그렇기 때문에 항상 Visibility를 바꿔줘야 한다.


### 실행

<img src="/img/posts/android_studio/2.PNG" style="width: 50%">

* 처음 실행 화면

<img src="/img/posts/android_studio/1.PNG" style="width: 50%">

* menu1을 선택한 화면


예상한대로 BottomNavigationView가 제대로 작동하는 모습을 볼 수 있다!