# loadViewIfNeeded, UIScrollView, Type Method and Memory

## 211210_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. `loadViewIfNeeded`

View Controller의 view가 아직 로드되지 않았을 때 메모리에 올려주는 역할을 한다. 

숲재, 나무, 아리와 이야기를 나누다가 알게되었는데, 이 메서드는 테스트를 위해 일반적으로 사용된다고 한다. 즉 프로덕션 코드에서 임의로 로드하기 위해서 사용하는데 사용되지는 않는 것 같다. 

비슷해보이는 `loadView()` 메서드는 `You should never call this method directly.`라고 문서에서 강력하게 말하고 있다. 

즉 라이프 사이클을 임의대로 부르는 것이 좋지 않다는 것을 말하는 것 같기도 하다!

### 2. (미해결) UIScrollView의 Content Layout Guide

나무, 숲재와 이야기를 나누다가 scrollview의 content layout guide 크기가 왜 케이스마다 어떤 경우는 safe area를 무시하고, 아닌 경우가 있는지 궁금했다. 
 
 상황은 다음과 같다. 
 
- 내부 content view가 스크롤 가능하지 않을 정도로 짧은 경우에는 빌드 이후 화면을 확인해보면 safe area를 넘어서서 superview 까지 영역이 잡힌다.
- 내부 content view가 스크롤 가능하게 될 정도로 길어지면 빌드 이후 화면을 확인해보면 safe area를 지키면서 화면이 보여지게 된다. 

이 부분은 아직 원인을 찾지 못했지만, 추측하기에는 ScrollView 내부의 Content View의 길이가 frame을 벗어나게되는 경우 safe area를 넘어서게 되면 컨텐츠가 가려질 위험이 있기에 safe area 내부에 위치시키는 게 아닐까 추측을 해보았다... 

이 부분은 좀 더 알아보고 학습해보면 좋을 것 같다. 

(사실 사용하는데에는 크게 문제가 없으나, 뷰들을 색칠해서 보면 그 차이가 보여서 언젠가 문제가 될 것 같긴하다..)

### 3. 타입 메서드와 메모리

사용자 정의 타입을 구성할 때 class, struct로 만들어서 내부에 메서드를 작성하게 된다. 이 때 인스턴스/타입 메서드로 크게 2가지 방법으로 작성할 수 있는데, 타입 메서드에 대해서 알아보자. 

타입 메서드는 인스턴스 없이도 호출할 수 있는 메서드이다. 즉 어떠한 객체에만 속해 있고, 그 객체가 인스턴스화 되지 않아도, 객체에 대해서 호출할 수 있는 것이다. 

그렇기 때문에 타입 메서드는 항상 메모리가 살아있다고 한다. 인스턴스가 생기면서 메모리에 올라오는 것과 달리, 이 부분에 큰 차이점이 있는 것 같다. 

이 때문에 메모리 관점에서 고려하여, 필요한 경우에만 인스턴스화하여 사용하도록 하는게 바람직한 경우도 있을 것 같다. 

예시로 util의 역할을 하는 JsonDecoder나 DateFormatter의 경우도 보면 필요할 때 인스턴스화 하여 사용도록 class 타입이며, 내부의 메서드는 인스턴스 메서드로 구현되어있다.

따라서 인스턴스 생성을 피하고 싶다는 이유만으로 타입 메서드를 작성하면 안될 것 같다.  

## 고민된 점 
- util 역할의 타입을 구현하는 방법
- scrollview와 content layout guide

## 해결 방법 
- 끝없는 검색과 고민 후 학습 내용에 정리해보기!

---

**Ref**

[loadViewIfNeeded](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621446-loadviewifneeded)

[loadView](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621454-loadview)

[Step3 리뷰](https://github.com/yagom-academy/ios-exposition-universelle/pull/117#discussion_r766491766)


