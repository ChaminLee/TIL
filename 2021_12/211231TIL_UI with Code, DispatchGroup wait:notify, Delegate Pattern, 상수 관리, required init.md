# UI with Code, DispatchGroup wait/notify, Delegate Pattern, 상수 관리, required init

## 211231_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

> 찰리와의 리뷰를 통해 얻은 지식을 정리해보자!

### 1. UI 인스턴스 생성

UI 인스턴스를 만들어주기 위한 방법 3가지를 봐보자. 

> 1 . Custom View
```swift 
class CustomerLabel: UILabel {
    override init(frame: CGRect) {
        super.init(frame: frame)
    }
    
    init(order: Int, type: Task) {
        super.init(frame: CGRect.zero)
        // init
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }
}
```

기본적으로 초기화 할 때 프로퍼티의 값을 넣어주어 인스턴스를 생성해주게 된다. 

> 2 . 일반적인 생성 방법 
```swift
let customerLabel = UILabel() 

func setupLabel() {
    customerLabel.text = "123"
    customerLabel.textColor = .black
    // ~~ 
}
```

인스턴스를 선언해두고 이에 대한 프로퍼티는 메서드를 통해 설정해주는 방식이다. 

UI 인스턴스와 해당하는 프로퍼티를 설정하는 코드가 떨어져 있어서 코드를 읽기가 어려운 점이 있다. 이에 코드를 모아서 보는 1,3번이 더 좋은 방식으로 보인다.

> 3 . 클로저 방식
```swift
let customerLabel: UILabel = {
    let label = UILabel()
    label.text = "123"
    // ~~ 
    return label
}()
```

이 방식의 경우 클로저를 만들어 바로 `( )`를 통해 실행하여 인스턴스를 생성하는 방식이다. 

이 경우 UI의 프로퍼티들을 한 군데에 모아서 관리할 수 있다는 장점이 있다. 또한 마찬가지로 선언과 동시에 프로퍼티를 지정할 수 있다. 하지만 이 경우 아직 `self`가 생기기 이전이기 때문에 클로저 블록 내부에서 `self`를 사용할 수 없다. 

그래서 이 방법을 추천받았다! 보기에도 custom view를 만드는 것 보다 공수가 덜 들기도 하면서 장점을 취할 수 있는 방법인 것 같다. 

### 2. DispatchGroup wait/notify

앞서서 wait은 동기적으로, notify는 비동기적으로 작동한다고 학습했었다. 

하지만 `wait()`의 경우는 메인 쓰레드를 블록할 수 있는 위험이 있다. `Thread.sleep()`과 마찬가지로 쓰레드를 정지시킨다는 것은 자원을 낭비하는 것과 같다고 한다. 

이제 쓰레드를 멈추는 일은 최대한 없어야 하고, 비동기적으로 해결해야한다. 

예를 들어서 A작업을 하고 3초 뒤에 B작업을 해야한다면 3초간 쓰레드를 정지시키는 것이 아니라 Timer를 이용하거나 다른 방법을 사용해서 3초 있다가 비동기적으로 B작업이 실행되게 해야한다. 

그래서 `wait()` 보다는 보통 `notify()`를 사용해서 비동기적으로 어떤 작업이 완료되었을 때 원하는 기능을 수행할 수 있다!

> 특수한 목적이 있다면 메인 쓰레드가 아닌 다른 쓰레드를 블록하는 것은 가능할 수 있을 것 같기도 하다.. 예를 들어서 `DispatchQueue.global().async { }` 코드 블록 내에서 `.wait()`를 쓰는 경우이다!

> 흐름 제어가 복잡해지는 경우 가독성이 떨어지게 된다. DispatchQueue를 여러 개 사용하면 그렇긴 하다..! 그래서 다른 해결 방법이 있다면 가독성을 고려해서 결정하는 것이 좋을 것 같다.

> 보통은 global 큐를 사용하지 않고, 커스텀하게 DispatchQueue를 만들어서 사용한다고 한다. 왜냐하면 내가 만들고 내가 관리하기에 나만 쓸 수 있어서 안전하다고 판단해 볼 수 있다고 한다.

stackview는 느린 컴포넌트, 하지만 이해하기 좋고 관리하기 좋음 

> flex layout 찾아보기 

