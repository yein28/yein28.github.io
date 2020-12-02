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

액티비티는 navigation drawer같은 사용자 인터페이스 주변에 global elements를 배치하기에 이상적인 장소이다.

반대로, 프래그먼트는 개별 스크린 혹은 스크린의 일부분의  UI를 정의, 관리하는데 더 적합하다.



다양한 화면 사이즈에 대응하는 앱을 생각해보면, 큰 화면에서는 앱의 static navigation drawer와 grid layout list를 보여줘야한다. 작은 화면에서,앱은 bottom navigation bar와 linear layout list를 보여줘야한다.

이러한 변경을 액티비티에서  관리하기는 어려울 수 있으며, 컨텐츠에서 navigation 요소를 분리하는 것은 이러한 과정을 더욱 관리하기 쉽게 만들 수 있다. 

액티비티는 프래그먼트가 적절한 레이아웃으로 list를 보여주는 동안 올바른 navigation UI(drawaer or bottom nav bar)를 표시하면된다. 



UI를 프래그먼트로 나누면 런타임에 액티비티의 모습을 더 쉽게 변경할 수 있다.

액티비티가 `STARTED` lifecycle state이거나 그 이상일때, 프래그먼츠는 추가될수 있고, 교체되거나 삭제될 수 있다. 

액티비티가 관리하는 백스택에 이러한 변경을 기록해두고, 변경사항을 되돌릴 수 있다.



동일한 액티비티, 다수의 액티비티, 심지어 다른 프래그먼트의 자식에서도 동일한 프래그먼트 클래스의 여러 인스턴스를 사용할 수 있다.

이를 염두에두고, 본인의 UI를 관리하는데 필요한 로직이 있는 프래그먼트만 제공해야한다(?)

하나의 프래그먼트를 다른 곳에서 의존하거나 조작하는것을 피해야 한다. 



# Create a fragment

`fragment` 는 액티비티 내 사용자 인터페이스의 모듈식 부분을 나타낸다. 본인의 lifecycle을 가지고, 입력 이벤트를 받으며, 프래그먼트를 포함하는 액티비티가 실행되는 동안 프래그먼트를 추가, 삭제할 수 있다.

이 문서는 프래그먼트를 생성하고, 액티비티에 포함하는 방법에 대해 설명한다.



## Setup your environment

디펜던시를 `build.gradle` 파일에 추가하세요.

(스킵)



## Create a fragment class

AndroidX `Fragment` 클래스를 상속받아 fragment를 만든다.그리고 앱의 로직을 집어 넣기위해 프래그먼트의 메소드들을 오버라이드 한다. Activity 클래스를 만드는 방법과 비슷하다. 

자체 레이아웃을 정의하는 minimal한 프래그먼트를 만드려면 다음의 예제처럼 프래그먼트의 레이아웃 리소스를 base constructor에 제공한다. 

```kotlin
class ExampleFragment : Fragment(R.layout.exmple_fragment)
```

Fragment 라이브러리는 또한 좀더 특화된 프래그먼트 베이스 클래스들을 제공한다.

`DialongFramgment`

- floating dialog를 표시한다. 이 클래스를 사용해 다이얼로그를 만들면 프래그먼트가 자동으로 `Dialog` 의 생성과 정리를 담당하기때문에 Activity 클래스에 있는 dialog 헬퍼 메소드들을 이용하는 것보다 좋은 대안이다. 

`PrefrenceFragmnetCompat`

- `Prefrence` 객체의 계층을 리스트로 표시한다.`PreferenceFragmentCompat` 을 사용해서 앱의 설정 화면을 만들 수 있다.



## Add a fragment to an activity

일반적으로, 액티비티의 레이아웃의 UI 일부를 제공하려면 프래그먼트는 반드시 AndroidX `FragmentActivity` 에 포함되어야 한다. 

`FragmentActivity` 는 `AppCompatActivity` 의 베이스 클래스이다. 그러므로 만약 앱의 하위 호환성을 위해 이미 `AppCompatActivity` 를 서브클래싱한다면 액티비티의 베이스 클래스를 변경할 필요가 없다.

프래그먼트는 다음과 같은 방법으로 액티비티의 뷰 계층에 추가할 수 있다.

- 액티비티의 레이아웃 파일에 프래그먼트를 정의하거나 

- 액티비티의 레이아웃 파일에 프래그먼트 컨테이너를 정의한 후 프래그먼트를 액티비티 내에서 programatically하게 추가

