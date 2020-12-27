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

<img src="/assets/1.png" width="500" >

타이틀을 포함해서 많은 항목들을 커스텀할 수 있음

<img src="/assets/2.png" width="500" >

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



## Create your custon layout

수동으로 childern을 measure & 배치 해야 할 수 있음, 이때는 `Layout` composable을 사용

> 기존의 View System에서는 cusom layout을 만드려면. Viewgroup을 extend하고 meaure와 layout 함수들을 구현해야 함. Compose에서는 Layout composable을 사용해서 간단하게 함수를 작성하면 됨

커스텀 레이아웃을 작성하는 방법을 알아보기 전에, Compose에서 Layout의 원칙에 대해 알아야 함



### Principles of layout in Compose

Some composable functions emit a piece of UI when invoked that is added to a UI tree that will get rendered on the screen. 각 emission(or element)는 하나의 parent와 잠재적인 많은 children을 가짐. 또한 parent내에서의 location: (x,y)position 과 크기: `width` , `height` 를 가짐



**Compose UI는 multi-pass measurement를 허락하지 않는다.**

- layout element가 다른 measurement configuration을 위해 두번이상 자식을 측정할 수 없음을 의미 
- 성능에 좋음 



###  Using the layout modifier

element의 measure와 배치를 컨트롤하기위해 `layout` 모디파이어를 사용

보통 custom `layout` modifier 구조는 다음과 같음

```kotlin
fun Modifier.custonLayoutModifier(...) = Modifier.layout { measurable, constraints
  ...
}
```

`layout` 모디파이어를 사용하면, 두가지 람다 파라미터를 얻음

- `measurable` : 측정되고 배치될 자식
- `constraints` : 자식의 최소, 최대 width와 height



만약 base line 부터 top까지의 거리를 조정하고 싶다면. `layout` 모디파이어를 사용해서 composable을 배치 해야함

<img src="/assets/3.png" width="500" >

`firstBaselineToTop` modifier를 만들어보자

```kotlin
fun Modifier.firstBaselineToTop(
	firstBaselineToTop: Dp
) = Modifier.layout { measurable, constraints ->
  
}
```

첫번째로 composable을 measure해야 함. 이전에 언급한 것 처럼 자식은 오직 한번만 measure할 수 있음

`measurable.measure(constraint)` 를 호출해서 composable을 measure한다. `measure()` 호출의 결과는 `placeRelative(x,y)  ` 를 호출해 배치될 수 있는`Placeable`  임



composable이 measure된 이후에, 그것의 사이즈를 측정하고 content를 배치하기 위해 람다를 받는 `layout(width, height)`을 호출해서 명시해야함

```kotlin
fun Modifier.firstBaselineToTop(
	firstBaselineToTop: Dp
) = Modifier.layout { mearusable, constarints ->
  val placeble = measurable.measure(constrains)
                     
  // Check the composable has a first baseline
  check(placeable[FirstBaseline] != AlignmentLine.Unspecified)
  val firstBaseline = placeable[FirstBaseline]

  // Height of the composable with padding - first baseline
  val placeableY = firstBaselineToTop.toIntPx() - firstBaseline
  val height = placeable.height + placeableY
  layout(placeable.width, height) {
    // composable이 배치되는 위치
    placeable.placeRelative(0, placeableY)
  }
}
```

이제는 `placeable.placeRelative(x,y)`를 호출해서 composable을 배치할 수 있음



### Using the Layout compsable

single composable이 measure되고 화면에 배치되는 방식을 컨트롤하는 대신에, Composable들의 그룹에 대해 동일한 필요성이 있을 수 있음. 이를 위해 `Layout` composable을 사용하여 layout의 자식을 measure하고 배치하는 방법을 수동으로 컨트롤할 수 있음

```kotlin
@Composable
fun CustomLayout(
	modifier: Modifier = Modifier,
  // custom layout attributes
  children: @Composable () -> Unit
) {
  Layout(
  	modifier = modifier,
    children = children
  ) { mearurable, constraints ->
    
  }
}
```

`Layout` 을 이용해 매우 기본적인 `Column` 을 구현해보겠음

### Implementing a basic Column

```kotlin
@Composable
fun MyOwnColumn(
    modifier: Modifier = Modifier,
    children: @Composable () -> Unit
) {
    Layout(
        modifier = modifier,
        children = children
    ) { measurables, constraints ->
        // Don't constrain child views further, measure them with given constraints
        val placeables = measurables.map { measurable ->
            measurable.measure(constraints)
        }

        // 자식을 수직으로 배치하기 위해 기록
        var yPosition = 0

       	// layout(width, height)를 호출해서 our own Column사이즈를 지정하면, 
        // 자식을 배치하는데 사용되는 람다도 제공함
        layout(constraints.maxWidth, constraints.maxHeight) {
            // Place children in the parent layout
            placeables.forEach { placeable ->
                // 화면에 아이템을 배치
                placeable.placeRelative(x = 0, y = yPosition)

                // Record the y co-ord placed up to
                yPosition += placeable.height
            }
        }
    }
}
```

