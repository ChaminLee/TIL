# Unit Test와 TDD 그리고 NavigationController

## 211104_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Unit Test

(야곰 닷넷의 Unit Test를 보고 정리해봅니다.)

Unit Test의 사전적 의미는 다음과 같다. 

> 소스 코드의 특정 모듈이 의도된 대로 정확히 작동하는지 검증하는 절차이다. 즉, 모든 함수와 메서드에 대한 테스트 케이스를 작성하는 절차를 말한다. 이상적으로, 각 테스트 케이스는 서로 분리되어야 한다. 이를 위해 가짜 객체(Mock object)를 생성하는 것도 좋은 방법이다. 
> -wiki

테스트 코드를 짜야한다라는 말은 많이 들어봤지만 어떠한 이유에서 테스트 코드를 짜야할까? 

우선 가장 중요한 키워드는 `안정성`과 `설계` 측면에서 이점이 있기 때문이지 아닐까 싶다. 여러 문서들을 보면 테스트 코드를 작성함으로써 얻을 수 있는 장점을 다음과 같이 설명하고 있다.

 - 스펙을 문서화
 - 유지보수에 유리
 - 클린 코드 작성

주로 이러한 이점들을 얻기 위해 귀찮음을 무릅쓰고 테스트 코드를 작성하는 것 같다. 이왕 테스트 코드를 짜기로 했으면 테스트 코드도 잘 짜야할텐데 어떤 가이드라인, 원칙이 있는지 봐보자. 

> FIRST 원칙

1. First 
	- 테스트는 빠르게 동작되어야 한다. 
	- 테스트 코드를 실행하는데만 시간이 오래 걸린다면 주객전도의 상황이 될 수도 있다. 
2. Independent
	- 각각의 테스트 코드는 서로 독립적이어야 한다. 
	- 최소한의 단위 테스트는 스스로에게만 집중할 수 있도록 해줘야 한다. 
	- 즉, 앞선 A 테스트 코드의 결과에 따라 B 테스트 코드가 영향을 받으면 안된다는 것이다. 
3. Repeatable
	- 언제 실행하던 항상 같은 결과가 반환되어야 한다. 
	- 이를 위해 주변을 통제해주는데, 이를 위해 통제가 어려운 부분은 테스트를 위한 객체를 만들어준다고 한다. 
4. Self-Validating
	- 테스트 코드는 Bool 타입을 활용하여 스스로 검증할 수 있어야 한다.
	- 즉 테스트 코드 내부에서 값과 값을 비교하던가, 다른 방식으로 판단할 수 있어야 한다. (ex. `XCTAssertEqual`로 값 비교)
6. Timely
	- 테스트 가능한 상황을 만들기 위해, 테스트 코드를 먼저 설계/작성하고 프로덕션 코드를 설계/작성해야한다. 
	- 만약 프로덕션 코드를 먼저 작성한다면, 이미 테스트가 불가능 코드가 되어있을 수 있다. 

이 `FIRST` 원칙을 지켜야한다고 말은 하지 못하지만, 지키려고 노력하는게 중요하다고 한다!

--- 

테스트 파일을 만들었다면 초기에 기본적으로 적혀있는 코드들이 있다. 

