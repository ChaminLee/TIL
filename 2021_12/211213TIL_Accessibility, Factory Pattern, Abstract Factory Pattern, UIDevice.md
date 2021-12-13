# Accessibility, Factory Pattern, Abstract Factory Pattern, UIDevice

## 211213_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### !복습 내용!

- JSON에서 없는 값은 `nil`이 아니라 `null`로 표기한다
- `content hugging`과 `resistance`는 반대되는 개념은 아니다.
- swift의 `contentConfiguration` 은 iOS 14.0 이상만 지원하며 iOS 14.0 아래로는 지원해주지 않는다. 
  - `.hangulWordPriority`도 iOS 14.0 이상에서 지원된다. 
  
  
### 1. Accessibility 

직역하면 접근성이다. 말 그대로 누구나 앱에 대한 접근성이 동일, 평등해야한다는 애플의 이념이 들어간 부분이다. 

예를 들어서 설명하자면 눈이나 귀가 조금 불편하신 분들이 앱을 사용하는데 문제가 없어야 한다는 것이다. 이에 폰트 사이즈나, 색의 대조, UI 요소를 읽어줄 때 맥락에 맞도록 하는 등 다양한 부분들을 Accessibility에서 관리하고 있다. 

#### 1-1. Text Accessibility

텍스트에 대한 부분 먼저 살펴보자.

먼저 iPhone의 설정에 가면 글자 크기를 바꿔줄 수 있다. 하지만 기기에서 바꿔준다고 모든 앱이 기기 설정에 맞게 글자 크기를 반영시켜주지 않는다. 이러한 앱들은 Accessibility를 신경쓰지 않은 앱이라고도 볼 수 있다. 

생각해보면 기기 설정을 바꿨는데 앱에 적용이 되지 않는다는게 조금 아이러니한 부분이다. 그렇기에 Accessibility를 고려해줄 필요가 있다. 이러한 이유 말고도 모두가 평등하게 앱을 사용할 수 있도록 하는 것이 조금 더 바람직한 모습에 가까운 것 같다. 

자 그러니 이제 폰트 사이즈를 사용자가 설정한 크기대로 맞춰서 보여줘야 한다. 이전까지는 시스템 폰트를 원하는 크기를 지정해서 설정했었다. 예시로 메인 화면에 나타나는 `UILabel`의 사이즈를 30으로 설정했다고 해보자. 그렇다면 이 `UILabel`의 크기는 기기의 폰트 크기와 상관없이 계속 30이 될 것 이다. 이러한 부분을 개선하기 위해서는 사용하는 폰트가 여러가지 사이즈를 지원해줘야 하는 것이 첫 번째 선행되어야 하는 부분이다. 

즉, 우리가 사용하는 폰트가 dynamic하게 변하려면 그만큼의 다양한 문자 크기를 가지고 있어야 한다. 

그래서 [Typography](https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/typography#dynamic-type-sizes)에 가보면 Apple이 지원하는 글씨체의 다양한 크기를 살펴볼 수 있다. LargeTitle부터 Caption2까지 총 11개의 카테고리를 기본적으로 제공해주고 있다. 이제 이 카테고리들이 기기의 폰트 크기에 따라 유동적으로 값이 변하게 되는 것이다. 

같은 LargeTitle이라고 해도 기기에서 `small`일 경우 32 포인트이며, `default`일 때는 34 포인트이다. 이제 이 표만 봐도 동적으로 문자 크기를 지원해준다는 것을 알 수 있다. 그렇다면 코드로는 어떻게 설정해야 하는지 살펴보자. 

```swift
// 1. 우선 카테고리 내 사이즈를 선택한다.
sampleLabel.font = UIFont.preferredFont(forTextStyle: .largeTitle)
// 2. 
sampleLabel.adjustsFontForContentSizeCategory = true
```

먼저 기본으로 주어지는 카테고리내에서 선택하여 폰트 크기를 정해야한다. 그 이후`adjustsFontForContentSizeCategory`를 true로 세팅해준다. `adjustsFontForContentSizeCategory` 프로퍼티는 Boolean 값으로 디바이스의 컨텐츠 사이즈 카테고리가 변화되었을 때 객체가 자동적으로 폰트를 업데이트 시킬지 여부를 나타낸다. 즉 현재의 경우 폰트 크기의 자동적인 변화를 필요로하기에 true로 세팅한 것을 볼 수 있다. 

> `adjustFontSizeToFitWidth`도 있는데, 이는 너비에 폰트 크기를 맞춰준다. 그래서 아무리 글씨가 커지려고 해도 그 크기는 너비에 따라 결정된다. 

#### 1-2. Voice Over

시각이 불편하신 분들을 위한 기능이 될 것 같다. 화면의 UI요소들을 문자가 아닌 소리로 표현을 해주는 방식이다. 예를 들어 화면에 어떤 상품을 장바구니에 넣는 `+` 버튼이 있다고 해보자. 화면을 볼 수 있는 사람이라면 문맥을 읽고 "아 A 제품을 카트에 넣는거구나" 인지할 수 있다. 하지만 화면을 보는데 어려움이 있는 사람들은 이를 유추하기 어렵기 때문에 음성으로 "A 제품을 카트에 추가"라고 알려주는 것이다.

그렇다면 이 안내 문구는 어떻게 추가해야하고, 어떤 방식으로 표현해야 적합하고 문맥에 맞을 수 있을까? 

우선 어떻게 추가하는지 알아보자. 

화면에 다음 뷰로 화면 전환을 해주는 `UIButton`이 하나 있다고 가정해보자. 먼저 스토리보드에서 세팅하는 방법을 알아보자. 

우선 `UIButton`을 누르고 우측 `Identity inspector`를 살펴보자.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FplPYG%2FbtrnJBR7uMZ%2FyImeneAltheZZyvte7B6z1%2Fimg.png)

