# required init 날 왜 계속 괴롭히니..

## 211125_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### `required init(coder: )` 
왜 `required init(coder: )` 는 기본적으로 `fatalError()`를 제공해줄까?
왜 상위의 이니셜라이저인 `super.init()`을 제공하지 않고, 앱이 꺼지게 만드는 `fatalError`를 제공하는지 찾아보았다. 

`fatalError`를 기본적으로 제공하는 이유는 다음과 같다. 

 - `fatalError`가 있는 이유는 아카이브 해제(unarchive)를 위한 일반적인 init를 제공할 방법이 없기 때문이다.

> archive : xib 모델 객체를 저장하기 위해 객체의 프로퍼티를 기록하고 파일 시스템에 저장하는 방식 (NSCoder의 encoder)
> unarchive : archive한 데이터로부터 객체를 생성 (NSCoder의 decoder)
	
즉 간단하게 정리해보자면, `fatalError`를 던져주는 것은 프로그래머에게 자율적으로 맡기는 부분이라고 생각해봐도 좋을 것 같다. 

그러니 `fatalError()`를 제거하고 init 함수의 목적을 달성하기 위해 필요한 코드를 작성해야 한다는 것이다. 이 `fatalError`는 단지 자신의 코드를 작성하는 것을 잊었을 때 테스트를 통해서 찾을 수 있게 도와주는 역할을 하는 것이다. 

결국은 `fatalError`를 그대로 두면 안된다는 것!!

## 고민된 점 
-  `required init(coder: )` 은 왜 `fatalError`를 기본적으로 제공하는가?

## 해결 방법 
- 검색 후 학습 내용에 정리!
---

**Ref**
[initialization in swift](https://developer.apple.com/forums/thread/8355)
