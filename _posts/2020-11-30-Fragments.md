---
layout: post
title:  "Fragments"
date:   2020-11-30
categories: [Android]
---

https://developer.android.com/guide/fragments

# Fragments

Framgents는 app의 UI중 재 사용 가능한 부분을 나타낸다.

fragment는 자신의 layout을 정의하고, 관리하며, 자체 lifecycle을 가지고, 자신의 입력 이벤트를 처리할 수 있다.

Framgnets는 독자적으로 존재할 수 없고, 액티비티나 다른 프래그먼트에 의해 호스트 되어야한다.

프래그먼트의 뷰 계층은 호스트 뷰 계층의 part of 혹은 attaches to 될 수 있다.

> 일부 Navigation, BottomNavigationView, ViewPager2 같은 일부 Android Jetpack 라이브러리는 fragemts와 동작하도록 설계되었습니다. 



## Modularity

프래그먼트는 UI를 chunk로 분할함으로써 modularity와 재사용성을 액티비티의 UI 에 도입한다. 

액티비티는 이상적인 장소, global emenets,  앱의 사용자 인터페이스, navigation drawaer같은 

반대로, 프래그먼트는 개별 스크린 혹은 스크린의 일부분의  UI를 정의, 관리하는데 더 적합하다.



다양한 화면 사이즈에 대응하는 앱을 생각해보면. 큰 화면에서는 앱의 static navigation drawer와 그리드 에이아웃 리스트를 보여줘야한다.

작은 화면에선,ㄴ 앱은 bottom navigation bar와 선형 레이아웃을 보여줘야한다.

이러한 모든 변경을 액티비티에서  관리하기는 어려울 수 있다.

컨텐츠에서 navigation 요소를 분리하는 것은 이러한 과정을 더욱 관리하기 쉽게 만들 수 있다. 

액티비티는 올바른 navigation UI(drawaer or bottom nav bar)를 표시하면된다. 프래그먼트가 적절한 레이아웃으로 리스트를 보여주는 동안



UI를 프래그먼트로 나누면 런타임에 액티비티의 모습을 더 쉽게 변경할 수 있다.

액티비티가 `STARTED` lifecycle state이거나 그 이상일때, 프래그먼츠는 추가될수 있고, 교체되거나 삭제될 수 있다. 

액티비티가 관리하는 백스택에 이러한 변경을 기록해두고, 변경사항을 되돌릴 수 있다.



동일한 액티비티, 다수의 액티비티 호긍ㄴ 심지어 다른 프래그먼트의 자식에서도 동일한 프래그먼트 클래스의 여러 인스턴스를 사용할 수 있다.

이를 염두에두고, 본인의 UI를 관리하는데 필요한 로직이 있는 프래그먼트만 제공해야한다(?)

하나의 프래그먼트를 다른 곳에서 의존하거나 조작하는것을 피해야 한다. 
