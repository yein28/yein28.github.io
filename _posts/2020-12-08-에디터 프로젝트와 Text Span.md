```
layout: post
title:  "에디터 프로젝트와 Text Span"
date:   2020-12-08
categories: [Android]
```

# 에디터 프로젝트와 Text Span

https://if.kakao.com/session/113 

세션을 듣고 정리



### 에디터 프로젝트

공통 에디터 프로젝트의 세가지 꼭지

- 데이터 호환 : pc, mobile 플랫폼 상관없이 동일한 문서를 만듦
- 서비스의 범용성 
- 플러그인 지원 : 원하는 기능만 집어넣을 수 있도록 



기존 모바일 APP에서는 모바일의 글만 수정가능 

데이터 관점에서 

- 모바일 APP의 경우 HTML의 내용을 모두 이해 x

- PC web은 HTML을 통째로 집어넣어도 ok



문제 해결을 위해 공통 데이터 모델 규칙을 만듦

공통 데이터 모델 

web에서 작성 글 <-> 데이터 모델 <-> App에서 작성한 글

장점 

- 모든 플랫폼에서 동일한 형태의 글을 작성할 수 있음
- 동일한 에디터를 사용한다면 다른 서비스와의 호환 가능



### Android Text Span

Span이란? 

- 문자나 단락 수준에서 텍스트를 꾸미기 위한 마크업



일반적인 TextView에 스타일을 지정하는 경우, 텍스트 뷰 전체에 적용된다

Span 의 경우 xml설정 기반으로 원하는 부분에만 Span을 통해 꾸밀 수 있음

![image-20201208205806979](/Users/yein/Library/Application Support/typora-user-images/image-20201208205806979.png)

TextView - SpannedString, SpannableString

EditText - SpannableStringBuilder

적용된 모습은 동일함

그러면 차이점? 

- 경계의 처리방법이 다름 -> 영상 11:00 쯤



`getSpan` 을 이용해 적용된 Span 클래스들을 얻어올 수 있음

`val spans = spannableString.getSpans(6, 8, Any::class.java)`



`removeSpan` 으로 특정 영역에서만 span을 삭제 

- 해당 span이 적용된 모든 부분이 삭제되므로, 원하는 범위만 삭제 하고 싶은경우에는 따로 계산이 필요함 



#### underlineSpan 주의점

- 특정 키보드가 키워드 추천등의 이유로 underline을 사용
  - 커스텀된 UnderlineSpan을 만들어 해결 
  - 키보드별 동작이 달라 해결 못한 문제 



#### 그 외 Span으로 할 수 있는 것들 소개

-  ImageSpan - 이미지 뷰 추가 없이 이미지 삽입 가능
- 카카오 이모티콘 SDK - 이모티콘 표시 위한 뷰 제공, Span이용해 이모티콘 삽입 기능 지원
- Clickable Span
  - 텍스트의 특정 영역에 클릭 이벤트 지정, URLSpan도 이 클래스 상속
  - 반드시 `LinkMovementMethod` 를 지정해줘야함

- TTS Span(API 21)
  - 텍스트 특정 영역에 TTS 명확하게 지정 가능
  - 2020.11.18 -> 날짜를 이공이공.. 대신 이천이십년 ~ 으로 읽어줌