`import XCTest` : 각종 테스트(Unit, Performance, UI Test를 위해 필요한 프레임워크이다. 

`: XCTestCase` : 테스트를 작성하기 위해 상속해야하는 가장 기본적인 클래스이다. 

`setUpWithError()` : 각 테스트 코드가 실행되기 전 모두 같은 상태 조건에서 실행될 수 있도록 만들어준다. 

`tearDownWithError()` : 각 테스트 코드 실행이 끝난 이후 호출되는 메서드이다. 주로 `setUpWithError()`에서 설정된 값들을 다시 초기화해주는데 사용된다고 한다!

이제 메인이 되는 테스트 코드의 메서드명은 항상 `test`로 시작해야한다. `test_blah_blah`, `testCode`이런 식으로 작성해줘야 인식이 되고 라인 넘버 대신 성공/실패를 나타내는 마름모(거터라고 부른다고 한다)가 뜨게 된다! 
(처음에 `test` 안적고 네이밍 했다가 아무 결과를 얻지 못했다...)

그리고 테스트 코드를 작성할 때는 BDD(Behavior Driven Development)의 방법에서 차용하여 given, when, then 순서로 작성하는 것이 좋다고 한다. 

- given: 어떤 상황이 주어지고
- when: 어떤 코드를 실행하고
- then: 테스트 결과를 확인하는

> 그래서 테스트 코드 파일은 다음과 같이 실행된다고 한다. 
>  `setUpWithError()`  -> 테스트 코드 1 -> `tearDownWithError()` -> `setUpWithError()`  -> 테스트 코드 2 -> `tearDownWithError()` -> `setUpWithError()`  -> 테스트 코드 3 -> `tearDownWithError()` ... 

> Tip. 프로덕션 코드 파일의 target membership에 test를 추가해줘야 한다!

> 메뉴바 > Product > scheme > edit scheme: Test > Options > Code Coverage: all target 이후 종료 
> 이후 테스트 코드를 실행 후 좌측 네비게이터의 가장 오른쪽 아이콘을 누르면 coverage가 뜨는데, 테스트 코드가 현재 대상이 되는 파일을 얼마나 커버하고 있는지를 수치로 알려준다. 파일을 누르면 빨간 음영으로 커버되지 않은 곳을 알려주기도 한다!!

### TDD

한 마디로 테스트 케이스를 작성한 후, 이를 통과할 수 있는 기능을 만드는 것이다. 

**테스트 주도 개발 사이클**
1. 실패하는 케이스를 만들어 테스트를 한다. 
2. 통과할 수 있는 기능을 추가 개발한다.
3. 해당 코드를 개선한다(리팩터링)

---

- 실패할 수 있는 가능성을 내포하고 시작하는 것. 
- 빠르게 실패하고 빠른 피드백을 받고 개선하는 것이 가장 바람직하다. 코드도 마찬가지! 

- 장점
	- 테스트는 훌륭한 스펙 정의 문서가 된다!
	- 테스트를 하지 않으면 코드 리팩토링이 어렵고 두려워진다. 
- 스펙을 알고 코드를 작성해야한다. 즉 요구사항을 모두 이해하고 테스트를 작성하고 코드를 작성해야 무엇을 해야하는지 정확하게 이해하고 진행할 수 있다. 
- 따로 테스트 할 수 있어야, 각각에서 문제의 원인을 찾을 수 있다. 
- 테스트 코드 끼리도 영향을 미치면 안된다. (같은 객체를 사용하면 안되는것, = 독립적이어야 한다)
- 테스트 코드는 모든 좋은 코드의 시발점이 된다. 

> Tip. Singleton의 단위 테스트가 어려운 이유는 테스트 코드끼리 독립적이지 않기 때문이다. Singleton 객체에 서로 연결이 되어있기 때문에 A 테스트 코드 실행시 B 테스트 코드에 영향을 줄 수 있기 때문에 테스트가 어려운 이유가 된다. 

### Queue

[Queue란?](https://leechamin.tistory.com/409?category=984965)
FIFO 선입선출

들어올 때 put (insert)
나갈 때 get (delete)

### Stack

[Stack이란?](https://leechamin.tistory.com/408?category=984965)
LIFO 후입선출(=선입후출)

들어올 때 push (insert)
나갈 때 pop (delete)


## 고민된 점 

UINavigation을 present하기 위해 덮어 씌웠을때 (`UINavigation(rootViewcontroller: ViewController)`) vs 목적 뷰컨에서 `self.navigationController`하는 경우 두 navigation은 같은 걸까?


## 해결 방법 

우선 타이틀 값을 비교해봤다. 

- `UINavigation.navigationBar.topitem?.title` 과 VC에서의 `self.title`는 다르다. 
   - 전자는 `navigationBar`의 프로퍼티, 후자는 `UIViewcontroller`의 프로퍼티이다
 - `UINavigation`을 만들어서 `present` 해줄 때나(presenting), 해당하는 뷰(presented), 스토리보드에서나 모두 `UINavigation.navigationBar`를 공유한다?
 - view life cycle에서 가장 마지막에 `navigationBar`를 초기화하는 놈이 장땡인 것 같다.
 - `ViewController`는 항상 `UINavigation`을 옵셔널 형태로 갖고 있다.
 - `navigationBar.topItem` (항상 불리는 순서에 따라 타이틀이 정해짐) > `navigationController.title` > `self.title` (본인 뷰의 네비게이션을 본인 뷰에서 호출하는 경우 가능

--- 

- `self.title` : 이게 가장 원하는 대로 쓰기 좋을 것 같다. 
	- `UINavigationController`의 `title`도 바뀌고
	- 해당하는 VC title도 바뀌고
	- `UINavigationController.navigationbar`도 바뀌고
- `self.navigationController?.title`
	- 무쓸모
	- 이 코드만 있을 경우 타이틀 텍스트도 안나타남
	- 계층 구조 상 `UINavigationController` 자체의 title만 바뀜
- `self.navigationController?.navigationBar.topItem?.text`
	- 정말 말 그대로 `navigationBar`에 위치하는 중앙 텍스트 값을 바꾸는 역할 
	- 다른 코드도 있는 경우 가장 `우선적`으로 뜸 
 
 
네비게이션 타이틀은 기본적으로 VC의 타이틀을 따라간다
-> 왜냐? navigation 문서에 따르면 기본적으로 VC의 타이틀을 default 값으로 갖는다고 한다!
 
 네비게이션 관련 문서들을 더 읽어봐야할 것 같다. 
 
---

**Ref**

[야곰닷넷 Unit Test](https://yagom.net/courses/unit-test-%EC%9E%91%EC%84%B1%ED%95%98%EA%B8%B0/lessons/unit-test%EA%B0%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80%EC%9A%94/)

