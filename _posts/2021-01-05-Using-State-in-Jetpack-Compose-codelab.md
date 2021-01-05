---
layout: post
title: "Using State in Jetpack Compose codelab"
date: "2021-01-05"
categories: [Android]
---



# Using State in Jetpack Compose codelab

들어가기전에, *state* 란 무엇인지 정확하게 정의하고 가자. 

state는 시간이 지남에 따라 변할 수 있는 값.

이는 광범위한 정의로서 Room DB 부터 클래스 변수까지 모든 항목이 포함 됨

> 예를 들어  Room DB에 저장된 값 또는 객체의 속서, 심지어는 가속도계에서 읽어오는 현재 값도 state

모든 안드로이드 앱은 state를 사용자에게 보여줌. 안드로이드 앱에서의 state의 몇가지 예제

1. 네트워크 연결되지 않은 경우 보여주는 스낵바
2. 블로그 글과 연관된 댓글들
3. 사용자가 클릭했을때 버튼의 리플 애니메이션
4. 사용자가 이미지 위에 그리는 스티커들



이 코드랩에서는 Jetpack Compose를 사용할 때 state를 어떻게 사용하고 생각 할 것 인지에 다룸



## Understanding Unidirectional Data Flow

> Android view system에서 사용하는 단방향 데이터 플로우 컨셉에 대한 안내 

### The UI update loop

state가 업데이트 되는 원인은 무엇인가? introduction에서 state를 시간이 지남에 따라 변하는 값으로 이야기함.

이는 android application에서 상태에 대한 이야기 중 일부에 불과

Android app에서 state는 이벤트에 대한 응답으로 업데이트 됨. **Events** 는 앱 외부에서 생성된 입력이다.(ex. OnClickListener를 호출하는 버튼, 새 값을 보내는 가속도계 등)

Compose 에서 상태를 관리하는 것은 상태와 이벤트가 서로 상호작용하는 방식을 이해하는 것



### Unstructured state

사용자에게서 이름을 받아 "Hello, {name}" 형태로 보여주는 액티비티를 만든다고 예시

이를 작성할 수 있는 한가지 방법 - 이벤트 콜백이 TextView에서 직접 상태를 설정

```kotlin
class HelloCodelabActivity : AppCompatActivity() {
  private lateinit var binding: ActivityHelloCodelabBinding
  var name = ""
  
  override fun onCreate(savedInstanceState: Bundle?) {
    /* ... */
    binding.textInput.doAfterTextChanger { text ->
      name = text.toString()
      updateHello()                                    
    }
  }
  
  private fun updateHello() {
    binding.helloText.text = "Hello, $name"
  }
}
```

> 해당 코드는 구조화 되지않은 상태가 Activity에 저장되고있는 것을 보여줌 

작은 예시에서는 괜찮지만, UI가 커질 수록 관리하기가 어려워짐

이러한 액티비티에 더 많은 이벤트와 상태가 추가될 수록 다름과 같은 문제가 발생할 수 있음:

1. Testing - UI의 state와 `View`가 얽혀있기때문에 코드를 테스트 하기 어려움
2. Partial state updates - 화면이 아주 많은 이벤트를 가지고 있는 경우, 이벤트에 대한 응답으로 상태를 업데이트 하는 것을 까먹기 쉬움, 결과적으로 사용자에게 일관성이 없거나 잘못된 UI가 표시될 수 있음
3. Partial UI updates - 상태가 변경될 때마다 수동으로 UI를 업데이트하기 때문에 까먹기 쉬움
4. Code complexity



### Using Unidirectional Data Flow

이러한 구조화 되지 않은 상태의 문제를 해결하기위해  `ViewModel` 과 `LiveData` 를 포함하고 있는 Android Architecture Component를 도입함