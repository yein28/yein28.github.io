---
layout : post
title : "Thinking in Compose"
date : 2020-12-15
categories : [Android]
---

# Thinking in Compose

https://www.youtube.com/watch?v=SMOhl9RK0BA&feature=emb_imp_woyt&ab_channel=AndroidDevelopers

Compose는 완전히 코틀린으로 작성되었으며, 코틀린 언어의 스타일과 인간공학을 수용한다.


기존 뷰 시스템과의 상호 운용이 가능하며 기본 플랫폼에서 번들로 제공되지 않는 전적으로 사용자 공간에서 빌드되므로 원하는 시간에 기능 개선 및 버그 수정을 활용할 수 있습니다.
(It allows for interop with the existing view system, and it's build entirely in user space, unbundled from the underlying platform, which means you can take advantage of feature improvements and bug fixes on your own time)


## 1.Composition
Composable 함수는 보기에는 보통의 함수 같지만, 몇가지 중요한 차이점이 있음

**@Composable Functions can re-compose**

- Composable functions might be called again in order to update the state of the UI
- The Compose runtime might re-execute the Composable function after they've been called the first time in order to to update the current state of the UI
  - 우리는 이러한  프로세스를 recomposition 이라고 부름


사용자의 프로필을 만들때, 테두리에 랜덤으로 색깔을 넣어주는 함수가 존재,
이 함수는 composable 함수가 재 실행될때마 색깔이 변경될 것임

이러한 상황을 방지해 줄 수 있는 것이 있음

**@Composable Functions have a memory**
- Composable 함수는 마지막으로 호출된 항목에 접근 할 수 있음

- remember의 primitive function

  ```kotlin
  @Composble fun <T> remember(
    vararg inputs: Any?, // 로컬 캐시의 key라고 생각할 수 있음
    calculation: () -> T
  ): T
  ```

  `val ringColot = remember { randomColor() }`

  -> 반환값이 바뀌는 것이 아니 함수의 첫번째 결과를 기억할것을 명시



영상속의 예제 채팅 앱에서는 일반 텍스트만 뿐만아니라 클릭할수 있는 링크와 코드 블럭도 있음

그러므로 `Text()` 를 커스텀 텍스트 컴포넌트인 `ChatText()` 로 구현

```kotlin
// 1
@Composable fun ChatText(text: String) {
  Text(text, style = typography.body1)
}
```

Compose 에서는 text Composable의 두가지 오버로드를 제공함

```kotlin
@Composable fun Text(text: String,  ...)
@Composable fun Text(text: AnnotatedString, ...)
```

`text`를 -> `AnnotatedString`으로 변환하는 함수를 만듦 `fun parseAnnotated()`

```kotlin
// 2
@Composable fun ChatText(text: String) {
  val annotated = parseAnnotated(text)
  Text(annotated, style = typography.body1)
}
```

`parseAnnotated` 호출은 expensive할 수 있음, chatText는 composable 이므로 re-compose될 수 있음

-> remember 함수를 사용

이 케이스에서는, remember의 첫번째 파라미터를 사용할 것

```kotlin
// 3
@Composable fun ChatText(text: String) {
  val annotated = remember(text) parseAnnotated(text) // 텍스트가 변경 될 때에만 다시 계산
  Text(annotated, style = typography.body1)
}
```

**@Composable functions can be composed**

<img src="/assets/image-20201215191150240.png" width="300" height="300">

다음과 같이 많은 일을 하고 있는 함수에서 메시지의 컨텐츠와 관련있는 부분만 뽑아낼 수 있다.
이 모든 파라미터들을 하나의 Composable content lambda로 뺄 수 있다.
```Kotlin
@Composable fun ChatMessage(
  authorName: String,
  authorImageUrl: String,
  dateSent: String,
  onUserPhotoClick: () -> Unit,
  content: @Composable () -> Unit, // !!여기가 핵심
  )
```
ChatMessage composable을 단순, 그리고 유연하게 만든다.

## 2.State
TBD