그러면 하단에 Accessibility라는 부분이 있을 것이다. 여기서 우선 기본적으로는 `Accessibility`에 Enabled로 체크가 되어있지만 잘 확인해주어야 한다. 

> #### Tip
>`UIImageView`는 기본적으로 `isAccessibilityElement`가 false(체크가 안되어있다)이다. 그렇기에 `true`로 바꿔줘야하며, `label` 또한 추가해줘야 한다. 

우리가 값을 넣어야 하는 곳을 살펴보자.

- `label` (`accessibilityLabel`) : 여기에 설정한 값을 읽어준다. 요소가 `UIButton`이기 때문에 "label의 값 + 버튼" 이라고 읽어준다.
- `Hint` (`accessibilityHint`) : 요소를 동작시키기 위한 방법을 알려준다.
- `value`: (`accessibilityValue`) : inspector에는 없지만 값이 달라지는 경우에 사용하는 요소이다. 즉, 그 때 그 때의 현재 값을 알려준다. `UISlider`나 `UITextField`에 사용할 수 있다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbz0q0V%2FbtrnP73fihX%2FS1qw3kJk9k9R7Ju1GqOEQk%2Fimg.png)

`UITextField`에 "주문하기"라는 텍스트가 들어간 것을 볼 수 있다. `UITextField`나 `UISlider`와 같은 `UIControl`을 상속하는 타입의 경우 기본적으로 컨텐츠를 `value`와 `frame` 속성에 제공한다. 그렇기에 따로 코드나 스토리보드에서 value, label을 세팅해줄 필요가 없을 것 같다.  

> #### Tip
> 외국어로 발음하려면?
> - 맥 손쉬운 사용- 콘텐츠 말하기 - 언어 변경

> #### Tip
> 버튼의 크기가 작다면? Hit Test의 영역이 작다면?
> 버튼의 contentEdgeInset을 늘려서 터치 영역을 넓힐 수 있다. 


### 2. Factory Pattern

Factory pattern이라는 말 그 자체로 객체를 생성해주는 역할을 한다. Factory pattern의 정의는 다음과 같다. 

> 객체를 생성하기 위한 인터페이스를 정의하는데, 어떤 클래스의 인스턴스를 만들지는 서브 클래스에서 결정하게 만드는 패턴이다. 

즉 객체를 통해 인스턴스를 만드는 것이 아니라 Factory method를 통해 인스턴스를 생성하고 사용하게 되는 것이다. 어떤 과정으로 인스턴스가 생성되는지 그 과정에서 알아야 하는 세부 사항들을 줄여주는 효과가 있다. 

예를 들어 storyboard를 기반으로 View Controller의 인스턴스를 만들려고 할 때 알아야 하는 값이 두 가지 있다. 바로 storyboard의 name과 identifier이다. 

매번 View Controller를 생성할 때 마다 storyboard 각각의 name과 identifier를 기억하고 있기는 힘들다. 이에 이런 부분을 factory method에 담아 클라이언트는 인스턴스가 어떻게 생기게 되는지 모르게 되고 그저 반환되는 인스턴스를 사용하게 된다. 즉 클라이언트 입장에서는 storyboard의 name과 identifier를 모르고 factory만 알면, 원하는 View Controller의 인스턴스를 생성하여 사용할 수 있다는 것이다. 

즉 동일한 공정을 이용해서 서로 다른 인스턴스를 만드는 것과 같다.

