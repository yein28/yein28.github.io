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



### Lesson 2: Lyaouts

UI element 들은 계층적이며, element들은 다른 element들에 포함된다. 

Compose에서, 계층적인 UI를 composable 함수를 다른 composable 함수 내에서 호출함으로써 구성할 수 있다. 



#### Start with some Text

액티비티로 돌아가서, `Greeting()` 함수를 새로운 `NewsStroy()` 함수로 교체한다. 남은 튜토리얼 동안, `NewsSotry()` 함수를 수정할 것이며 더이상 `Activity` 내 코드에 손댈필요 없다. 

이는 앱에서 호출하지 않는 분리된 preview 함수를 만드는 좋은 예제 이다.

전용 preview 함수를 가지는것은 퍼포먼스를 향상시키고, 나중에 여러 preview를 쉽게 설정할 수 있다. 

아무것도 하지 않지만 `NewsStory`를 호출하는 default preview 함수를 만든다. 튜토리얼 동안 `NewsStory()`에 변경이 생기면 preview 에서 변경을 반영할것이다.

아래 코드는 content view 내에서 세가지 text element를 만든다. 그러나, 우리는 그들을 어떻게 배치할지 아무런 정보도 제공하지 않았기 때문에, 텍스트 엘리먼트들은 text를 읽을 수 없도록 각각 상단에 그려질것이다.



#### Using a Column

`Column` 함수는 element들을 수직으로 쌓게 해준다. 디폴트 세팅은 모든 children들을 간격없이 쌓는다.

컬럼 자체는 content view의 왼쪽 상단 모서리에 배치된다.



#### Add style setting to the column

`Column` 호출에 파라미터를 넘기면, 컬럼의 사이즈와 위치를 구성할 수 있고, 컬럼의 children을 어떻게 배치할지 정할 수 있다.

- `modifier` : 레이아웃을 구성하게 해줌.

  ```kotlin
  Column(
    modifier = Modifier.padding(16.dp) // ex 16dp 패딩을 추가
  ) { ... }
  ```

  

#### Add a picture

텍스트 상단에 그래픽을 추가하고 싶다. `Resource Manager`를 사용해서 앱의 drawable에 추가할 수 있다.

이제 `NewsStory()` 함수를 수정해서, `Column`  내에 `Image()` 를 사용해 그래픽을 추가한다.이 composable은 `foundation` 패키지에서 사용가능하다. 지금은 이미지가 제대로 균형잡혀 있지않지만, 다음 스텝에서 수정할것이다.

`val image = imageResource(id = R.drawable.header)`



그래픽이 레이아웃에 추가되었지만, 적절한 사이즈가 아니다. 그래픽의 스타일을 변경하기위해서 `Image()` 호출에 size `Modifier` 를 넘긴다.

- `preferredHeight(180.dp)` : 이미지의 height 를 정의
- `fillMaxWidth` : 이미지가 속한 레이아웃을 꽉 채워야 함을 명시

또 `contentScale` 파라미터를 `Image()` 함께 넘겨주어야 한다. 

-  `contentScale = ContentScale.Crop` : 그래픽이 column의 width를 꽉 채워야함을 명시, 그리고 필요하다면 적절한 높이로 crop 한다.

```kotlin
val imageModifier = Modifier.preferredHeight(180.dp).fillMaxWidth()
Image(image, modifier = imageModifier, contentScale = ContentScale.Crop)
```

상단과 그래픽 사이를 `Spacer` 를 추가해서 분리한다.





