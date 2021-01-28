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



event를 위로 올리기

```kotlin
@Composable
private fun TodoActivityScreen(todoViewModel: TodoViewModel) {
   val items = listOf<TodoItem>()
   TodoScreen(
       items = items,
       onAddItem = { todoViewModel.addItem(it) },
       onRemoveItem = { todoViewModel.removeItem(it) }
   )
}
```

state 를 아래로 내리기 

```kotlin
@Composable
private fun TodoActivityScreen(todoViewModel: TodoViewModel) {
 	 // 변경된 부분
   val items: List<TodoItem> by todoViewModel.todoItems.observeAsState(listOf())
   TodoScreen(
       items = items,
       onAddItem = { todoViewModel.addItem(it) },
       onRemoveItem = { todoViewModel.removeItem(it) }
   )
}
```

- `val items: List<TodoItem>` , `List<TodoItem>` 타입의 `items` 변수를 선언
- `todoViewModel.todoItems`, `ViewModel` 이 가지고 있는  `LiveData<List<TodoItem>>`

- `.observeAsState` , `LiveData<T>` 를 옵저브하여 `State<T>` 객체로 변환하여, Compose 가 값의 변화에 반응할 수 있도록 함
- `listOf()` 는 `LiveData` 초기화 되기전에 가능한 null 결과를 피하기위한 초기값, 만약 넘기지 않는 경우 `items` 는 nullable한 값을 가지게됨(`List<TodoItem>?`)

- `by` 는 코틀린에서의 프로퍼티 델리게이트 문법, `State<List<TodoItem>>` 을 자동으로 unwrap 해서 `List<TodoItem>` 으로

> observeAsState는 LiveData 를 옵저브하고, LiveData가 변경될 때마다 업데이트 되는 State 객체를 반환함
>
> composition에서 composable이 삭제되는 경우 자동으로 observe를 중지함



## Memory in Compose

이 섹션과 다음 섹션에 거쳐 stateful 한 composable을 만드는 방법을 배울 것 

이 섹션에서는 composable 함수에 memory를 추가하는 방법을 살펴봄

TodoRow가 추가될 때 마다 icon의 alpha값이 바뀌도록 요구사항이 들어옴

TodoRow Composable 내 Icon의 tint가 랜덤으로 값이 바뀌도록 수정

문제점

- TodoRow를 추가할때마다 기존 리스트의 아이콘들의 alpha가 바뀜 

  -> recomposotion 프로세스에서 `randomTint` 를 호출하기 때문에 

**Recomposition** 은 새로운 입력으로 composable을 재 호출하여 compose tree를 업데이트 하는 프로세스 

`TodoScreen` 이 새로운 리스트와 함께 호출될때, `LazyColumn` 이 화면의 모든 자식들을 recompose하게 됨

이는 다시 `TodoRow` 를 호출하고, 새롭게 random tint를 생성 하게 됨



Compose는 tree를 만드는데, Android View System의 UI tree와는 조금 다름

UI 위젯의 트리 대신에, Compose는 composable 트리를 생성함

`TodoScreen` 을 시각화 해보면 다음과 같음 

<img src="/assets/10.png" width="500" >

TodoRow의  recompose 때마다 icon이 업데이트 되는 이유는 TodoRow에 숨겨진 side-effect가 있기 때문

side-effect는 composable 함수의 실행 외부에서 보이는 모든 변경 사항을 말함

> Composable의 recompose는 side-effect가 없어야 함
>
> 예를 들어서, ViewModel에서 state 업데이트, Random.nextInt() 호출, 또는 db에 쓰기 모두 side-effect임 



### Introducing memory to composable functions

`TodoRow` 의 recompose 때마다 tint가 바뀌는 것은 원하지 않으므로. 마지막 composition에서 사용한 tint를 기억할 공간이 필요함.

Compose 는 compostion tree에 값을 저장할 수 있게하므로, `TodoRow` 를 업데이트하여 `icomAlpha` 값을 composition tree에 저장할 수 있음 

> **remember** 는 composable function memory를 제공
>
> **remember**가 계산한 값은 composition tree에 저장됨, 그리고 오직 **remeber**의 key가 변경될 때에만 재 계산 됨
>
> object에 private val 프로퍼티가 하는 것과 같은 방식으로, **remember** 가 단일 객체에 대한 저장소를 함수에 제공하는 것으로 생각할 수 있음

