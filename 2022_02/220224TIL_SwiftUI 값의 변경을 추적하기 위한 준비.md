# SwiftUI 값의 변경을 추적하기 위한 준비

## 220224_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. SwiftUI

뷰는 자신이 어떻게 그려질지 본인이 선언하고 있다는 것이 핵심이다. 색이 어떻게 바뀔지, 모양이 어떻게 바뀔지 컨트롤러가 아닌 뷰가 알고 있다는 것을 생각하자! 

#### `@State`

Property wrapper 타입으로 읽기/쓰기가 가능한 값이며 SwiftUI에 의해 관리된다. 

SwiftUI는 state라고 선언하는 프로퍼티의 저장소들을 관리해준다. 값이 변경되면, SwiftUI는 해당 값에 의존하는 뷰 계층의 일부 부분을 업데이트 해준다. 

`@State` 인스턴스는 값 자체가 아니라 값을 읽고 쓰는 수단이다. state의 기본 값에 접근하려면 해당 프로퍼티 이름을 사용하여 참조하면 `wrappedValue` 프로퍼티 값을 반환해준다. 

예를 들어, 아래 코드처럼 `PlayButton` 뷰에서 프로퍼티를 직접 참조해주면서 `isPlaying` state 프로퍼티를 읽고 업데이트할 수 있다 

즉 뷰 내부에서 특정 뷰의 상태를 나타내는 변수라고 봐도 될 것 같다. 내부에서만 사용 가능하기에 `private`을 붙여줘야 한다! 

```swift
struct PlayButton: View {
    @State private var isPlaying: Bool = false

    var body: some View {
        Button(isPlaying ? "Pause" : "Play") {
            isPlaying.toggle()
        }
    }
}
```

만약 자식 뷰에 state 프로퍼티를 넘겨주게 되면, SwiftUI는 부모의 해당 값이 변경되면 자식 뷰를 업데이트하게 된다. 하지만 자식은 값을 변경할 수 없다. 자식 뷰가 저장된 값을 변경하게 해주려면 `Binding`을 사용해주면 된다. state의 `projectedValue`에 접근하여 state 값에 대한 binding을 가져올 수 있다. `projectedValue`는 프로퍼티 이름 앞에 `$`를 붙여서 얻을 수 있다. 

> ### Binding 
> 바인딩을 사용하여 데이터를 저장하는 프로퍼티과 데이터를 표시하고 변경하는 뷰 간에 양방향 연결을 만들 수 있다. 바인딩은 데이터를 직접 저장하는 대신, 프로퍼티를 다른 곳에 저장된 진짜 소스에 연결한다. 예를 들어 재생과, 일시 중지 사이에서 전환시키는 버튼은 `@Binding` 을 사용하여 상위 뷰의 프로퍼티에 대한 바인딩을 만들 수 있다. 

예를 들어, 위의 예시인 플레이 버튼에서 `isPlaying` state를 제거하는 대신, 버튼을 state와 바인딩해 줄 수 있다.

```swift
struct PlayButton: View {
    @Binding var isPlaying: Bool

    var body: some View {
        Button(isPlaying ? "Pause" : "Play") {
            isPlaying.toggle()
        }
    }
}
```

그런 다음 state를 선언하고 `$` 접두사를 사용하여 state에 대한 바인딩을 만드는 playerView를 정의할 수 있다. 

```swift
struct PlayerView: View {
    var episode: Episode
    @State private var isPlaying: Bool = false

    var body: some View {
        VStack {
            Text(episode.title)
                .foregroundStyle(isPlaying ? .primary : .secondary)
            PlayButton(isPlaying: $isPlaying) // Pass a binding.
        }
    }
}
```

해당 뷰를 인스턴스화 하는 뷰 계층에서는 초기화하면 안된다! 초기화하면 swiftUI가 제공하는 저장소 관리와 충돌이 발생할 수 있다고 한다. 이를 피하기 위해서, 항상 state는 `private`으로 선언하고, 뷰 계층 내 가장 상위 뷰에 위치시켜야 한다. 

