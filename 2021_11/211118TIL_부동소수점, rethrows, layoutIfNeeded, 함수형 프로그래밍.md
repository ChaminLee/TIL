# 부동소수점, rethrows, layoutIfNeeded, 함수형 프로그래밍

## 211118_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 함수형 프로그래밍

함수형 프로그래밍이란 거의 모든 것을 `순수 함수`로 나누어 문제를 해결하는 기법이라고 한다. 

이러한 함수형 프로그래밍의 특징은 아래와 같다. 

- 같은 입력을 주는 경우 항상 같은 출력값을 반환해야한다. 
	- 수학에서의 함수와 같다. 
- 주변 상태나 문맥에 따라 출력값이 변하지 않고 항상 같아야 한다. 

순수 함수란 그렇다면 무엇일까? 

- 언제 같은 입력을 주고 실행해도 항상 같은 결과가 나오는 함수 
- 외부의 상태값을 변경시키지 않는 함수

```swift
func introduce(name: String) {
	print("내 이름은 \(name)야")
}
```

`name`에 `chacha`가 들어가는 경우 이 함수는 항상 "내 이름은 chacha야"를 출력하게 된다. 

즉 외부 상태 값에 영향을 받지도, 변경을 시키지도 않으며, 같은 입력에 대해서는 항상 같은 출력값을 보장한다!

> 동시성 문제
> - 다중 프로세서 환경에서 발생
> - A,B가 동시에 프로세서에 접근하여 문제가 발생하는 경우
> - 대기열을 만들게 되면 연산에 시간이 오래 걸리고, 비용이 많이 든다. 

하지만 객체 지향 프로그래밍에서도 함수형의 개념을 일부 들여올 수 있는데, 이 때 상태를 아예 만들지 않을 수는 없다. 

이에 상태값을 나타내는 프로퍼티를 불변하게 만들어주는 것이 조금이나마 도움이 된다고 한다. 

상태값이 가변적이라는 것은 곧 함수가 외부 상태에 따라 값이 변경될 가능성이 있다는 것이기에 `let`으로 선언하여 이러한 가능성을 없애라는 말 같다!

### `rethrows`란? 

자신의 매개변수로 전달받은 함수가 오류를 던질 수 있다는 것을 나타냄 

고차함수, 즉 함수 안에 클로저를 파라미터로 받을 때 그 클로저가 에러를 던질 수 있다면 함수에는 rethrows를 써줘야 한다! 

```swift
func calculate(num1: Int, num2: Int, arithmetic: (Int,Int) throws -> Int) rethrows -> Int {
    return try arithmetic(num1, num2)
}
```

### `LayoutIfNeeded `vs `SetNeedsLayout`

- `SetNeedsLayout`
	- `layoutsubviews`를 예약하는 행위, 다음 update cycle에서 반영됨
	- 대신 update cycle에서 반영될 때 애니메이션은 일어나지 않는다
	- 비동기적으로 호출되고 바로 반환된다
- `LayoutIfNeeded`
	- layoutsubview를 예약하는 것은 setneedsLayout과 같다. 
	- 바로 예약한 `layoutsubviews`를 실행시킨다. (바로 실행되기에 애니메이션도 반영된다!)
	- 동기적으로 바로 발동한다

### `Decimal`이란??

우리는 초등학교 때 배웠다.. 10을 3으로 나누면 딱 나누어 떨어지지 않는다는 것을.. 무한한 `3.3333333`을 마주할 수 있었을 것 이다.  

이를 프로그래밍을 통해 구현해보면 안타깝게도 손으로 직접 계산한 것 보다 못한 값이 나온다.

```swift
print(10.0 / 3.0)
// 3.3333335
```

문제가 무얼까 하니 `부동소수점` 때문이었다. 부동소수점은 값을 정확하게 나타내는 것이 아니라 `근사치`를 나타내게 되는데 이 때문에 연산을 할 때에 작은 차이가 계속 발생하게 되는 것이다. 

나눗셈이야 그렇다 쳐도... `1.04 - 0.42`를 해도 `0.62`가 나오지 않고 `0.61000001`이라는 값이 나오게 된다. 

이를 해결하기 위한 타입이 바로 `Decimal`! 

```swift
print(Decimal(10.0) /  Decimal(3.0))
// 3.333333
```

`Double` 타입으로 계산하는 것이 아니라 `Decimal` 타입으로 바꿔서 계산을 해주면 부동소수점 문제를 해결할 수 있다. 

마찬가지로 빼기 연산도 가능해진다

```swift
print(Decimal(1.04) -  Decimal(0.42))
// 0.62
```

자 이제 이렇게 `Decimal`로 연산을 했으니 값을 어딘가 써야할 순간들이 오게 될 것이다. 이때 보통 다시 `Double` 타입으로 사용하고 싶을 텐데 이 때는 `NSDecimalNumber`를 사용해야한다. 물론 다른 타입으로의 변환도 지원한다. 

```swift
let result = Decimal(1.04) - Decimal(0.42)

NSDecimalNumber(decimal: result).stringValue // "0.62"
NSDecimalNumber(decimal: result).doubleValue // 0.62
NSDecimalNumber(decimal: result).intValue // 0

let result2 = Decimal(10.0) / Decimal(3.0)

NSDecimalNumber(decimal: result2).stringValue // "3.3333333333333333333333333333333333333"
NSDecimalNumber(decimal: result2).doubleValue // 3.333333333333333
NSDecimalNumber(decimal: result2).intValue // 0
```
이렇게 `Decimal`로 계산한 결과를 원하는 타입으로 변환할 수 있다. 

위 타입 말고도 다양한 타입을 지원하고 있다!
```swift
open var int8Value: CChar { get }

open var uint8Value: UInt8 { get }

open var int16Value: Int16 { get }

open var uint16Value: UInt16 { get }

open var int32Value: Int32 { get }

open var uint32Value: UInt32 { get }


open var int64Value: Int64 { get }

open var uint64Value: UInt64 { get }

open var floatValue: Float { get }

open var doubleValue: Double { get }

open var boolValue: Bool { get }

@available(iOS 2.0, *)
open var intValue: Int { get }

@available(iOS 2.0, *)
open var uintValue: UInt { get }


open var stringValue: String { get }
```

알맞게 골라쓰면 될 듯 하다. 

> 번외... 아직 미해결
>  `Decimal(100.0) /  Decimal(3.0)` 하는 경우 33.33333이 나와야 한다. 물론 Decimal 연산에서는 잘 나오나 `.doubleValue`를 하면 33.3333334가 나온다... 
>  근데 또 `Decimal(10.0) /  Decimal(3.0)`하고 `.doubleValue`하면 3.33333이 잘 나온다... 
>  이건 아직 해결 못함..ㅠ

## 고민된 점 
- `Double`의 연산에서 발생하는 부동소수점 문제
- 즉각적인 뷰 반영을 위한 방법

## 해결 방법 
- `Decimal` 사용
- `layoutIfNeeded` 사용
 ---
 

**Ref**
[layoutifneeded](https://stackoverflow.com/questions/1182945/how-is-layoutifneeded-used)
[layoutIfNeeded and setNeedsLayout](https://baked-corn.tistory.com/105)
[부동소수점 wiki](https://ko.wikipedia.org/wiki/%EB%B6%80%EB%8F%99%EC%86%8C%EC%88%98%EC%A0%90)
[부동소수점](https://github.com/TheSwiftists/effective-swift/blob/main/9%EC%9E%A5_%EC%9D%BC%EB%B0%98%EC%A0%81%EC%9D%B8_%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D_%EC%9B%90%EC%B9%99/item60.md)