```kotlin
val iconAlpha = remember(todo.id) { randomTint() }
```

`TodoRow` 의 새로운 compose tree를 살펴보면, `iconAlpha` 가 compose tree에 추가된 것을 볼 수 있음

<img src="/assets/11.png" width="500" >

앱을 다시 실행하면, 리스트가 변경되어도 tint가 변경되지 않음을 확인할 수 있음

recomposition이 일어나면, `remember`에 저장된 값이 반환 됨

remember의 호출을 좀 더 들여다 보면 `key` 아규먼트에 `todo.id` 를 넘기는 것을 볼 수 있음

remember의 호출은 두가지 부분으로 나뉨

1. **key arguments** - remember가 사용하는 "key"
2. **calculation** - 기억 할 새로운 값을 계산하는 람다

첫번째 comose에서, remember는 항상 `randomTint` 를 호출하고 다음 recompostion 을 위해 결과를 기억함

`TodoRow` 에 새로운 `todo.id` 가 전달되지 않는 한, 기억한 tint 값을 반환 함

> idempotent(멱등) composable은 같은 input에 대해 항상 같은 결과를 냄, 그리고 recomposition에서 어떠한 side-effect가 없음
>
> Composalbe은 recomposition을 지원하기위해 idempotent 해야 함 



### Making remembered value controllable

`TodoRow` 의 호출자가 tint를 명시할 수 없는 문제가 있음

caller가 해당 값을 컨트롤할 수 있게하기위해서는 단순하게 새로운 `iconAlpha` 파라미터의 기본인수로 remember 호출을 이동하면 됨

```kotlin
// AS-IS
@Composable
fun TodoRow(
  todo: TodoItem, 
  onItemClicked: (TodoItem) -> Unit, 
  modifier: Modifier = Modifier
) 

// TO-BE
@Composable
fun TodoRow(
   todo: TodoItem,
   onItemClicked: (TodoItem) -> Unit,
   modifier: Modifier = Modifier,
   iconAlpha: Float = remember(todo.id) { randomTint() }
)
```

`remember` 의 사용법에 미묘한 버그가 있음

스크롤이 될 만큼 아이템을 추가하고나서 스크롤링 해보면, 화면에서 스크롤 할때 마다 아이콘의 alpha 값이 변함

> Remember 는 Composition에 값을 저장하고, remember를 호출한 composable 이 제거되면 값을 제거함
>
> 이는 `Lazycolumn` 같은 자식을 추가하고 제거하는 composable 내부에 중요한 것들을 저장하기 위해 remember에 의존해서는 안된다는 것을 의미 



## State in Compose

Composable에 state를 추가하기위해 memory를 사용하는 방법을 알아 볼 것

<img src="/assets/12.png" width="500" >



UI에서 텍스트를 편집하는 것은 stateful 함. 

Android view system에서, 이 state는 `EditText` 내에 있으며 `onTExtChanged` 리스너를 통해서 노출됨, 

하지만 compose는 단방향 데이터 플로우를 위해 설계되었기때문에 적합하지 않다. 

> TextField is the compose equivalent to Material's EditText

compose에서 `TextField` 는 stateless한 compsable 이다. `TodoScreen` 이 변화하는 todo 리스트를 보여주기만 하는 것 처럼, `TextField` 는 사용자가 알려주는 내용을 표시하고, 사용자가 입력할때 이벤트를 발행한다.

> **Built-in composable 은 unidirection data flow를 위해 설계됨**
>
> 대부분의 빌트인 컴포저들블은 각 API에 대해 적어도 하나 이상의 stateless 버전을 제공함
>
> View system과 비교해서, 빌트인 composable은 편집가능한 텍스트와 같은 stateful UI 의 내부 state가 없는 옵션을 제공함
>
> 인는 앱과 component 사이에 중복되는 state를 방지함
>
> 예를들어, Compose에서는 CheckBox의 상태를 서버 베이스 API로 hoist 할 수 있따. 중복되는 상태 없이 (??)



### Create a stateful TextField composable

stateful 한 component를 만들어 볼것, editable `TextField` 를 보여주기위해 

다음과 같은 함수를 `TodoScreen.kt` 에 추가

