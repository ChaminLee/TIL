
# @propertyWrapper, Atomic Type 생성(DispatchQueue, NSLock)

## 211229_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. `@propertyWrapper`란?

단어 의미 그대로 속성을 덮는다...?라는 뜻을 가지는 것 같다. 차차보면 알겠지만.. "프로퍼티에 특정 기능을 부여해준다" 라고 이해해봐도 좋을 것 같다. 

예를 들어 아래와 같이 각 과목(타입)의 점수와 평균을 반올림 해줘야 하는 경우가 있다고 해보자. 

```swift
struct Math {
    private var _score: Double = 0
    
    var score: Double {
        get {
            return self._score.rounded()
        } set {
            self._score = newValue
        }
    }
    
    init(score: Double) {
        self.score = score
    }
}

struct English {
    private var _average: Double = 0
    
    var average: Double {
        get {
            return self._average.rounded()
        } set {
            self._average = newValue
        }
    }
    
    init(average: Double) {
        self.average = average
    }
}


let math = Math(score: 85.88)
let english = English(average: 75.10)
print(math.score) // 86.0
print(english.average) // 75.0
```

이렇게 각 과목별로 타입을 만들어 내부에 연산 프로퍼티를 두어 기능을 수행하도록 만들어 줄 수 있다. 

여기서 중복되는 코드, 즉 보일러플레이트 코드를 발견할 수 있을 것이다. 바로 각 타입의 연산 프로퍼티에서 `.rounded()`를 중복적으로 사용하고 있다는 점이다. 물론 지금은 타입의 수가 적기에 크게 문제가 없겠지만 여러 타입에 해당 기능을 지원해야하는 경우에 `@propertyWrapper`를 써볼 수 있다. 

클래스, 구조체와 열거형에`@propertyWrapper`를 써주면 property wrapper가 된다. 이 속성(attribute)를 타입에 적용할 때는 `@propertyWrapper` 아래에 정의한 타입과 동일한 이름을 사용하여 속성을 줄 수 있다. 

> 연산 프로퍼티, 전역 변수, 상수에는 property wrapper를 사용할 수 없다. 

```swift
@propertyWrapper 
struct Rounded { }
// ...

@Rounded var name: String
```

대략 코드만 적어보았다. 앞서 말한 것 처럼 property wrapper로 지정된 타입의 이름을 속성으로 사용해서 나타내주면 된다는 것이다. 

이제 이 속성이 붙은 프로퍼티는 특수한 기능을 사용할 수 있게 되는 것이다. 

그렇다면 이제 propertyWrapper 역할을 해줄 타입을 만들어주자. 이 타입은 특정한 행동을 정의하는 타입으로 다른 프로퍼티에 적용시키고 싶은 것들을 사전에 구현해둔다고 생각해두면 이해가 쉬울 것 같다. 

```swift
@propertyWrapper
struct Rounded {
    var value: Double
    
    var wrappedValue: Double {
        get {
            return value.rounded()
        } set {
            self.value = newValue
        }
    }
    
    init(initialValue: Double) {
        self.value = initialValue
    }
}
```

- `wrappedValue` 필수 구현
- `init()` 필수 구현

> 1 . `wrappedValue` 필수 구현

위 처럼 구조체를 정의하게 되는 순간 `wrappedValue`가 없다면 에러가 날 것이다. 그러니 이를 정의해주자. 보통은 연산 프로퍼티로 많이 사용하지만, 저장 프로퍼티로도 사용될 수 있다고 한다. 

이 부분에서 로직이 들어가게 된다. 값을 읽어올 때 반올림 해주는 로직을 추가해준 것을 확인할 수 있다.

이제는 사용법을 보자. 

```swift
struct Math {
    @Rounded var score: Double
}
```

사용 방법은 간단하다. 변수 앞에 `@타입명`을 붙여주면 된다! 

이제 인스턴스를 만들어서 score를 출력해보면 반올림 처리된 상태로 출력되는 것을 확인해 볼 수 있다. 

