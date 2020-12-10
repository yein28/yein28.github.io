---
layout : post
title: "Fragment manager"
date: 2020-12-10
categories: [Android]
---

# Fragment manager

https://developer.android.com/guide/fragments/fragmentmanager

> 참고 : 우리는 앱의 navigation을 위해서 Navigation library를 사용하기를 강력하게 권장함, 해당 프레임워크는 프래그먼트, 백스택, 프래그먼트 매니저와의 작업에 대한 베스트 프랙티스를 따름. 

`FragmentManager` 는 앱의 프래그먼트에 대해 추가, 삭제, 교체, 백스택에 추가 등의 작업을 수행하는 클래스임

 

Jetpack Navigation 라이브러리를 사용한다면 `FragmentManager`와 직접적으로 상호작용할 일은 없을 것(라이브러리가 대신 해줌)

즉, 프래그먼트를 사용하는 앱은 일정 수준에서는 `FragmentManater`를 사용하기때문에, 이것이 무것이고 어떻게 작동하는지를 이해하는 것은 중요함



이 토픽은 `FragmentManager`에 어떻게 접근하는지, 액티비티와 프래그먼트와 관련해서 `FragmentManager` 의  역할은 무엇인지,  `FragmentManager` 로 백스택 관리하기, 그리고 프래그먼트에 데이터와 의존성을 제공하는 방법에 대해서 다룸



## Access the FragmentManager

**Accessing in an activity**

모든 `FragmentActivity`와  `AppCompatActivity` 같은 서브 클래스는 `getSupportFragmentManager()` 메소드를 통해  `FragmentManater`에 접근함



**Accessing in a Fragment**

Fragment는 또한 하나 이상의 자식 프래그먼트를 호스팅할 수 있따. 프래그먼트 안에서 `getChildFragmentManager()`를 통해서 자식 프래그먼트를 관리하는 `FragmentManager` 에 대한 참조를 얻을 수 있음

호스트 `FragmentManager` 에 접근해야한다면, `getParentFragmentManager()` 를 사용할 수 있음

프래그먼트와 호스트 그리고 FragmentManager 인스턴스 간의 관계를 확인하기 위해 몇가지 예를 살펴봄

![image-20201210215834585](/Users/yein/Library/Application Support/typora-user-images/image-20201210215834585.png)

단일 Activity 호스트가 있는 예시 

