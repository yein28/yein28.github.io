---
layout: post
title: "Jetpack Compose basics codelab"
date: "2020-12-20"
categories: [Android]
---

# Jetpack Compose basics codelab

- Compose 는 무엇인지
- Compose로 UI를 만드는 법
- Composable 함수에서 state를 다루는 법
- Compose에서 Data flow 원칙



Text의 배경에 색을 넣고 싶은 경우 `Surface` 정의

```Kotlin
@Composable
fun Greeting(name: String) {
  Surface(color = Color.Yellow) {
    Text(text = "Hello $name!")
  }
}
```



**Modifiers**

대부분의 Compose UI 요소들은 옵셔너하게 modfier 파라미터를 받는다.

Modifier parameters tell a UI element how to lay out, display, or behave within its parent layout. Modifiers are regular Kotlin objects.

변수에 할당하고 재사용 가능

modifier들을 factory-extention 함수를 이용해 체이닝하거나 `then`으로 하나의 아규먼트로 합칠 수 있다.



```Kotlin
/**
 * Composes the given composable into the given activity. The [content] will become the root view
 * of the given activity.
 *
 * [Composition.dispose] is called automatically when the Activity is destroyed.
 *
 * @param parent The parent composition reference to coordinate scheduling of composition updates
 * @param content A `@Composable` function declaring the UI contents
 */
fun ComponentActivity.setContent(
    // Note: Recomposer.current() is the default here since all Activity view trees are hosted
    // on the main thread.
    parent: CompositionReference = Recomposer.current(),
    content: @Composable () -> Unit
): Composition {
    GlobalSnapshotManager.ensureStarted()
    val composeView: AndroidOwner = window.decorView
        .findViewById<ViewGroup>(android.R.id.content)
        .getChildAt(0) as? AndroidOwner
        ?: AndroidComposeView(this).also {
            setContentView(it.view, DefaultLayoutParams)
        }
    return doSetContent(composeView, parent, content)
}

```



**Making container functions**

앱의 Common 설정들을 가지고 있는 컨테이너를 만드려면 어떻게 해야할까?

generic container를 만드려면, `Unit` 을 반환하는 Composable 함수를 파라미터로 받는 Composable 함수를 만든다. (Composable 함수는 UI 컴포넌트를 반환하지 않기때문에 반드시 Unit을 리턴해야 함)

```kotlin
@Composable
fun MyApp(content: @Composable () -> Unit) {
  MyAppTheme {
    Surface(color = Color.Yellow) {
      content()
    }
  }
}
```

가독성, 재사용성 향상



**Calling Composable functions multiple times using Layouts**

UI 컴포넌트를 Composable 함수로 만들었기때문에, 중복되는 코드 없이 재사용할 수 있음

> Divier를 이용하면 수평 구분선을 만들 수 잇음



## State in Compose

Compose에서는 앱의 데이터 변경을 observe하기위한 툴을 제공하며 자동으로 함수를 재 호출 함 - recomposing

composable에 internal state를 추가하기 위해 `mutableStateOf` 함수를 사용, composable mutable memory를 부여하는.

매 recomposition 마다 다른 상태를 가지지 않게 하기 위해서는, `remember`를 사용

composable이 화면 여러곳에 여러 인스턴스로 존재하는경우, 각 카피는 고유한 버전의 샅애를 얻음

internal state를 클래스의 private 변수라고 생각해도 됨

```kotlin
/**
 * Return a new [MutableState] initialized with the passed in [value]
 *
 * The MutableState class is a single value holder whose reads and writes are observed by
 * Compose. Additionally, writes to it are transacted as part of the [Snapshot] system.
 *
 * @param value the initial value for the [MutableState]
 * @param policy a policy to controls how changes are handled in mutable snapshots.
 *
 * @see State
 * @see MutableState
 * @see SnapshotMutationPolicy
 */
fun <T> mutableStateOf(
    value: T,
    policy: SnapshotMutationPolicy<T> = structuralEqualityPolicy()
): MutableState<T> = SnapshotMutableState(value, policy)

/**
```



**Source of truth**

Composable 함수에서, 함수 호출에 유용한 state는 반드시 노출되어야한다. 왜냐하면 소비하거나 제어할 수 있는 유일한 방법이기때문에 - 이러한 프로세스를 hoisting이라고 부름

State hoisting은 호출한 함수에의해서 내부 상태를 제어할 수 있게 만드는 방법

제어된 composable 함수의 파라미터를 통해서 상태를 노출하고, 제어하는 composable에서 외부적으로 이를 인스턴스와 하여 수행할 수 있다. 

예를들어, `Counter`의 소비자가 state에 관심이 있는경우, Counter의 파라미터로 (count, updateCount)를 도입하여 호출자에게로 defer(연기)할 수 있다.

이렇게 하면 `Counter`는 본인의 상태를 hoisting할 수 있다.

```kotlin
@Composable
fun MyScreenContent(names: List<String> = listOf("Android", "there")) {
    val counterState = remember { mutableStateOf(0) }
    Column {
        for (name in names) {
            Greeting(name = name)
            Divider(color = Color.Black)
        }
        Divider(color = Color.Transparent, thickness = 32.dp)
        Counter(
            count = counterState.value,
            updateCount = { newCount ->
                counterState.value = newCount
            }
        )
    }
}

// hoisting을 위해 value(state)와 onValueChanged이벤트가 추가될 수 있다.
@Composable
fun Counter(count: Int, updateCount: (Int) -> Unit) {
    Button(onClick = { updateCount(count + 1) }) {
        Text("clicked $count times")
    }
}
```

hoisting 개념 - https://developer.android.com/jetpack/compose/state

Composable 에 상태가 있는경우 hoisting(상태 끌어올리기)를 통해 stateless로 만들 수 있음

hoisting은 컴포저블의 내부 상태를 파라미터와 이벤트로 대체하여 상태를 composable의 호출자로 옮기는 패턴



## Theming your app

미리 정의된 스타일을 copy함수를 이용해서 변경할 수 있음 - 일반 kotlin data 클래스 이므로

ex) `style = MaterialTheme.typography.body1.copy(color = Color.Yellow)`