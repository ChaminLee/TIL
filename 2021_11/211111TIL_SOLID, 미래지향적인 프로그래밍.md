# SOLID, 미래지향적인 프로그래밍

## 211111_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### SOLID 원칙

붱이의 SOLID 강의를 듣고, 다른 참고 자료도 보면서 5가지 원칙의 핵심만 우선 간략하게 정리해보려고 한다. 

소프트웨어에서의 중요한 가치 : 가독성, 단순성, 유연성

> 1 . 단일 책임의 원칙 (SRP: Single Responsibility Principle)

**하나의 객체는 하나의 책임을 가져야한다는 원칙이다.** 즉, 하나의 객체가 여러 책임을 가지게 된다면 유연성이 떨어지고, 재사용성 또한 떨어지게 된다. 

또 이런 이야기가 있다.
> 클래스가 변하는데 하나 이상의 이유가 있으면 안된다. 

그러니까 책임이 여러 개인 경우, 각 책임에 대한 변경이 있을 경우 여러 이유들로 인해 해당 클래스가 바뀔 수 있다는 것이다. 이러한 상황을 지양하기 위해 "하나의 클래스는 하나의 책임만을 갖고, 이를 관리한다"라고 이해해도 좋을 것 같다. 

> 2. 개방-폐쇄의 원칙 (OCP: Open-Closed Principle)

**확장에는 열려있고, 변경에는 닫혀있어야 한다.**는 원칙이다. 여기서 확장이란 새로운 코드(기능)의 추가가 될 것이고, 변화는 새로운 코드가 추가됨에 동반될 수 있는 변경사항을 이야기하는 것 같다. 

즉 **확장**했을 때 별다른 추가 수정 없이 기능을 확장할 수 있어야 하고, 확장에 따라 발생할 수 있는 **변경**을 없애는 것이 OCP의 핵심이 될 것 같다. 

```swift
class Vehicle {
    func printData() {
        let cars = [
            Car(name: "Batmobile", color: "Black"),
            Car(name: "SuperCar", color: "Gold"),
            Car(name: "FamilyCar", color: "Grey")
        ]

        cars.forEach { car in
            print(car.printDetails())
        }
    }
}

class Car {
    let name: String
    let color: String

    init(name: String, color: String) {
        self.name = name
        self.color = color
    }

    func printDetails() -> String {
        return "I'm \(name) and my color is \(color)"
    }
}
```

예시로 많이 드는 `Vehicle`과 `Car` 클래스를 보면 처음에는 `Car`만 존재하기 때문에 문제가 없어보인다. 하지만 `Bicycle`이라는 새로운 클래스가 추가되는 경우에 해당 코드는 OCP를 위배하게 될 것이다. 

```swift
class Vehicle {
    func printData() {
        let cars = [
            Car(name: "Batmobile", color: "Black"),
            Car(name: "SuperCar", color: "Gold"),
            Car(name: "FamilyCar", color: "Grey")
        ]

        cars.forEach { car in
            print(car.printDetails())
        }
        // 새로운 클래스 추가로 인한 불필요한 반복 = 재활용 못하고 있음
        let bicycles = [
            Bicycle(type: "BMX"),
            Bicycle(type: "Tandem")
        ]

        bicycles.forEach { bicycles in
            print(bicycles.printDetails())
        }
    }
}

class Car {
    let name: String
    let color: String

    init(name: String, color: String) {
        self.name = name
        self.color = color
    }

    func printDetails() -> String {
        return "I'm \(name) and my color is \(color)"
    }
}

class Bicycle {
    let type: String

    init(type: String) {
        self.type = type
    }

    func printDetails() -> String {
        return "I'm a \(type)"
    }
}
```

왜냐? 확장함에 있어 쉽게 할 수 없고, 변경되는 부분이 있기 때문에 OCP를 위반한다고 볼 수 있을 것 같다.

이에 `추상화`에 의존하는 방식으로 변경하면 확장이 쉽고, 변경또한 방지할 수 있다고 한다. 

```swift
protocol Printable {
    func printDetails() -> String
}

class Vehicle {
    func printData() {
        let vehicles: [Printable] = [
            Car(name: "Batmobile", color: "Black"),
            Car(name: "SuperCar", color: "Gold"),
            Car(name: "FamilyCar", color: "Grey"),
            Bicycle(type: "BMX"),
            Bicycle(type: "Tandem")
        ]

        vehicles.forEach { car in
            print(car.printDetails())
        }
    }
}

class Car: Printable {
    let name: String
    let color: String

    init(name: String, color: String) {
        self.name = name
        self.color = color
    }
    // 프로토콜 채택 이후, 각 클래스 내에서 기능 구현
    func printDetails() -> String {
        return "I'm \(name) and my color is \(color)"
    }
}

class Bicycle: Printable {
    let type: String

    init(type: String) {
        self.type = type
    }
    // 프로토콜 채택 이후, 각 클래스 내에서 기능 구현
    func printDetails() -> String {
        return "I'm a \(type)"
    }
}
```

