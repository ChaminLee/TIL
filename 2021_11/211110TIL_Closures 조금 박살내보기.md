# Closures 조금 박살내보기

## 211110_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Closures

Closures란 여기저기서 사용될 수 있는 기능을 갖고 있는 코드 블럭이라고 이해할 수 있을 것 같다. 파이썬의 람다와 같은 느낌..!

Closures는 상수나 변수가 선언된 위치에서 참조를 획득하고 저장할 수 있다. 획득(capturing)하는 것은 아래에서 다뤄볼 예정!

클로저가 익숙한 이유는, 함수 또한 이름을 가지는 클로저이기 때문이다. 이러한 클로저는 3가지 형태를 가진다. 

- 어떠한 값도 획득하지 않고 이름을 가진 클로저인 전역 함수 형태
- 중첩된 함수에서 값을 획득할 수 있고 이름을 가진 클로저인 중첩 함수 형태
- 축약 문법으로 주변 맥락에 따라 값을 획득할 수 있고 이름은 없는 형태 

이러한 클로저 표현은 최적화를 통해 명확하고, 깔끔하게 사용할 수 있다. 최적화 요소들은 다음과 같다.

- 맥락으로부터 파라미터와 리턴 값 타입을 추론
- 한 줄로 된 클로저는 암시적으로 반환값 취급
- 축약된 인자 이름 
- 후행 클로저 문법


클로저의 축약 문법은 문서를 보면 잘 이해가 되니 다른 부분을 정리하고 학습해보자. 

#### Escaping Closure

Escaping closure란 말 그대로 탈출 클로저이다. 무엇을 탈출하는것인가 하니, 주로 함수의 파라미터에 클로저를 사용할 때 함수를 탈출하는 것을 의미한다고 한다.

 Escaping 클로저임을 나타내주기 위해 클로저 앞에 `@escaping`을 붙여준다. 이를 붙여줌으로써 이제 해당 클로저는 함수가 종료된 뒤에 실행이 된다. 

함수의 파라미터에 있던 클로저가 함수가 종료된 뒤에 실행이 된다고...? 사실 조금 더 자세하게 이야기하면 종료된 뒤에 실행이라기 보다는 해당 함수를 벗어난 밖에서 실행이 된다고 한다. 

그렇다면 non-escaping과 escaping의 차이를 우선 알아보자. 

> 1. Non-Escaping

non-escaping 클로저의 경우, 우리가 평소에 쓰던 방식과 같다. 

```swift 
func test(handler: () -> String) {
    print("이제 \(handler())하자")
}

func play() -> String {
    return "게임"
}

test(handler: { play() } )
```

클로저가 실행되는 순서는 다음과 같다. 

1. `test` 함수의 `handler` 인자로 클로저가 전달된다. 
2. `test` 함수 내에서 `handler()` 클로저가 실행된다. 
3. `test` 함수가 값을 반환하고 종료된다. 

즉 함수가 시작하고 종료되는 그 사이클 내부에서 클로저 또한 실행이 되는 것이다. 

반면에 escaping 클로저는 어떨까? 

> 2. Escaping

우선 escaping 클로저란, 앞서 말했듯이 함수의 실행/종료 사이에서 실행되는 클로저가 아니라 종료 이후 밖에서 실행 가능한 클로저이다. 

```swift
var completionHandler: (() -> String)?

func test(handler: @escaping () -> String) {
    completionHandler = handler
}

func play() -> String {
    return "게임"
}

test(handler: { play() } )

// 해당 클로저를 부르기 전까지 handler 클로저는 실행 안됨
completionHandler!()
```

똑같이 클로저가 실행되는 순서를 보자. 

1. `test` 함수의 `handler` 인자로 클로저가 전달된다. 
2. `test` 함수 내 `handler()` 클로저가 `completionHandler`에 저장된다. 
3. `test` 함수가 값을 반환하고 종료된다. 
4. `handler` 클로저는 아직 실행되지 않았다. 
5. 별도로 `completionHandler`를 외부에서 실행시켜야 해당 클로저가 실행된다. 

보통 비동기 작업을 할 때 많이 사용된다고 한다. 함수의 호출이 모두 끝난 이후 필요에 따라 호출할 수 있다는 장점이 있는 것 같다. 

하지만 이 두 가지 방법들에도 각각 장단점이 있다. 

- Non-Escaping Closure
	- 컴파일러는 클로저가 언제 실행될지 알고 있다
	- 객체의 life cycle을 보다 더 효율적으로 관리할 수 있다. 
- Escaping Closure
	- 클로저가 함수 밖에서도 사용될 수 있도록 보장하기 위해 reference cycle을 추가적으로 관리해줘야 한다. 

이에 따라 필요시에만 `@escaping` 수식어를 붙여 사용한다고 합니다!

#### Autoclosures

직역하면 자동 클로저이다. 무엇을 자동으로 해준다는 걸까? 
함수의 파라미터로 전달되는 코드를 감싸서 자동으로 클로저로 만들어 준다고 한다. 즉 클로저가 아닌 형태로 적어도 클로저처럼 사용할 수 있다는 것이다. 

