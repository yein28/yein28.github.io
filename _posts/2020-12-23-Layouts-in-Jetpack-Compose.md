---
layout : post
title : "Layouts in Jetpack Compose"
date : 2020-12-23
categories : [Android]
---

# Compose by example

https://www.youtube.com/watch?v=DDd6IOlH3io&feature=emb_imp_woyt&ab_channel=AndroidDevelopers

https://github.com/android/compose-samples

compose를 사용한 공식적인 예제 앱들로 설명

## Themeing





# Layouts in Jetpack Compose

## Slot APIs

Slot APIs는 customization(맞춤화) 레이어를 composable 위에 가져오기위해 Compose에서 도입한 패턴이다.

Material Button을 생각해보면, Button의 모양과 내용에 대한 가이드라인이 있음.

그러나 종종 원하는것을 구현하기위해 컴포넌트들을 커스텀해야하는 경우가 있음, 커스텀을 위해 파라미터를 추가하면 추가할수록, 통제할 수 없어짐

컴포넌트 커스텀을 위해 여러 파라미터를 추가하는 대신에, 우리는 Slots을 추가했음

**Slot은 개발자가 원하는 대로 채울 수 있도록 UI에 빈공간을 남겨둠**



예를 들어서 Button 케이스에서, 아이콘과 텍스트가 있는 행을 삽입하려는 사용자가 Button 내부를 채울 수 있음

이를 가능하게 하기 위해서, 자식 composable 람다를 받는 Button API를 제공 함(`content: @Composable () -> Unit`)

이는 Button에서 내가 정의한 compsable을 emitted할 수 있도록 함

```kotlin
@Composable
fun Button(
	modifier: Modifier = Modifier.None,
  onclick: (() -> Unit)? = null,
  ...
  content: @Composable () -> Unit
)
```



Compose 에서는 Top App Bar같은 복잡한 구조에서 Slot을 헤비하게 사용하고 있음

<img src="/assets/1.png" width=500px\>

타이틀을 포함해서 많은 항목들을 커스텀할 수 있음

<img src="/assets/2.png" width=500px\>

예시

```kotlin
TopAppBar(
	title = {
    Text(text = "title", maxLines = 2)
  },
  navigationIcon = {
    Icon(myNavIcon)
  }
)
```

own 컴포저블을 만들때, Slots API 패턴을 사용해서 more reusuable 하게 만들 수 있음



## Material Components

Compose는 앱을 만들 때 사용할 수 있는 Material Component Composable과 함께 제공된다.

가장 high-level composable은 `Scaffold` 이다.

### Scaffold

`Scaffold` 는 기본적인 Material Design layout구조를 가지고 UI를 만들 수 있게 해줌.

TopAppBar, BottomAppBar, FAB, Drawer 같은 가장 흔한 top-level 머터리얼 컴포넌트들의 slot을 제공함



### Placing modifiers

새로운 composable을 만들 때 마다, `Modifier ` 를 디폴트로 가지는 `modifier` 파라미터를 가지는 것은 composable을 재사용 할 수 있게만드는 좋은 방법이다.

`BodyContent` Composable은 이미 modifier를 파라미터로 가지고 있다. 만약 extra padding을 BodyContent에 추가하고 싶다면, padding modifier는 어디에 두어야 할까?

두가지 방법이 가능하다 :

1. modifier를 컴포저블 내 direct child에만 적용하면 모든 `BydyConent` 호출에 extra padding 이 적용 됨

   ```kotlin
   @Composable
   fun BodyContent(modifier: Modifier = Modifier) {
     Column(modifier = modifier.padding(8.dp)) {
       Text(text = "Hi there!")
     }
   }
   ```

2. 필요할때에만 extra 패딩을 추가하는 composable을 호출할때에만 modifier를 적용

   ```kotlin 
   @Composable
   fun LayoutsCodelab() {
     Scaffold(...) { innerPadding ->
     	BodyContent(Modifier.padding(innerPadding).padding(8.dp))
     }
   }
   ```

어디서 적용할 것인지는 composable의 타입과 유즈 케이스에 따라 다르다.

Modifiers can be chained by calling each successive modifier function on the previous one.

가능한 체이닝 메서드가 없다면 `.then()` 을 사용해라. 