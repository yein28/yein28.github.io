---
layout : post
title : "Jetpack Compose"
date : 2020-12-11
categoris : [Android]
---

# Jetpack Compose

언젠가는 사용할거같으므로,, 미리 익숙해지면,, 좋을듯,,,

https://developer.android.com/courses/pathways/compose



## 1. Tutorial: Compose Basics

Jetpack Compose는 네이티브 UI를 만들기 위한 안드로이드의 모던 툴킷

Android에서 UI개발을 단순하고 빠르게 해줌(with 적은 코드, 강력한 툴들, 직관적인 Kotlin API와 함께)

이 튜토리얼에서는 선언적 함수를 이용해 단순한 UI 컴포넌트를 만들어 볼것, XML 레이아웃을 편집하거나 직접적으로 UI 위젯을 만들지 않아도됨

대신에, Jetpack Compose 함수를 호출해서 내가 어떤 엘리먼트를 원하는지 알려주면 나머지는 Compose compiler가 수행함 



### Lesson1: Composable funcions

Jetpack Compose는 composable 함수들을 중심으로 구성됨. 이러한 함수를 사용하여 모양과 데이터 의존성을 programatically하게 앱의 UI를 정의하게 해준다, UI 구성 프로세스에 초점을 맞추지않고

composable함수를 만드려면, `@Composable` 어노테이션을 붙이면된다.



#### Add a text element

시작하기전에, Jetpack Compose 셋업 가이드를 보고 준비, Android studio canary build를 받아야함

Empty Compose Activity 템플릿을 이용해서 앱을 하나 만들어야 함. 이 디폴트 템플릿은 이미 몇가지 Compose element들을 가지고 있으나 스텝 바이 스텝으로 만들어보자.

첫째로, "Greeting"과 "Default Preview" 함수를 지워라, 그리고 `MainActivy` 내 `setContent` 를 지워 액티비티를 빈 상태로 만들고 빈 앱을 컴파일하고 실행해봐라 

이제 빈 액티비티에 text element를 추가해보자. ~content~ block을 정의하고  `Text()` 함수를 호출하면 됨 

setContent 블럭은 액티비티의 레이아웃을 정의함. XML 파일로 레이아웃 컨텐츠들을 정의하는 대신에, composable 함수들을 호출하면 됨

Jetpack Compose는 이러한 composable 함수들을 앱의 UI 엘리먼트로 바꾸기 위해 커스텀 코틀린 컴파일러 플러그인을 사용함

예를들어서, `Text() ` 함수는 Compose Ui 라이브러리에 정의되어있음, text element를 앱에 선언하기 위해서 해당 함수를 호출하면 됨 



#### Define a composable function

Composable 함수들은 오직 다른 composable 함수의 스코프 내에서만 호출할 수 있음.

함수를 composable하게 만들기 위해서는 `@Composable` 어노테이션을 추가하면됨. 

```kotlin
@Composable
fun Greeting(name: String) {
    Text (text = "Hello $name!")
}
```



#### Preview your function in Android Studio

현 canary 빌드에서는 IDE 내에서 composable 함수들을 미리볼 수 있게해줌 앱을 기기에 다운받거나 에뮬레이터를 돌릴 필요 없이 

중요한 제한 사항은, 해당 composable 함수가 어떠한 파라미터도 가지면 안됨(ㅋㅋ)

이러한 이유때문에, 위에서 정의한 `Greeting()` 함수는 미리볼 수 없음. 대신에 `Greeting`을 호출하는  `PreviewGreeting()` 함수를 만들어서 확인할 수 있음

`@Preview` 어노테이션을 `@Composable`전에 추가 

```kotlin
@Preview
@Composable
fun PreviewGreeting() {
    Greeting("Android")
}
```

프로젝트를 rebuild함, 앱은 변한게 없지만 Android Studio 에서 preview window를 추가함 

 -> rebuild 했는데도 window가 안보여서 껐다 키니까 보임 ;

이 윈도우는 @Preview 어노테이션으로 표시된 composable 함수가 만든 UI element에 대한 preview를 보여줌

미리보기를 업데이트 하고 싶다면 preview 윈도우 상단의 refresh 버튼을 클릭하면 됨

