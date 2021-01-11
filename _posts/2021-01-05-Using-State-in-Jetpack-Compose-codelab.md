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

`ViewModel`은 UI에서 상태를 추출하고 UI가 해당 상태를 업데이트 하기 위해 호출 할 수 있는 이벤트를 정의함.

동일한 Activity를 `ViewModel`을 사용해 작성하면 다음과 같음

```kotlin
class HelloCodelabViewModel : ViewMode() {
  // UI가 observe하고 있는 상태를 LiveData가 들고 있음
  // (상태 flow down from ViewModel)
  private val _name = MutableLiveData("")
  val name: LiveData<String> = _name
  
  // onNameChanged는 UI가 호출(invoke)할 수 있도록 정의한 이벤트
  // (이벤트는 flow up from UI)
  fun onNameChanged(newName: String) {
    _name.value = newName
  }
}

class HelloCodeLabActivityWithViewModel : AppCompatActivity() {
  val helloViewModel by viewModels<HelloCodelabViewModel>()
  
  override fun onCreate(savedInstanceState: Bundle?) {
    /* ... */
    binding.textInput.doAfterTextChanged {
      // input이 변경될 때마다 UI가 호출
      helloViewModel.onNameChanged(it.toString())
    }
    
    helloViewModel.name.observe(this) { name ->
      binding.helloText.text = "Hello, $name"
    }
  }
} 
```

상태를 Activity에서 ViewModel로 옮기고, ViewModel에서 상태는 LiveData로 표현됨

UI에서는 `observe` 메서드를 사용해서 상태가 변경될 때마다 UI 를 업데이트 함

> observable이란 상태 객체이다. 상태의 변경사항을 수신할 수 있는 방법을 제공하는. 
>
> LiveData, StateFlow, Flow, Observable 모두 observable 임

이러한 패턴을 **unidirectional data flow(단방향 데이터 플로우)** 라고 함.

<img src="/assets/9.png" width="500" >

- state는 아래로, 이벤트는 위로 흐르는 구조.

다음과 같은 장점을 얻을 수 있음

- Testability - 상태와 상태를 표시하는 UI의 결합을 풂으로써(decoupling), ViewModel과 Activity 각각 쉽게 테스트할 수 있다
- State encapsulation - 상태는 한 곳(ViewModel) 에서만 업데이트 되므로 UI 가 커져도 UI의 부분적 업데이트 같은 버그를 낼 가능성이 적음
- UI consistency - 모든 상태 업데이트는 UI에 즉시 반영됨



## Compose and ViewModels

Compose에서 ViewModels를 사용해 어떻게 단방향 데이터 플로우를 다루는지 

Todo 화면을 구현 할 것 

```kotlin
@Composable
fun TodoScreen(
   items: List<TodoItem>,
   onAddItem: (TodoItem) -> Unit,
   onRemoveItem: (TodoItem) -> Unit
) {
   /* ... */
}
```

이 composable은 편집할 수 있는 TODO 리스트를 보여주지만, 어떠한 state도 가지고 있지 않음

state는 변할 수 있는 어떠한 값인데, TodoScreen의 아규먼트는 변경될 수 없음

- `items` - 화면에 보여줄 immutable한 아이템 리스트
- `onAddItem` - 사용자가 아이템 추가를 요청했을때를 위한 이벤트
- `onRemoveItem` - 사용자가 아이템 삭제를 요청했을 때를 위한 이벤트

In fact, 이 composable은 **stateless** 임. 오직 넘어오는 리스트만 보여주며, 직접적으로 리스트를 편집할 방법은 없음

대신에, 변경을 요청할 수 있는 두가지 이벤트를 넘긴다(`onRemoveItem`, `onAddItem`)

> **stateless composable** 은 어떠한 state도 직접적으로 변경할 수 없는 composable 임

몇 가지 의문이 들 것: 만약 stateless라면, 어떻게 편집가능한 리스트를 보여줄 것인가? 

-> **state hoisting** 이란 기법을 사용한다.

State hoisting은 component를 stateless하게 만들기위한 state를 위로 올리는 패턴임

Stateless component는 테스트하기 쉽고, 버그가 적을 가능성이 높고, 재사용성이 높음



이러한 파라미터의 조합은 caller가 composable에서 state를 hoist(끌어올림)

- Event - 사용자가 아이템의 추가나 삭제를 요청, TodoScreen이 onAddItem또는 onRemoveItem 호출
- Update state - TodoScreen의 caller가 state를 업데이트 하여 이러한 이벤트에 응답할 수 있음
- Display state - state가 업데이트되면 새로운 items와 함께 TodoScreen이 재호출, 그리고 화면에 보여줌

caller는 어디에서, 어떻게 state를 들고 있을지에 대한 책임이 있음. Caller는 `items`를 메모리에 저장하거나 Room DB에서 읽을 수 있음

`TodoScreen`은 state가 어떻게 관리되는지와 완벽하게 decouple되어있음

>**State hoisting**은 component를 stateless하게 만들기 위해 state를 위로 올리는 패턴이다.
>
>composable에 도입될 때, 종종 두개의 매개변수를 도입하게 된다.
>
>- value: T - the current value to display
>- onValueChange: (T) -> Unit - an event that requests the value to change, where T is the proposed new value

```kotlin
@Composable
private fun TodoActivityScreen(todoViewModel: TodoViewModel) {
  val items = listOf<TodoItem>()
  TodoScreen(
    items = items,
    onAddItem = {},
    onRemoveItem = {}
  )
}
```

이러한 composable은 ViewModel에 저장된 state와 `TodoScreen` 사이의 브릿지의 역할을 함

`TodoScreen` 을 변경하여 `ViewModel`을 직접 가져올 수 있지만 재사용성이 약간 떨어짐

`List<TodoItem>` 같은 심플한 파라미터를 선호함으로써, `TodoScreen`은 state가 hoist가 되는 특정위치에 결합되지 않음