```kotlin
@Composable
// 원래는 state를 hoist 해야하지만 하지 않음, 나중에 지울 함수임
fun TodoInputTextField(modifier: Modifier) {
  val (text, setText) = remember { mutableStateOf("") }
  TodoInputText(text, setText, modifier)
}
```

이 함수에서는 `remember` 를 사용해 memory를 추가함. 그리고 메모리에 `mutableStateOf` 를 저장해서 관찰가능한 state 홀더를 제공하는 Compose의 빌트인 타입 인 `MutableState<String> ` 을 만든다. 

`TodoInputText` 에 바로 value와 setter를 넘길것이므로 `MutableState` 를 getter 와 setter로 분해 함

>[`mutableStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#mutableStateOf(androidx.compose.runtime.mutableStateOf.T, androidx.compose.runtime.SnapshotMutationPolicy)) creates a [`MutableState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/MutableState) which is an observable state holder built into compose.
>
>```kotlin 
>interface MutableState<T> : State <T> {
>  override var value: T
>}
>```
>
>value에 변경이 일어나면 이 state를 읽는 모든  composable 함수가 자동으로 recompose 됨
>
>composable 내에서 MutableState 객체를 다음과 같은 세가지 방법으로 선언할 수 있음.
>
>1. val state = remember { mutableStateOf(default) }
>2. var value by rememer { mutableStateOf(default) }
>3. val (value, setValue) = remember { mutableStateOf(default) }
>
>composition에서 `State<T>` 를 만들때 (또는 다른 stateful한 객체들), `remember` 는 중요함
>
>remember 가 없으면 매 composition마다 재-초기화 될 것 
>
>`MutableState<T>` 는 `MutableLiveData<T>` 와 유사하지만, compose runtime 과 통합되었음(?)
>
>observable 하기 때문에,  it will tell compose whenever it's updated so compose can recompose any composables that read it.

`TodoInputTextField` 에서 우리는 internal state를 만든 것



### Make the button click add an item

이제는 "Add" 버튼을 누르면 실제로 `TodoItem`을 추가하게 만들어 볼 것 

이를 위해서는, `TodoInputTextField`로부터 `text`에 접근할 수 있어야 함

<img src="/assets/13.png" width="500" >

TodoInputItem composition tree

`text` state가 `TodoItemInput` 에 노출 되어야 하고, 동시에 unidirectional data flow 를 사용해야 함

`TodoItemInput` 의 state는 `TodoInputTextField` 로 

`TodoInputTextField` event는 `TodoItemInput` 으로 

이를 위해서 자식 컴포저블인 `TodoInputTextField` 에서 부모 인 `TodoItemInput` 으로 상태를 옮겨야 함 

<img src="/assets/14.png" width="500" >

이러한 패턴을 **state hoisting** 이라고 함

상태를 hoist 하기위해 `TodoInputTextField` 에 (value, onValueChane) 파라미터를 추가

```kotlin
@Composable
fun TodoInputTextField(text: String, onTextChange: (String) -> Unit, modifier: Modifier) {
  TodoInputText(text, onTextChange, modifier)
}
```

이렇게 상태가 hoist 되고나면, 몇가지 중요한 속성을 가짐

- **Single source of truth** - 상태를 복제하는대신 이동함으로, 텍스트의 source 는 단 하나 뿐임을 보장. 버그 피할수있음
- **Encapsulated** - 오직 `TodoInputItem` 만이  state를 변경할 수 있음, 다른 component에서는 `TodoItem`에 이벤트를 보낼 수 있음. 이렇게 hoisting 하면, 오직 하나의  composable 만이 stateful 함. 다른 composable 들이 해당 state를 사용하더라도
- **Shareable** - immutable 한 값을 다수의 composable과 나눌 수 있음. 이 예제에서는 state를 `TodoInputTextField`와 `TodoEditButton` 에서 함께 사용 
- **Interceptable** - `TodoItemInput` 이 state 를 변경하기 전에 이벤트를 무시하거나 수정할 수 있음.
- **Decoupled** - `TodoInputTextField` 를 위한 state는 어디에던지 저장될 수 있음. `TodoInputTextField` 의 수정없이 문자를 입력할 때마다 업데이트 되는 Room db 를 통해 state를 지원하도록 선택할 수 있음