- `Factory Pattern`의 장점
	- 확장성이 높다.
	- 의존성/결합도가 낮아진다.
	- 인스턴스의 사용과 생성을 분리할 수 있다. 

자 이제 어떤 이유에서 factory pattern을 사용해야하는지 예제를 보면서 살펴보자. 

탈 것을 예로 들어서 이야기해보자.

우선 탈 수 있다는 것을 나타내기 위한 `Rideable` 프로토콜을 정의해준다. 

```swift
protocol Rideable {
    var brandName: String { get }
    
    func ride()
}
```
자 이제 이 프로토콜을 채택하는 차, 자전거 타입을 생성해보자. 

```swift
struct Car: Rideable {
    var brandName: String
    
    func ride() {
        print("\(self.brandName)타고 가자!")
    }
}

struct Bicycle: Rideable {
    var brandName: String
    
    func ride() {
        print("\(self.brandName)타고 가자!")
    }
}
```

보통은 이렇게까지 만들고 필요한 부분에서 `Car`나 `Bicycle`을 초기화하여 인스턴스를 생성해서 사용했을 것이다. 

만약 어떤 특정 View Controller에서 인스턴스를 생성해서 사용했다고 가정해보자. 

```swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
		let bmw = Car(brandName: "BMW")
        bmw.ride() // BMW타고 가자!
    }
}
```

자 그런데 만약에 `Car`에 새로운 프로퍼티가 추가되어서 이니셜라이저가 변경되는 경우를 생각해보자. 만약 `price`라는 프로퍼티가 추가되었다고 생각해보자. 

그렇다면 타입을 수정하게 되고, 이에 따라 불가피하게 해당 타입의 인스턴스를 생성했던 부분의 수정 또한 필요하게 된다. 지금은 한 군데에만 사용을 했지만.... 만약 10개의 View Controller에서 `Car` 타입의 인스턴스를 생성했다면 일일이 찾아가서 초기화 부분을 모두 다 수정해줘야 할 것이다. 

이런 부분에 있어서 효율적으로 사용할 수 있는 것이 바로 Factory pattern이다. 

위에서 말한 것 처럼 직접 인스턴스를 생성하는 것이 아니라 Factory를 통해 인스턴스를 생성해주는 것이다. 

```swift
enum VehicleType {
    case car
    case bicycle
}

class VehicleFactory {
    func creatVehicle(brandName: String, vehicleType: VehicleType) -> Rideable {
        switch vehicleType {
        case .car:
            return Car(brandName: brandName)
        case .bicycle:
            return Bicycle(brandName: brandName)
        }
    }
}
```

이렇게 Factory 타입을 만들어줄 수 있다. 이제 실제로 사용되는 부분에서는 인스턴스화 되는 과정을 세세하게 알지 못해도 인스턴스를 생성하고 사용할 수 있게 된다. 

아까의 View Controller의 코드가 아래처럼 바뀔 수 있다. 

```swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
		let vehicleFactory = VehicleFactory()
        let bmw = vehicleFactory.creatVehicle(brandName: "BMW", vehicleType: .car)
        bmw.ride() // BMW타고 가자!
    }
}
```

이제는 `Car`의 이니셜라이저가 아무리 바뀌어도 ViewController에서는 신경써주지 않아도 된다. 딱 한 군데, Factory method 부분만 수정해주면 된다. 이러한 부분에서 확장성과 재사용성이 높다고 말할 수 있을 것 같다. 또한 직접적으로 `Car`에 접근하는 것이 아니라 Factory를 거치기 때문에 의존성/결합도 또한 낮아지는 것을 확인할 수 있다.

결론적으로. 이제 직접 각 객체를 통해 인스턴스를 만들지 않고 Factory를 통해 인스턴스를 만들 수 있다는 것이 큰 장점이다.


### 3. Abstract Factory Pattern 

이를 "추상 팩토리 패턴"이라고 많이 부르는 것 같다. Abstract Factory Pattern은 앞서 봤던 Factory Pattern과 유사한데, 차이점이라고 하면 Abstract Factory Pattern은 Factory를 만드는 상위 클래스가 있다는 점이다.

즉 쉽게 말하면 Factory를 만드는 더 상위의 Factory가 있는 셈이다. 

앞선 예제를 변형해보면서 이해해보자. 

차를 만드는 회사 중에 다들 알고 있을 만한 회사로 예시를 들어보자. `BMW`와 `Benz`가 있다고 해보자. 이 둘은 모두 이동수단을 만드는 역할을 한다.

만약에 이제부터 차 말고도 자전거도 같이 만들어야 한다고 하는 경우라고 가정해보자. 

```swift
protocol VehicleFactory {
    func createCar() -> Car
    func createBicycle() -> Bicycle
}
```