두 케이스 모두, 액티비티의 뷰 계층 내에서 어디에 위치해야할지 결정하기위해 `FragmentContainerView` 를 추가해야한다.

프래그먼트의 컨테이너로서 항상 `FragmentContainerView` 를 사용할것을 강력하게 추천한다. 

`FrameLayout` 같은 다른 뷰에서는 제공하지 않는 특정 프래그먼트에 관련된 수정사항을 포함하기 때문에



### Add a fragment via XML

액티비티의 레이아웃 XML에 선언적으로 프래그먼트를 추가하기 위해서는 `FragmentContainerView` 요소를 사용해라.

아래는 하나의 `FragmentContainerView` 를 포함하는 액티비티 레이아웃의 예제이다.

```xml
<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:name="com.example.ExampleFragment" />
```

`android:name` 어트리뷰트는 인스턴스화 할 `Fragment` 이름을 명시한다. 액티비티의 레이아웃이 inflate 되면, 명시된 프래그먼트가 인스턴스화되고, 새롭게 인스턴스화 되는 프래그먼트에서 `onInflate()` 가 호출된다, 그리고  `FragmentManager` 에 프래그먼트를 추가하기위해 `FragmentTransaction` 이 생성된다.

> android:name 어트리뷰트 대신 class 어트리뷰트를 사용할 수 있다. 



### Add a fragment programmatically 

액티비티의 레이아웃에 programmatically하게 프래그먼트를 추가하려면,  레이아웃은 프래그먼트의 컨테이너의 역할을 할 `FragmentContainerView` 를 포함해야한다.

```xml
<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

XML 방식과는 다르게, `android:name` 어트리뷰트는 사용하지 않는다, 즉 특정 프래그먼트가 자동으로 인스턴스화 되지 않는다. 대신에, 프래그먼트를 인스턴스화 하고 이를 액티비티의 레이아웃에 추가하기위해  `FragmentTransaction` 이 사용된다. 



액티비티가 실행 되는 동안, 추가, 삭제, 프래그먼트 replace같은 프래그먼트 트랜잭션을 수행할 수 있다. `FragmentActivity`  에서 FragmentTransaction을 만드는데 사용할 `FragmentManager` 의 인스턴스를 얻을 수 있다. 

그리고, 액티비티의 `onCreate()` 메서드에서 `FragmentTransaction.add()` 를 사용해서 프래그먼트를 인스턴스화 할 수 있다. 

```kotlin
class ExampleActivity : AppCompatActivity(R.layout.example_activity) {
  override fun onCreate(savedInstanceState: Bundle?) {
  	super.onCreate(savedInstanceState)
    if (savedInstanceState == null) {
      supportFragmentManager.commit {
        setReorderingAllowed(true)
        add<ExampleFragment>(R.id.fragment_container_view)
      }
    }
  }
}
```

> FragmentTransaction을 수행하는 경우 항상 setReorderingAllowed(true)를 사용해야한다. 



이전 예제에서, 프래그먼트 트랜잭션이 `savedInstanceState` 가 `null`인경우에만 생성된것을 확인해라.

이는 액티비티가 처음 생성되었을때, fragment가 오직 한번만 포함되는 것을 보장하기 위함이다. configuration 변경이 일어나고 액티비티가 재생성되면 `savedInstanceState` 는 더이상 `null` 이 아니며, 프래그먼트는 savedInstanceState에서 자동으로 복원되기때문에 fragment를 두번 추가할 필요가 없다.



만약 프래그먼트가 몇가지 초기 데이터를 필요로하는경우, `FragmentTransaction.add()` 호출에 Bundle을 제공해서 arguments들을 전달할 수 있다.

```kotlin
val bundle = bundleOf("sone_int" to 0)
supportFragmentManager.commit {
  setReorderingAllowed(true)
  add<ExampleFragment>(R.id.fragment_container_view)
}
```

`Bundle` arguments는 프래그먼트에서 `requireArguments()` 를 호출해서 얻을 수 있고, 적절한 `Bundle` getter method들을 사용해서 각 argument를 얻을 수 있다.

```kotlin
val someInt = requireArguments().getInt("some_int")
```



알아 볼 것

- 프래그먼트의 컨테이너로 FragmentContainerView를 추천하는 이유, 어떤 수정사항을 포함하는 건지
- FragmentTransaction을 수행하는 경우 항상 setReorderingAllowed(true)를 사용해야하는 이유 
  - Fragment transactions 참고

- XML, 코드에서 프래그먼트를 추가하는 경우 라이프 사이클 차이