> 2 . `init()` 필수 구현

그렇다면 `init()`은 왜 있어야 할까? 

만약 `Rounded`에 `init()`이 없다면 어떻게 될까?
```swift
let math = Math(score: <#T##Rounded#>)
```

위 처럼 Math 타입이 제공하는 멤버와이즈 이니셜라이저를 보면 Double 타입이 아니라 property wrapper 타입인 Rounded를 가리키고 있는 것을 볼 수 있다. 

이를 방지하는 방법은 2가지가 있을 것 같다. 

#### 1. Math 타입에 이니셜라이저 생성

```swift
struct Math {
    @Rounded var score: Double
    
    init(score: Double) {
        self.score = score
    }
}
```
기본 제공해주는 이니셜라이저가 아니라 직접 만들어서 사용하면 예상하는대로 Double 타입을 요구하는 이니셜라이저가 등장한다!

#### 2. Rounded 타입에 이니셜라이저 생성 

```swift
@propertyWrapper
struct Rounded {
    var value: Double
    
    var wrappedValue: Double {
        get {
            return value.rounded()
        } set {
            self.value = newValue
        }
    }
    
    init(initialValue: Double) {
        self.value = initialValue
    }
}
```
아까처럼 Rounded 타입에 이니셜라이저를 두면 이 property wrapper를 채택(?)하는 프로퍼티들은 정상적으로 본인 타입을 요구하는 이니셜라이저를 사용할 수 있게 된다. 

이렇게 property wrapper를 사용하면 불필요한 코드를 제거할 수 있을 뿐만 아니라 타입 자체를 thread-safe하게 만들어 줄 수 있다. 

Generics를 함께 사용해서 property wrapper를 만드는데, 내부 동작을 serial queue에서 동기적으로 처리해주거나, `NSLock`을 활용하여 lock/unlock 해주는 방식이다. 

### 2. `@propertyWrapper`를 활용한 Atomic(thread-safe) 타입 생성

> 1 . DispatchQueue 사용
```swift
@propertyWrapper
struct Atomic<Value> {
    private let queue = DispatchQueue(label: "atomic")
    private var value: Value

    init(wrappedValue: Value) {
        self.value = wrappedValue
    }

    var wrappedValue: Value {
        get {
            return queue.sync { value }
        }
        set {
            queue.sync { value = newValue }
        }
    }
}

struct AtomicArray {
    @Atomic var array: [Int]
}
```

serial queue를 만들어 해당 타입에 get/set으로 호출했을 때, 접근 가능한 쓰레드에 제한을 둘 수 있다. 

> 2. NSLock 사용

```swift
@propertyWrapper
struct Atomic<Value> {

    private var value: Value
    private let lock = NSLock()

    init(wrappedValue value: Value) {
        self.value = value
    }

    var wrappedValue: Value {
      get { return load() }
      set { store(newValue: newValue) }
    }

    func load() -> Value {
        lock.lock()
        defer { lock.unlock() }
        return value
    }

    mutating func store(newValue: Value) {
        lock.lock()
        defer { lock.unlock() }
        value = newValue
    }
}

struct AtomicArray {
    @Atomic var array: [Int]
}
```

1번과 같은 논리이다. 하지만 DispatchQueue가 아니라 NSLock을 통한 방법이라는 것이 그 차이이다. 

쓰레드가 접근하면 `lock()`해버려서 다른 쓰레드가 접근할 수 없게 되는 것이다. 

적용 방법 또한 동일하다! 

## 고민된 점 
- `@propertyWrapper`란?
- 타입을 Atomic(Thread-safe)하게 만드는 방법

## 해결 방법 
- 공식문서 읽기!
- propertyWrapper + DispatchQueue/NSLock
---

**Ref**

https://docs.swift.org/swift-book/ReferenceManual/Attributes.htmlhttps://www.onswiftwings.com/posts/atomic-property-wrapper/

https://tomzurkan.medium.com/creating-an-atomic-property-in-swift-988fa55cc71

https://zeddios.tistory.com/1221
