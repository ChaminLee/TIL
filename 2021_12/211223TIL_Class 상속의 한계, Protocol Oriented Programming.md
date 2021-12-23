# Class 상속의 한계, Protocol Oriented Programming

## 211223_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Class 상속의 한계

> 1 . 상속을 하게 되면 필요없는 기능도 받아야 한다.

적합한 예시가 떠오르진 않지만... 예를 들어서 A 클래스에는 a, b 메서드가 구현되어 있는데, B가 A 클래스를 상속했다고 해보자. 

근데 만약 B는 A 클래스의 a 기능만 원하고 b 기능은 원하지 않더라도, 통째로 상속받게 된다. 

> 2 . retain count를 관리하는 비용이 든다.

클래스의 경우 참조 타입으로 heap 영역에 할당되며 동적으로 retain count를 관리하게 된다. 이를 관리하는 비용이 부담이 될 수 있다. 

> 3 . 다중 상속이 불가능하다

클래스는 다중 상속이 불가능하다. A 클래스에 B, C 클래스의 기능을 모두 상속해주고 싶은 경우에 두 개를 상속해줄 것 처럼 보이겠지만, 클래스는 다중 상속이 안된다. 

### Protocol Oriented Programming란?

protocol은 기존 class 상속의 한계를 극복할 수 있다. 수직적인 관계를 나타내는 상속과는 달리 protocol은 수평적인 관계를 나타낼 수 있다. 
또한 class 뿐만 아니라 struct, enum에 protocol 채택이 가능하여 상속을 대신할 수 있다. 

이 덕분에 class 보다 성능이 좋다는 struct를 사용할 수 있게 되었다. 

장점을 간단하게 살펴보자. 

- 다중 채택이 가능하다. 
- protocol은 struct, enum, class 모두 채택 가능하다
- 수평 구조의 기능 확장이 가능하다

이러한 protocol을 사용하여 상속처럼 사용할 수 있는데, 이는 extension과 protocol의 조합 덕분이다. 

기존에는 protocol만 사용했기에 청사진 역할만 했지만, 이제는 필요한 기능에 대해서는 extension에서 기본 구현이 가능하게 되었다. 

이전에는 protocol을 구현하고 채택하면, 채택한 타입에서 일일이 요구 사항들을 구현해줘야했다. 

```swift
protocol Walkable {
    func walk()
}

struct Person: Walkable {
    func walk() {
        print("걷다")
    }
}

struct Dog: Walkable {
    func walk() {
        print("걷다")
    }
}

struct Cat: Walkable {
    func walk() {
        print("걷다")
    }
}
```

동일한 기능을 하는 `walk()` 메서드의 구현이 불필요하게 반복되는 것을 볼 수 있다. 

이러한 부분을 extension + protocol로 해결할 수 있다는 것이다. 

```swift
protocol Walkable {
    func walk()
}

extension Walkable {
    func walk() {
        print("걷다")
    }
}

struct Person: Walkable { }
struct Dog: Walkable { }
struct Cat: Walkable { }

let person = Person()
person.walk()

let dog = Dog()
dog.walk()

let cat = Cat()
cat.walk()
```

extension에 동일한 기능을 기본 구현해주면, 해당 protocol을 채택하는 타입은 필수 구현을 해주지 않아도 기본 구현된 메서드를 사용할 수 있게 된다. 

하지만 기본 구현이 있음에도 불구하고 타입 내에 기능을 구현해주는 경우에는 타입 내 구현이 우선적으로 작동된다. 

```swift
protocol Walkable {
    func walk()
}

extension Walkable {
    func walk() {
        print("걷다")
    }
}

struct Person: Walkable {
    func walk() {
        print("걷자아아")
    }
}

let person = Person()
person.walk() // 걷자아아
```

만약 여기서 protocol의 기본 요구 메서드를 제거해보면 어떻게 될까? 

```swift
protocol Walkable { }

extension Walkable {
    func walk() {
        print("걷다")
    }
}

struct Person: Walkable {
    func walk() {
        print("걷자아아")
    }
}

let person: Walkable = Person()
person.walk() // 걷다
```

`Walkable` protocol의 청사진(?)에 메서드를 제거하고 기본 구현으로만 해주었다. 이 때 `person`이 `Walkable` 타입이라면 extension의 `walk()`를 찾아가서 "걷다"를 출력하게 되고, `person`이 `Person` 타입이라면 타입 내부의 메서드를 찾아가 "걷자아아"를 출력하게 된다. 

### Protocol Oriented Programming 한계 

이렇게 좋다좋다하는 POP도 한계가 있다. 
(프로토콜의 한계도 같이 보자)
	
- Objective-C 프로토콜에는 extension으로 기본 구현을 하지 못한다. 
	- 아직까지는 지원이 되지 않고 있다고 한다.
- 채택시 요구되는 기능을 모두 구현해야한다.
	- class의 상속은 추가 구현 없이 모두 기능을 상속시켜줬었다.
- 프로토콜을 채택한 타입에서 구현하게 되면, extension에서 한 기본 구현을 덮어쓴다.
	- 기본 구현을 인지하고 있어야 한다는 단점이 있다.
	- 모르고 썼다가는 덮어쓰거나, 원치 않는 기능을 호출하게 될 위험이 있다. 
- 저장 프로퍼티를 사용할 수 없다.
- 프로토콜 기본 구현을 하게 되면, 이후 타입이 프로토콜 채택시 필수 구현 요소로 알려주지 않는다. 
	- 기본 구현되어있는 요소들을 찾아봐야 어떤 기능을 지원하는지 알 수 있게된다. 

이렇듯 POP는 장점도 있지만, 분명히 한계점도 있다. 그렇기에 protocol만을 고집하기 보다는 상황에 따라서 class의 상속을 쓰기도 하고, 때로는 두 가지를 조합해서 문제를 해결하는 방식이 바람직하다고 생각된다.


## 고민된 점 
- POP란?
- POP를 현명하게 쓸 수 있는 방법?

## 해결 방법 
- POP와 class 상속의 한계 파악
---

**Ref**

[야곰 POP 강의](https://academy.realm.io/kr/posts/protocol-oriented-programming-in-swift/)