이제는 어떠한 새로운 클래스가 추가되더라도, `Printable` 프로토콜만 채택하고 있다면 `Vehicle`의 `printData()` 메서드의 기능을 쉽게 확장할 수 있게 된다. 또한 확장으로 인해 발생하는 기존 코드의 변경 사항이 없기 때문에 OCP 원칙을 지키고 있다고 볼 수 있다. 

> 3. 리스코프 치환 법칙 (LSP: The Liskov Substitution Principle)

치환이라는 단어에 포커싱을 맞춰 이해하면 좋을 것 같다. 상위 타입을 S라고 하고 하위 타입을 T라고 할 때, 프로그램의 속성(코드) 변경없이 **하위 타입(T)이 상위 타입(S)으로 치환(교체)될 수 있어야 한다**는 원칙이다.

쉽게 생각해보면 상위 타입을 상속하기에 하위 타입이 생길 수 있다. 그렇다면 상위 타입에서 제공하는 기능이나 속성들을 모두 사용할 수 있어야 한다. 이에 따라 역으로 생각했을 때 하위 타입은 상위 타입이 갖고 있는 요소들을 가지고 있으니 치환될 수 있어야 한다는 것이다. 

하지만 주로 예시로 드는 직사각형, 정사각형 예시를 보면 LSP를 위반하고 있음을 볼 수 있다. 

직사각형은 정사각형보다 더 큰 개념이다. (정사각형은 사각형이면서 네 변의 길이가 같아야 하기 때문이다) 

```swift
class Rectangle {
    var width: Float = 0
    var length: Float = 0

    var area: Float {
        return width * length
    }
}

class Square: Rectangle {
    override var width: Float {
        didSet {
            length = width
        }
    }
}
```

이렇게 직사각형(Rectangle)과 정사각형(Square)을 정의해 볼 수 있다. 이제 Rectangle 타입을 파라미터로 받아 넓이를 구해주는 메서드를 만들어보자. 

```swift
func printArea(of rectangle: Rectangle) {
    rectangle.length = 5
    rectangle.width = 2
    print(rectangle.area)
}

let rectangle = Rectangle()
printArea(of: rectangle) // 10

// -------------------------------

let square = Square()
printArea(of: square) // 4
```

`printArea(of: )` 메서드에 상위 타입에 해당하는 `Rectangle` 타입의 인스턴스를 넣으면 정상적으로 넓이를 구할 수 있다. 하지만 하위 타입의 인스턴스인 `square`를 넣게 되면 직사각형의 넓이가 아닌 정사각형의 넓이가 나오게 된다. 
즉 하위 타입이 상위 타입을 완벽하게 치환하지 못하는 상황이 발생하는 것이다. 다시 정리해보면 치환하지 못하는 이유는, 하위 타입이 상위 타입의 역할을 모두 수행하지 못하기 때문이다. 그렇기에 하위 타입이 상위 타입의 역할을 모두 수행하기 위해서는 `추상화`에 의존해야한다고 말한다. protocol이다!

```swift
protocol Polygon {
    var area: Float { get }
}

class Rectangle: Polygon {

    private let width: Float
    private let length: Float

    init(width: Float, length: Float) {
        self.width = width
        self.length = length
    }

    var area: Float {
        return width * length
    }
}

class Square: Polygon {

    private let side: Float

    init(side: Float) {
        self.side = side
    }

    var area: Float {
        return pow(side, 2)
    }
}

// Client Method

func printArea(of polygon: Polygon) {
    print(polygon.area)
}

// Usage

let rectangle = Rectangle(width: 2, length: 5)
printArea(of: rectangle) // 10

let square = Square(side: 2)
printArea(of: square) // 4
```

이제는 `Square` 타입은 `Rectangle`의 하위 타입이 아니라, `Polygon`이라고 하는 추상화에 의존하게 된다. 이 덕분에 예상하는대로 값을 얻어올 수 있게 되고 LSP를 지키는 코드를 작성할 수 있게 된다. 

> 4. 인터페이스 분리 원칙 (ISP: The Interface Segregation Principle)

클라이언트들은 이들이 직접 사용하지 않는 인터페이스에 의존해서는 안된다는 원칙이다. 즉, 불필요한 인터페이스 요소들을 포함시키지 말라는 것이다. 이렇게 불필요한 인터페이스들이 많은 경우를 `Fat`하다고 말하는 것 같다. 