우선 기본적인 클로저를 다시 살펴보자
```swift
func test(handler: () -> String) {
    print("이제 \(handler())하자")
}

func play() -> String {
    return "게임"
}

test(handler: { play() } )
// 이제 게임하자

// test 함수는 파라미터로 () -> String을 받는다
// test(handler: () -> String)
```

handler에 클로저를 써주기 위해서 `() -> String` 타입인 함수를 넣어줬다. 하지만 `{ }`로 감싸서 전달해주고 있는 것을 볼 수 있다. 여기서 autoclosure를 사용하면 어떻게 될까?

`@autoclosure`를 사용하면 조금 더 편리하게 클로저를 이용 할 수 있다.

```swift
func test(handler: @autoclosure () -> String) {
    print("출력 내용: \(handler())")
}

func play() -> String {
    return "게임"
}

// 문자열처럼 쓸 수 있음
test(handler: play())
// 출력 내용: 게임 
test(handler: "반환 타입의 값을 넣을 수 있어")
// 출력 내용: 반환 타입의 값을 넣을 수 있어
``` 

사용하고 싶은 클로저 앞에 `@autoclosure`를 붙여주기만 하면 된다. 

실제 클로저를 전달해주는 부분을 보면 `{ }`없이도 전달이 가능한 것을 볼 수 있다. 

또한 `@autoclosure`를 쓰면 반환 타입의 값을 클로저 대신에 넘겨줄 수 있는 것이다. 

```swift
// @autoclosure 사용시 매개변수 타입이 String이 된다
// test2(handler: String)
```

하지만 `@autoclosure`는 매개변수가 없는 클로저에 대해서만 사용될 수 있다. 즉 아래와 같이 매개변수가 있는 경우에는 사용할 수 없다. 

```swift
// Argument type of @autoclosure parameter must be '()'
func test(handler: @autoclosure (Int) -> String) {
    print("이제 \(handler())하자")
}
```


> **Tip. 클로저를 묶어서 표현하기**
>  ```swift
>  let multipy: ((Int) -> Void)?
>  ```
>  이렇게 되어있는 것은 함수 타입 자체를 옵셔널로 만들어 주기 위해 묶어주는 것. 묶는다는 것 이외의 의미가 없다고 봐도 될듯하다. 

> **Tip. 클로저 배열로 담기**
> ```swift
> let functions: [(Int, Int) -> Int] = [add, subtract]
> ```
> 해당 클로저 타입인 것들을 배열로 묶어둘 수 있다. 

> **Tip. 하나의 함수에서 2개 이상의 클로저를 파라미터로 받을 때**
> 파라미터 순서에 따라 호출의 순서가 정해지는게 아니고, 클로저는 그저 정의 되는 것이고 호출은 나중에 사용된다. 
> 케이스에 따라 원하는 클로저를 실행하는 등 다양하게 사용할 수 있다. 

클로저는 다음과 같은 순서를 거친다고 봐도 된다. 

```swift
// 1. 함수 타입 작성
func test(handler: () -> String) {
	// 2. 클로저 호출
    handler()
}

// 3. 사용할 클로저 정의  
let completionHandler = { print("클로저 정의") }
```

### 일급객체란?

일급함수의 특성과 유사한 것 같다. 거의 같다고 봐도 될 듯..?

- 변수나 상수에 저장 및 할당될 수 있어야 한다. 
- 파라미터로 전달 할 수 있어야 한다. 
- 함수에서 return 할 수 있어야 한다. 

## 고민된 점 

- 테스트 코드의 이름을 어떻게 더 명확하게 할 수 있는가? 
- Unit Test시 setUp과 tearDown의 활용
 
## 해결 방법 

- 처음에는 테스트 코드의 메서드 명을 영어로 작성했었다. 하지만 TDD의 장점 중에 스펙을 문서화 할 수 있다는 부분에서 한글로 메서드 명을 바꿔도 좋을 것 같다는 생각이 들었다. 그리고 한국인이니 영어보다는 한글이 더 직관적으로 테스트 코드를 작성하고 이해할 수 있을 거라 판단하고 한글로 수정해봤다. 
- `setUp`과 `tearDown`메서드를 활용하여 테스트 코드의 메서드에서 반복되는 부분들을 사전/사후에 한 번에 처리해주는게 좋을 것 같다고 생각했다. 하지만 다른 케이스를 주고싶다면 이렇게 사용하는 건 부적절할 것 같다는 생각이 들었다. 예를 들어 제네릭으로 어떤 타입을 정의했다고 해보자. 어떤 경우에는 String으로 테스트 해보고 싶고, 다른 경우는 Int 타입으로 테스트 해보고 싶을 경우 `setUp`에서 이 요구사항들을 모두 들어주기 힘들 것 같다. 그래서 테스트 케이스가 한정적인 경우, 공통적인 부분이 많은 경우에는 `setUp`과 `tearDown`을 적절히 사용하여 공수를 줄이는 것도 좋은 방법일 듯 하다!

---

**Ref**

[Closures](https://docs.swift.org/swift-book/LanguageGuide/Closures.html)
[Escaping closure](https://jusung.github.io/Escaping-Closure/)