`@State`가 붙은 프로퍼티의 값이 바뀌면, 바뀐 값으로 뷰를 다시 그려준다고 한다. 즉 뷰가 다시 렌더링되는 것이다!

정리해보자.
> `@State`: 해당 뷰 안에서만 연동할 정보 

---

#### `ObservableObject`

```swift
protocol ObservableObject : AnyObject
```

객체가 변경되기 전에 내보내는 publisher가 있는 객체 타입이다. 

기본적으로 ObservableObject는 `@Publisehd` 프로퍼티가 변경되기 전에 변경된 값을 내보내는 `objectWillChange` publisher와 동기화된다. 

```swift
class Contact: ObservableObject {
    @Published var name: String
    @Published var age: Int

    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }

    func haveBirthday() -> Int {
        age += 1
        return age
    }
}

let john = Contact(name: "John Appleseed", age: 24)
cancellable = john.objectWillChange
    .sink { _ in
        print("\(john.age) will change")
}
print(john.haveBirthday())
// Prints "24 will change"
// Prints "25"
```
UIKit의 property observer같은 느낌이다!

#### `@Published`

```swift
@propertyWrapper struct Published<Value>
```

속성이 표시된 프로퍼티를 publish하는 타입이다. 

`@Published`를 붙이면 해당 타입의 publisher가 생성된다. `$`를 사용하여 publisher에 접근할 수 있다. 

프로퍼티가 변경되면 프로퍼티의 `willSet` 블록내에서 publishing이 발생한다. 즉 구독자는 프로퍼티에 실제로 설정되기 전에 새 값을 수신할 수 있다.

```swift
class Weather {
    @Published var temperature: Double
    init(temperature: Double) {
        self.temperature = temperature
    }
}

let weather = Weather(temperature: 20)
cancellable = weather.$temperature
    .sink() {
        print ("Temperature now: \($0)")
}
weather.temperature = 25

// Prints:
// Temperature now: 20.0
// Temperature now: 25.0
```

위의 예시에서 sink가 클로저를 두 번째로 실행시킬 때, 25의 값을 받는다. 하지만 클로저가 `weather.temperature`의 값을 구할 때에는 20을 얻게 된다. 

> `@Published` 특성은 클래스에서만 사용해야 한다! 구조체와 같은 클래스가 아닌 타입에 대해서는 사용할 수 없다!

정리해보자.

> ObservableObject, @Published들은 프로퍼티가 바뀌면 알려주는 역할을 한다. 마치 viewModel과 유사하다고 볼 수 있다. 

- `.environmentObject`: 특정 뷰 내부의 모든 뷰에서 모델을 추적할 수 있게 하겠다!

값의 변경을 관측할 수 있게 도와주는 property wrapper
- @ObservedObject
	- 인스턴스 생성시마다 model을 이니셜라이저에 넣어줘야 하는 불편함이 있다. 
- @StateObject
- @EnvironmentObject

- @ObservedObject, @StateObject는 거의 비슷하다. 
	- 변경사항을 구독하는 view에게 알려줌 
	- 또 필요하다면 view가 하위 뷰에 알려줄 수 있다. 
	- 직접 주입의 형태
- @EnvironmentObject
	- 주입할 최상위뷰에 변화를 알려주면, 모든 하위뷰에 주입하지않더라도 접근할 수 있다. 

> SwiftUI <-> MVVM 
	- 핵심은 뷰모델은 뷰를 모른다. 
	- 뷰는 뷰모델의 값을 꺼내써야한다는 것! 

## 고민된 점 
- SwiftUI에서 뷰 처리 로직, 값의 변경을 추적하는 방법

## 해결 방법 
- 각종 property wrapper 찾아보기

---

**Ref**

코다의 활동학습

https://developer.apple.com/documentation/swiftui/state

https://developer.apple.com/documentation/swiftui/binding

https://developer.apple.com/documentation/combine/observableobject

https://developer.apple.com/documentation/combine/published