이를 활용하여 각 제조사별 Factory를 생성할 수 있다. 

```swift
class BMWFactory: VehicleFactory {
    func createCar() -> Car {
        return Car(brandName: "BMW")
    }
    
    func createBicycle() -> Bicycle {
        return Bicycle(brandName: "BMW")
    }
}

class MercedesBenzFactory: VehicleFactory {
    func createCar() -> Car {
        return Car(brandName: "Mercedes-Benz")
    }
    
    func createBicycle() -> Bicycle {
        return Bicycle(brandName: "Mercedes-Benz")
    }
}

```

사실 이렇게 각 Factory를 만든 것 까지 `Factory Pattern`이라고 볼 수 있다. 하지만 지금은 N개의 Factory가 만들어질 수 있는 상황이고 이를 묶어주는 상위의 Factory를 보기 위함이다. 

이제 이를 가지고 무난하게 사용해볼 수 있다. 

```swift
let bmwFactory = BMWFactory()
let bmwCar = bmwFactory.createCar()
let bmwBicycle = bmwFactory.createBicycle()
```

`Mercedes-Benz`도 마찬가지로 동일하게 사용해줄 수 있는데, 이제 제조사 또한 신경을 크게 쓰지 않아도 되도록 상위 Factory인 `ManufacturerFactory`를 만들어 볼 것이다. 

```swift
class ManufacturerFactory {
    static func createFactory(manufacturer: VehicleManufacturer) -> VehicleFactory {
        switch manufacturer {
        case .bmw:
            return BMWFactory()
        case .mercedesBenz:
            return MercedesBenzFactory()
        }
    }
}
```

이제 이 상위 Factory를 가지고 Factory 객체를 만들어 줄 수 있다. 이제 그 다음부터는 Factory pattern 때 사용하던 방식과 동일하다. 

- `Abstract Factory Pattern`의 장점
	- 확장성이 높다.
	- 객체간의 의존성/결합도가 낮아진다.
	- SOLID의 SRP, OCP를 따른다. 
		- 생성자 코드를 분리하여 코드의 역할이 분배되었다
		- 새로운 타입 추가에 자유롭다
- 단점 
	- 너무 깊어지면 찾기가 어렵다..!

이렇게 Factory Pattern에 대해 알아보았는데, 이를 실제 View Controller를 생성한다거나, 특정 객체를 생성할 때 적용해서 장점을 취하면 좋을 것 같다. 

### 4. UIDevice

```swift
@MainActor class UIDevice : NSObject
```

현재 디바이스에 대한 정보를 알려주는 역할을 한다. 

디바이스의 모델이나, 운영체제 이름, 버전등을 알 수 있다. 이번에는 OS의 이름과 버전의 정보를 알아내보자. 


#### 1. `UIDevice().systemVersion` 
- 현재 OS의 버전을 알려준다. 
- 만약 `iOS 14.5`였다면 `14.5` 라고 나타난다. 

이를 활용해서 버전에 따라 레거시 코드를 관리할 수 도 있고, 버전에 따라 지원 가능한 기능들을 추가해줄 수도 있다!

```swift
// iOS 14,0 이상일 때 실행할 코드 
if  #available(iOS  14.0, *) { 
    // function 
}

// iOS 14.0 이상부터 가능한 코드 
@available(iOS  14.0, *)
// ~ 코드 ~ 
```

#### 2. `UIDevice().systemName`
- 운영체제의 이름을 알려준다. 
- 기기마다 iOS, iPadOS, macOS 등 다른 운영체제를 사용하는데 이 이름을 알려준다. 
- 즉 iPhone에서 호출하면 `iOS`라고 알려준다. 



## 고민된 점 
- `Factory Pattern`이란?
- Accessibility를 지원하는 방법은?

## 해결 방법 

---

**Ref**

[scaling fonts automatically](https://developer.apple.com/documentation/uikit/uifont/scaling_fonts_automatically)

[UIControl-Accessibility](https://developer.apple.com/documentation/uikit/uicontrol)

[UIImageView-Accessibility](https://developer.apple.com/forums/thread/72729
)

[View Controller Factory in Swift](https://medium.com/@sreeks/view-controller-factory-in-swift-1d4ee8f9b54)

[The Factory Pattern using Swift](https://stevenpcurtis.medium.com/the-factory-pattern-using-swift-b534ae9f983f)

[Factory Pattern 1](https://velog.io/@ellyheetov/Factory-Pattern)

[Factory Pattern 2](https://jeonyeohun.tistory.com/211)

[Factory Pattern 3](https://icksw.tistory.com/237)

[Abstract Factory Pattern 1](https://icksw.tistory.com/235)