이제 state를 `TodoItemInput` 에 추가하고 `TodoInputTextField`에 넘긴다.

```kotlin 
@Composable
fun TodoItemInput(onItemComplete: (TodoItem) -> Unit) {
   // state
   val (text, setText) = remember { mutableStateOf("") }
   Column {
       Row(Modifier.padding(horizontal = 16.dp).padding(top = 16.dp)) {
           TodoInputTextField(
               text = text,
               onTextChange = setText,
               modifier = Modifier
                   .weight(1f)
                   .padding(end = 8.dp)
           )
           TodoEditButton(
               // 동일한 state인 text를 사용
               onClick = {
                 onItemComplete(TodoItem(text)) // send onItemComplete event up
       					 setText("") // clear the internal text 
               },
               text = "Add",
               modifier = Modifier.align(Alignment.CenterVertically),
               enabled = text.isNotBlank() // enable if text is not blank
           )
       }
   }
}
```



## Dynamic UI based on state

state를 기반으로 동적인 UI를 구축할 것 

ReComposibion은 새로운 데이터에 기반해 composition tree의 구조를 변경할 수 있음 



compose 에서 "visibility" 프로퍼티는 존재 하지 않음

compose는 동적으로 composition을 변경 할 수 있으므로 visibility를 gone으로 설정할 필요가 없음

대신에 composition에서 composble 을 삭제하면 됨



## Extracting stateless composable

Statefull한 TodoItemInput Composable을 재 사용 하기위해 두가지 composable로 분리 

- 상태를 가지는(statefull) TodoItemEntryInput() 
- 상태를 가지고 있지 않은, 재사용할 수 있는 TodoItemInput()
  - UI와 관련된 코드들을 가짐



## Use State in ViewModel

state를 호이스팅할때, state를 어디에 두어야할지 결정하는데 도움이 되는 세가지 규칙

1. State는 적어도 state를 사용 또는 읽는 모든 composable들의 lowest common parent 까지는 호이스트되어야함 
2. State는 적어도 변경될 수 있는(또는 수정될 수 있는) 최고 수준으로 hoist 해야 함
3. 동일한 이벤트에 대해 두가지의 상태가 변경된다면 둘은 반드시 함께 hoist되어야 함



### Convert TodoViewModel to use mutableStateOf

이 섹션에서는 editor를 위한 상태를 `TodoViewModel` 에 추가할 것

```kotlin
// 새로운 MutableStateOf<List<TodoItems>> 를 만들고
// 프로퍼티 delegate 문법을 사용해서 이를 일반 List<TodoItem> 으로 변환함 
var todoItems: List<TodoItem> by mutableStateOf(listOf())
    private set
```

`MutableState` 는 idiomatic Kotlin을 염두에 두고 빌드됨, 그리고 property delegate 문법을 지원함

Composable에서도 사용할 수 있지만, `ViewModel` 같은 stateful 클래스 내에서도 사용할 수 있음

> ViewModel이 기존 view system에서도 사용된다면, 기존의 LiveData를 계속 사용하는 것 이 좋음
>
> Mutable state는 Compose가 읽으려고 사용



### Define editor state

이제 eidtor를 위해 state 를 추가할 것

todo text가 중복되는 것을 방지하기위해 list를 직접 편집할 것

```kotlin
class TodoViewModel : ViewModel() {
  private var currentEditPosition by mutableStateOf(-1)
  
  var todoItems by mutableStateOf(listOf())
  	private set
  
  val currentEditItem: TodoItem?
  	get() = todoItems.getOrNull(currentEditPosition)
}
```

composable이 `currentEditItem` 을 호출할 때마다, `todoItems` 와 `currentEditPosition`  모두의 변경을 observe 함 

둘중 하나의 값이 변경되면, composable이 새로운 값을 get 함

functional transform에 익숙하다면,  currentEditItem 은 currentEditPosition과 todoItems 모두에 의존하고, zip을 사용해서 결합하는 것과 동일

> State<T> transgormations이 동작하려면, 반드시 State<T> 객체로부터 state를 읽어야함
>
> currentEditPosition 을 일반적인 Int로 선언했다면, Compose에서는 변경사항을 관찰할 수 없음



## Reuse stateless composables