### 3. Delegate Pattern 

객체 지향에 대해 고민해보다 보면 객체들 사이의 관계에 대해서 고민해보게 된다. 이 때 객체 간 소통을 돕기 위해 delegate pattern을 사용하기도 하는데 어떤 상황에서 적합하고, 부적합할까? 

- `부모`가 `자식`에게 이야기 해야 할 때
	- 부모가 자식과 대화하는데 굳이 delegate pattern은 필요하지 않다.
	- 부모가 자식을 알도록(의존) 해도 되는 관계이다. 
- `자식`이 `부모`에게 이야기 해야 할 때
	- 이 때 delegate pattern이 적합한 방법 중 하나이다.
	- 자식이 부모를 알게 되면 문제가 발생할 수 있다고 한다. 
	- 이에 의존성을 낮추기 위해 protocol을 사용하는 것이다 
	- 의존성이란 곧 서로의 변경에 서로 영향을 받는다는 것이다. 
	- 이에 변경을 최소화하기 위해 delegate pattern을 사용하는 것이다.
- `자식A`가 `자식B`에게 이야기 할 때
	- 동등한 레벨이라고 하면 이 경우도 굳이 delegate pattern이 사용되지 않아도 된다. 

정리해보면 의존하기 어려운 상황에 delegate pattern을 사용하여 추상 개념에 의존하도록 하여 변경에 취약하지 않도록 구현해주는 것 같다!

### 4. 상수 관리

코드를 짜다보면 특정한 문자열, 숫자값 등 상수를 관리할 필요가 있다. 이에 그냥 직접적으로 적어주는 것 보다 한 곳에서 관리하는 것이 좋다. 

이전에도 학습해봤지만 name space를 사용하는 것이 좋다. 

보통 파일 내에 private enum으로 만들어서 관리한다.

```swift
private enum Design {
	static let leadingMargin: CGFloat = 10
	// ~
}
```

상수를 한 군데에서 관리하면 값의 수정(관리)도 편하고, 해당 상수가 들어가는 코드를 읽기도 좋아진다. 

### 5. `removeFromSuperview`보다는 `isHidden`

`removeFromSuperview`의 경우 조심히 사용해야 한다고 한다. 뷰를 제거하는 것이기 때문에 auto layout 계산이 무조건적으로 따라오게 되고 제거하는 것 자체가 불필요하다고 판단될 수 있다고 한다. 

이에 요소를 숨겨주는 `isHidden`을 많이 사용한다고 한다. 요소를 삭제하는 것이 아니라 숨기기만 하는 것 이기에 좋다고 한다!

### 6. required init

required init은 Interface builder에서 기본적으로 호출된다. 

그래서 사실 코드단에서는 직접적으로 써줄 필요가 없다. 이에 fatalError를 쓰거나, interface builder용이라고 적어두기도 한다고 한다. 

만약 interface builder로 구현한다면 기본 구현 내용을 같이 써줄 수 있다! (기본 init에 들어가는 텍스트 ,색, 정렬  프로퍼티에 대한 부분)

### 7. 깨알 팁

- for문은 없다고 생각해라. 
	- 고차 함수 사용하자!!!
- while문 보다 고차함수를 사용하자 (같은 맥락!)
- 분기가 있는 건 나쁜 코드라고 봐도 된다..?
	- 분기가 없도록 하는 로직이 가독성이 높고 좋은 코드다
	- 닐 병합 연산자 사용시 Optional을 extension해도 좋다. 

```swift
extension Optional where Element == String {
    var orEmpty: String {
        return self ?? ""
    }
}

let str: String?
str.orEmpty
```

이렇게 구현하면 `String?`인 옵셔널에 대해서만 확장하는 것이다. 문장을 읽는 것 처럼 `orEmpty`라고 연산 프로퍼티를 만들어서 사용하면 더 가독성이 높아지고 해결하기 좋아진다!

## 고민된 점 
- DispatchQueue의 적합한 활용 방법에 대하여
- 상수 관리 방법에 대하여 
- wait/notify등 비동기 프로그래밍에서 사용해야하는 방법은?

## 해결 방법 
- 문서를 읽고 리뷰어와의 대화를 통한 해결!!

---

**Ref**

찰리와의 Step 4 리뷰





