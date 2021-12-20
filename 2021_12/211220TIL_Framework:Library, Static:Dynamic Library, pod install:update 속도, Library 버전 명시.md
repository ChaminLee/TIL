# Framework/Library, Static/Dynamic Library, pod install/update 속도, Library 버전 명시

## 211220_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용


### !복습 내용!
 
- `Thread safe`하다는 것은 `atomic`하다는 것이다. 
	- 즉 swift 기본 타입은 `thread-unsafe`하기에 `atomic` 하지 않다!

### 1. 프레임워크 vs 라이브러리

프레임워크와 라이브러리를 어떠한 기준으로 구분해 낼 수 있을까?

- 없어도 만들 수 있는가?
	- 있으면 `라이브러리`
	- 없으면 `프레임워크`

라이브러리는 가져다 쓰며 도움을 받는 것, 즉 `도구`와 같은 느낌이다. 프레임워크는 기반이 되는 코드 묶음으로 `틀`이라고 이해해도 좋을 것 같다. 

### 2. Static / Dynamic library

![](https://minsone.github.io/image/2019/10/2.png)

- Source file과 static libraries를 병합하는 과정을 `Link`라고 한다. 
- 이제 application file에 앱 소스 코드가 복사된다. (static libraries를 포함)
- 앱이 실행되면 Source file + static libraries 를 포함한 코드들이 앱의 주소 공간 (Heap)에 할당된다. 

> static library의 경우 직접적으로 코드가 복사되고 큰 메모리 공간을 가지기 때문에 조금 느릴 수 있다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbuKhO7%2FbtqyCJ48JP5%2FHKruKI6gkJr8OOBGBtiV61%2Fimg.png)

- Static과 동일하게 `Link` 작업을 진행한다. 
- 하지만 library 코드를 복사하지 않고 dynamic library에 대한 참조만 application file에 포함된다. 
- 이제 앱이 실행될 때 앱 주소 공간에 dynamic library가 올라간다. 
 
### 3. `pod install`, `pod update`가 느린 이유 

 CocoaPod의 경우 라이브러리를 master repository에 관리한다고 한다. `install / update`을 할 경우 마스터에 가서 해당하는 라이브러리 이름을 기준으로 `pod`을 찾아오게 된다. 하지만 master는 매번 업데이트 되기 때문에, 이를 찾는데 시간이 엄청 걸리게 되는 것이다. CocoaPod과는 달리 Carthage, SPM은 주소값으로 라이브러리를 찾기 때문에, 이름으로 찾는 CocoaPods의 경우보다 빠르다고 볼 수 있다. 

### 4. 라이브러리 버전을 명시했을 때 장점

라이브러리의 버전을 명시를 해야 원하는 환경에서 동작하도록 할 수 있다. 

예를 들어 CocoaPods를 사용해서 의존성을 관리하고 있고, podfile에 A 라이브러리의 이름만 적어뒀다고 해보자!

그렇다면 이후에 해당 라이브러리의 최신 버전이 있을 경우 `pod install`을 하게 되면 자동으로 버전이 올라가게 될 것 이다. 그렇다면 불가피하게 라이브러리가 업데이트 되면서 원치 않는 상황이 발생할 수 있다. 

극단적인 예로 어떤 메서드 지원을 앞으로 하지 않겠다고 이후 버전에서 선언을 했다면, 업데이트 이후에 해당 메서드를 사용할 수 없을 것이다. 

이 뿐만 아니라 유사한 작은 문제들이 발생할 수 있기에 버전을 명시함으로써 환경을 제어하는 것도 중요할 것 같다. 

## 고민된 점 
- 의존성 관리 도구의 장단점
- `pod install`은 왜 느린가?

## 해결 방법 
- WWDC 및 야곰에게 질문!
---

**Ref**

[(Static/Dynamic) Library](https://zeddios.tistory.com/1308)

https://minsone.github.io/mac/ios/managed-third-party-library-static-dynamic

[야곰닷넷 오픈소스 라이브러리 만들기 코스](https://yagom.net/courses/open-source-library/lessons/%ec%bd%94%ec%bd%94%ec%95%84%ed%8c%9f-vs-%ec%b9%b4%eb%a5%b4%ed%83%80%ea%b3%a0-vs-%ec%8a%a4%ec%9c%84%ed%94%84%ed%8a%b8-%ed%8c%a8%ed%82%a4%ec%a7%80-%eb%a7%a4%eb%8b%88%ec%a0%80-2/)