버튼을 터치했을 때 반응을 구현해줄 메서드를 protocol에 넣은 케이스를 봐보자.

```swift
protocol GestureProtocol {
    func didTap()
    func didDoubleTap()
    func didLongPress()
}
```

어떤 버튼은 저 3가지 메서드를 필요로 할 수 있지만, 다른 버튼은 `didTap()`만 필요로 하는 경우가 있을 수 있다. 그렇다면 이 때 그냥 다른 메서드들은 무시하고 `GestureProtocol`을 채택해야 할까? 답은.. 그렇게 채택해버리면 ISP 원칙을 무시하게 되는 것이다. 

결국에 필요한 인터페이스들만 제공해주기 위해 protocol을 분리하는게 방법일 것 같다. 

```swift
protocol TapProtocol {
    func didTap()
}

protocol DoubleTapProtocol {
    func didDoubleTap()
}

protocol LongPressProtocol {
    func didLongPress()
}

class SuperButton: TapProtocol, DoubleTapProtocol, LongPressProtocol {
    func didTap() {
        // send tap action
    }

    func didDoubleTap() {
        // send double tap action
    }

    func didLongPress() {
        // send long press action
    }
}

class PoorButton: TapProtocol {
    func didTap() {
        // send tap action
    }
}
```

이렇게 protocol을 나누고 필요한 인터페이스가 있다면 골라서 채택하는 방식으로 한다면 ISP 원칙을 지킬 수 있다. 


> 5. 의존관계 역전 원칙 (DIP: Dependency Inversion Principle)

의존관계 역전 원칙에는 두 가지 제약이 있다. 

- 고차원의 모듈들은 하위 모듈에 의존하면 안된다. 두 개 모두 추상 개념에 의존해야한다. 
- 추상 개념(abstraction)은 세부 사항(details)에 의존하면 안된다. 세부 사항이 추상 개념에 의존해야한다. 

SOLID에서 계속 이야기하는 "추상화", "추상 개념"을 우선 protocol로 이해해봐도 좋을 것 같다. 위의 두 가지 조건을 다시 이야기 해보자면 결국은 "추상화"에 의존해야한다는 건데.... 추상화의 특징은 "중요한 특성", "변화하지 않는 특성" 이라고 정리해 볼 수 있을 것 같다. 

음... 다시 생각해보면 **변하지 않는 것에 의존해서 확장 가능한 코드를 만들어라**는게 핵심인 것 같다. 

```swift
class Handler { 
    let fm = FilesystemManager()
 
    func handle(string: String) {
        fm.save(string: string)
    }
}
 
class FilesystemManager { 
    func save(string: String) {
        // Open a file
        // Save the string in this file
        // Close the file
    }
}
```
이렇게 고차원인 `Handler`와 저차원인 `FilesystemManager`의 관계를 봤을 때 DIP를 위반했다고 볼 수 있다.  이런 상황에서는 Handler를 재사용하기가 어려울 것이다. (고차원 모듈이 저차원 모듈에 의존하고 있고, 추상 개념이 아닌 세부 사항에 의존하고 있기 때문)

이 경우에도 추상 개념에 의존하도록 바꿔주면 된다. 

```swift
protocol Storage {
   func save(string: String)
}

class Handler {
    let storage: Storage
 
    init(storage: Storage) {
        self.storage = storage
    }
 
    func handle(string: String) {
        storage.save(string: string)
    }
}
 
class FilesystemManager: Storage {
    func save(string: String) {
        // Open a file in read-mode
        // Save the string in this file
        // Close the file
    }
}
 
class DatabaseManager: Storage {
    func save(string: String) {
        // Connect to the database
        // Execute the query to save the string in a table
        // Close the connection
    }
}
```

이렇게 `Storage`라는 추상 개념을 만들고 이에 의존하게 된다면 `Handler` 자체의 재사용성도 올라갈 뿐 아니라, 확장에도 용이하게 된다. `DatabaseManager`과 같은 새로운 클래스를 만들 때도 추상 개념에 의존하도록 만들어 쉽게 확장할 수 있다. 

> 붱이의 TDD TIP. 가짜 객체를 만드는 이유는 제어할 수 없는 상황을 제어할 수 있는 환경으로 만들어주기 위함이다. 인위적으로 값을 세팅해서 원하는 환경을 만들어주는 것이다. 



## 고민된 점 

- SOLID란 무엇이며... 왜 쓰며... 어떻게 쓰는가..

## 해결 방법 

- 학습 내용에 정리 
---

**Ref**

붱이의 SOLID 강의
[전에 정리했던 SOLID](https://leechamin.tistory.com/518)
[SOLID](https://www.marcosantadev.com/solid-principles-applied-swift/)
[SOLID-DIP](https://huisam.tistory.com/entry/DIP)
