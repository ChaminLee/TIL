# Semaphore, Closure Capture

## 211228_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. Semaphore에 대해 

이전에 semaphore에 대해 맛만 본 적이 있다. ([Semaphore](https://leechamin.tistory.com/546?category=1012929)) 

이해하기로는 **"멀티 프로그래밍 환경에서 공유 자원에 대한 접근을 제한하는"** 역할을 주로 한다고 생각했다. 이에 주로 semaphore의 value는 번갈아가며 1 혹은 0인 상태를 유지할 것이라고 생각했다. 

하지만 0 이상의 정수 즉 2개도 가능은 할 것 같다고 생각했다. 말 그대로 공유 자원에 접근을 제한하기 위해 사용하지만, 만약 공유 자원에 접근하는 부분을 쓰레드에서 사용하지 않는다면??

```swift
DispatchQueue.global().async { 
    print("123")
}
```

예를 들어 위의 코드 블록에서 어떠한 작업을 수행하는데, 출력만 한다고 생각해보자. 출력 같은 경우는 일반적으로 어떠한 공유자원에 접근한다기 보다는 어떠한 값을 출력하는데 주 목적을 두고 있다. 
(다만 공유 자원을 출력하는 경우에는 위에 말하는 경우와 다를 것 같다)



이에 여러 쓰레드에서 호출되어도 문제가 없을거라 판단했다. 그래서 이러한 경우에는 semaphore의 수가 실행가능한 쓰레드의 수가 되는 것 같다. 

```swift
let semaphore = DispatchSemaphore(value: 2)

for i in 0..<4 {
    DispatchQueue.global().async {
        semaphore.wait()
        print("\(i) 시작")
        Thread.sleep(forTimeInterval: 2)
        print("\(i) 끝")
        semaphore.signal()
    }
}
```

위와 같은 코드를 실행하게 되면 두 개씩(`0 시작`, `1 시작`) 묶음처럼 출력된다. 만약 semaphore를 3으로 한다면?? 현재와 같이 concurrent하고 비동기적인 상황에서는 3개의 출력이 나오게 될 것이다. 

이렇게 공유자원과 무관하게 된다면 간단하게 생각해봐도 될 것 같다. 

하지만 현실을 녹록치 않으니,,, 항상 공유 자원의 접근으로 인해 발생할 수 있는 문제들을 조심해야 할 것 같다.

### 2. Closure의 `self`

Closure를 사용하다보면 내부 코드 블럭에 `self.`를 필수적으로 붙여 사용해본 경험이 있을 것이다. 

```swift
// 에러 발생 코드
class Dog {
    var name: String = "댕댕이"
    
    lazy var closure: () -> () = {
        print("강아지 이름은 \(name)입니다.") // Reference to property 'name' in closure requires explicit use of 'self' to make capture semantics explicit
    }
}

```

![](https://i.imgur.com/Ncn6tAA.png)

캡처 의미를 명확하게 만들기 위해 `self`를 명시적으로 사용하라고 말하고 있다. 그렇다면 먼저 Closure Capture에 대해서 알아볼 필요가 있을 것 같다. 

> 1. 클로저 캡처란?

우선 클로저는 context 내에 있는 상수/변수에 대한 참조를 캡처하여 저장할 수 있다. 보통은 함수가 종료되면 내부의 프로퍼티들은 메모리 해제가 되는데, 클로저의 캡처를 하면 원본이 사라져도 클로저 body 부분에서는 이를 수정하거나 참조할 수 있게 된다. 

swift 공식 문서의 예제를 보자 


```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}

let closure = makeIncrementer(forIncrement: 10)
closure() // 10
closure() // 20
closure() // 30
```

`closure()` 라고 상수에 저장한 클로저를 계속 실행하면 내부의 `runningTotal`의 값이 초기화되지 않고 계속 상승하는 것을 볼 수 있다. 이를 보면 클로저 캡처가 body 내의 변수/상수들을 저장하고 있음을 알 수 있다. 

이제 다시 돌아와서 에러 문구를보면, `name` 프로퍼티에 대한 참조를 클로저 내부에서 가지기 위해서는 명시적으로 `self`를 써줘야 한다는 것이다. 

이에 대한 이유를 찾아보니 아래와 같은 이야기들이 주를 이룬다. 

- 사용자가 변수/상수/함수가 캡처되고 있음을 인지하고 있는지 확인하고자
- 블록 내에서 프로퍼티를 사용하려면 전체 객체를 참조해야하기 때문
- 의미론적으로 self를 참조한다고 나타내줘야 한다. 
- 자동적으로 캡처 리스트에 self가 추가되는데, 따라서 closure에서는 인스턴스에 직접 접근하지 못하고, 자동으로 추가된 self 변수를 통해 접근해야 한다. 

정리해보자면 캡처하기 위해 참조가 필요한데 이때 참조를 분명하게 나타내주기 위함인 것 같다. 

이에 앞선 오류가 나는 코드를 다음과 같이 바꿔볼 수 있을 것이다. 

```swift
class Dog {
    var name: String = "댕댕이"
    
    lazy var closure: () -> () = {
        print("강아지 이름은 \(self.name)입니다.")
    }
}
```

앞서 말했던 것 처럼 캡처라는 것은 참조한다는 것인데, 만약 참조하는 것이 reference type이라면 순환 참조가 생길 수 있을 것이다. 

```swift
class Dog {
    var name: String = "댕댕이"
    
    lazy var closure: () -> () = {
        print("강아지 이름은 \(self.name)입니다.")
    }
    
    deinit {
        print("\(self) 메모리 해제")
    }
}

var dog: Dog? = Dog()
dog?.closure() // 강아지 이름은 댕댕이입니다.
dog = nil // 메모리 해제 X
```

이 경우에는 캡처하고 있는 `name`을 강한 참조로 가지고 있어서 해제가 되지 않는 것이다. 

![](https://i.imgur.com/826fgXv.png)

이러한 상황이기에 클로저 내부에서 캡처하고 있는 부분을 약한 참조로 바꿔줘야 한다. 

이 때 `self`를 약한 참조로 줘야하기 때문에 클로저 캡처시에 `[weak self] in`을 넘겨주면 된다.

```swift 
class Dog {
    var name: String = "댕댕이"
    
    lazy var closure: () -> () = { [weak self] in
        print("강아지 이름은 \(self?.name)입니다.")
    }
    
    deinit {
        print("\(name) 메모리 해제")
    }
}

var dog: Dog? = Dog()
dog?.closure() // 강아지 이름은 Optional("댕댕이")입니다.
dog = nil // 댕댕이 메모리 해제
```

![](https://i.imgur.com/t1bOfib.png)

관계가 이렇게 변경되어서 `dog = nil`을 하면 정상적으로 메모리에서 해제된다. 

## 고민된 점 
- semaphore의 value를 2 이상으로 두어도 될까?
- closure에 self는 왜 쓰는가 

## 해결 방법 
- 학습 내용에 정리!!
---

**Ref**

https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html

https://eastjohntech.blogspot.com/2019/12/closure-self.html
