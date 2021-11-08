# Protocol과 Delegation

## 211108_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Protocol (1)

프로토콜에 대해 설명할 때 주로 아래의 단어들을 함께 이야기 한다. 

> - 기능/타입의 **청사진** 
> - 기능 요구사항(명세서)

프로토콜은 말 그대로 설계도만 줄 뿐 이를 채택하는 class, struct, enum등이 내부에서 요구사항에 따라 실체화 해줘야 한다. 

프로토콜을 작성하는 문법은 패스! 
(문서를 보면 아주 잘 나와있다)

> 프로토콜도 상속처럼 물론 여러 개 채택할 수 있다. 하지만 각 프로토콜이 요구하는 요구사항을 분리하여 보기 위해서는 `extension`을 활용하여 하나씩 채택하여 실체화하는 것이 보기 좋을 것 같다.

#### 1. 프로퍼티 요구사항

프로토콜은 자기 자신을 채택하는 타입에 대해 특정 이름과 타입을 갖는 인스턴스 프로퍼티나, 타입 프로퍼티를 요구할 수 있다. 프로토콜은 저장 프로퍼티나 연산 프로퍼티를 구체화하지 않고 이름, 타입만을 명시를 해준다. 이 과정에서 프로퍼티가 gettable(외부에서 값을 읽을 수 있는지), settable(외부에서 값을 변경할 수 있는지) 여부를 설정해줄 수 있다. (`{ get }` or `{ get set }`)

```swift
protocol Eatable {
	var calorie: Double { get }
}
```

만약 프로토콜이 어떤 프로퍼티를 gettable하고 settable하게 요구하고 싶어하는 상황이라면, 상수 저장 프로퍼티(변경 불가)나 읽기전용 연산 프로퍼티(변경 불가)로는 이를 충족시킬 수 없다. 만약 프로토콜이 gettable 하나만 요구하는 경우에는 어떤 종류의 프로퍼티로도 요구사항 충족이 가능하며, 만약 현재 코드에서 유용할 것 같다면, settable일 경우도 추가가 가능하다. 

그래서 프로토콜의 프로퍼티는 항상 `var`로 시작하고 gettable인지 settable인지 나타내기 위해 `{ get set }`, `{ get }`으로 표현할 수 있다. 

타입 프로퍼티임을 나타내기 위해 항상`static` 키워드를 추가한다고 한다. 대신 class로 구현되었을 때는 접두사가 `class`
일 수도 있다. 

#### 2. 메서드 요구사항

프로퍼티와 마찬가지로 프로토콜은 인스턴스 메서드와 타입 메서드를 이를 채택하는 타입에게 요구할 수 있다. 메서드를 정의해줄 때에는 `{ }`없이 작성해준다. Variadic parameters도 적어줄 수 있다. 

> Variadic parameters?
> : 0개 혹은 그 이상의 값이 들어올 수 있음을 나타낸다. 
> `func test(target: String...)` 과 같이 `...`
을 붙여준다. 

하지만 메서드의 파라미터에 기본값을 줄 수는 없다. 

```swift
protocol Eatable {
	// Default argument not permitted in a protocol method
	func test(target: String = "123")
}
``` 

앞선 타입 프로퍼티와 마찬가지로 프로토콜 내에 타입 메서드임을 나타내고 싶을 때는 `static` 접두사를 붙여준다. (대신 class로 구현되었을 때는 접두사가 `class`일 수도 있다.)

#### 3. 변경 가능한(mutating) 메서드 요구 사항

메서드는 보유하고 있는 인스턴스를 수정해야 할 필요가 종종 있다. 값 타입(struct, enum)에서의 인스턴스 메서드에는 `func`앞에 `mutating` 키워드를 붙여 인스턴스의 프로퍼티를 수정할 수 있도록 허용해준다. 

> `mutating` 키워드를 프로토콜 내 메서드에 붙였다면 이는 struct와 enum에서만 사용 가능하면 class에서는 사용하지 못한다!

#### 4. 초기화 요구사항

프로토콜은 채택하는 타입에 대해 지정 이니셜라이저를 요구할 수도 있다. 평소에 쓰는 것 처럼 이니셜라이저를 쓰면 되지만 메서드 처럼 `{ }`만 빼두면 된다. 

```swift
protocol Eatable {
	init(food: String)
}
```

클래스에서 프로토콜을 채택 함에 따라 이니셜라이저(지정 이니셜라이저 or 편의 이니셜라이저)를 구현해야 할 때에는 `init( )` 앞에 `required` 수식어를 붙여줘야 한다. 

```swift
class InstantFood: Eatable {
	required init(food: String) {
		// 초기화 내용 작성~
	}
}
```

그렇다면 왜 `required`를 붙여줄까? 

`required` 수식어를 사용하는 이유는 해당 클래스의 모든 하위 클래스에서도 해당 프로토콜을 채택해야하기 때문이다. 

즉 `InstantFood`를 상속받는 `Hamburger`라는 클래스가 있다고 하면, `Hamburger` 또한 `Eatable`프로토콜을 채택해야하기 때문에, 최상위 클래스에서 "필수적인 이니셜라이저"를 추가해주는 것이다. 

만약 어떤 클래스가 상속도 받고 있고, 프로토콜도 채택하고 있다면 `required`와 `override` 키워드를 모두 써줘야 한다.

```swift
class Hamburger: someClass, SomeProtocol {
	required override init() {

	}
}
```

그리고 프로토콜에서 실패 가능한 이니셜라이저를 정의할 수도 있다!

#### 5. 타입으로서의 프로토콜 

사실 프로토콜은 스스로 기능성을 구현할 수 없다. 그럼에도 불구하고, 프로토콜은 독립적인 타입으로서 작동할 수 있다. 타입으로서의 프로토콜은 `existential type`이라고 부르며 "프로토콜을 채택하는 T라는 타입"을 지칭한다. 

프로토콜은 다음과 같은 경우 타입으로서 사용할 수 있다. 

- 함수, 메서드, 이니셜라이저의 파라미터 타입이나 리턴 타입
- 상수, 변수나 프로퍼티 타입
- 배열의 아이템들, 딕셔너리, 다른 컨테이너의 타입

> 프로토콜은 타입이기 때문에 항상 대문자로 시작한다!

#### 6. Delegation 

Delegation은 class나 struct가 다른 타입의 인스턴스에 책임을 넘겨주는 것을 가능케하는 디자인 패턴 중 하나이다. Delegation이라는 말 그대로 어떠한 업무를 위임 받는 행위로 이해하면 될 듯 하다. 그렇기에 delegation은 특정 행동에 반응하도록 하거나, 외부 소스를 알지 못하더라도 외부 소스로부터 데이터를 받아올 수 있게 한다. 

공식 문서와 조금 다른 예제를 살펴보자. 

축구팀을 이끌어나가기 위해서는 선수들도 필요하지만, **감독**과 **코치**의 역할도 중요하다. 

이 때 감독은 총괄 역할을 맡아 시키기만 할 뿐, 코치가 실제로 모든 훈련들을 관리하고, 전략을 수립하고 있다고 해보자. 이런 경우를 `감독`이 `코치`에게 일부 역할을 위임(delegate)했다고 볼 수 있다. 

이를 코드로 봐보자. (이해를 위해 메서드는 간단히 해보자)

```swift
// Director가 채택하게 될 프로토콜
// 전체적인 부분을 감독할 수 있게 directing이라는 메서드 추가 
protocol Directing {
    var name: String { get }
    func directing()
}

// 감독이 directing했을 때 실제로 이루어 질 행위들이다.
// 전략을 짜고, 코칭하고, 관리하고... 이제 이 일들을 위임해볼 것이다. 
protocol GameDirectingDelegate: AnyObject {
    func coaching()
    func managing()
    func makeStrategy()
}

// 감독 객체는 Directing 프로토콜을 채택하여 전체 총괄 지휘를 한다. 
class Director: Directing {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    // 위임을 받아 줄 객체를 우선 옵셔널로 생성해준다.
    weak var delegate: GameDirectingDelegate?
    

    func directing() {
        // 감독이 지시했을 때 위임받은 객체가 해야 할 행위들
        delegate?.makeStrategy()
        delegate?.coaching()
        delegate?.managing()
    }
}

// 감독보다 더 실무에 가까운 코치다.
// 앞선 위임을 받기 위해 GameDirectingDelegate 프로토콜을 채택하고 위임받아야 할 업무들을 구체화한다. 
class Coach: GameDirectingDelegate {
    func coaching() {
        print("이제부터 내가 코칭한다")
    }
    
    func managing() {
        print("이제부터 내가 선수들 관리한다")
    }
    
    func makeStrategy() {
        print("이제부터 내가 전략을 짠다")
    }
}

// Director와 Coach의 인스턴스를 생성하고
let director = Director(name: "바지사장 같은 감독", age: 50)
let primeCoach = Coach()

// 감독이 코치에게 시킨다. 
// Director 타입이 Coach 타입에게 일부 위임한다. 
director.delegate = primeCoach

// 1. 감독이 총괄 지휘를 한다.
// 2. 실상 일은 코치가 다한다. 
director.directing()
// 이제부터 내가 전략을 짠다
// 이제부터 내가 코칭한다
// 이제부터 내가 선수들 관리한다
```

주석을 따라 읽어보면 이해가 될 것이다. 

순서를 정리해보자면 다음과 같이 말할 수 있을 것 같다!

> 1. 시킬 일을 정리한다 (protocol 만들기)
> 2. 일을 시키는 주체가 시킬 사람을 고를 수 있도록 만들어준다. (delegate? 프로퍼티 생성)
> 3. 일을 받는 사람은 1번에서 시킬 일 리스트를 보고 어떻게 일할지 스스로 세부 내용을 정리한다. (1번 protocol 채택 후 내부 구현하기)
> 4. 일을 시킬 사람이 받을 사람에게 위임해준다. (delegate 프로퍼티 주입
> 5. 일을 시킬 사람을 정했으니, 이제 일을 시킨다. (관련 메서드 실행) 


### 자료구조 

자료를 효율적으로 표현할 수 있는 방법론 

배열은 크기가 정해져있다. 
- 인덱싱(검색)은 빠름
- 추가/삭제가 느림

연결 리스트 
- 인덱싱(검색) 느림 (다음 칸은 이전 칸에 정보가 있음)
- 추가/삭제가 빠름
- 메모리 차지 용량도 더 큼 (값과 다음 주소)



## 고민된 점 

- 데이터를 전달할 때의 delegation과 이벤트를 통해 특정 행위를 시키는 delegation 방식이 동일한가?

## 해결 방법 

동일해보인다. delegation 문서 내용을 보면 특정 행위를 시키거나, 데이터를 전달할 때 사용한다는 것을 보니 같은 방법인 것 같다. 
조금 더 살펴보면, 구현 방식이 동일하다는 것을 알 수 있다. 

만약 VC 사이에서 데이터를 전달해주는 경우라면 순서는 다음과 같을 것이다. (sender와 receiver라고 하자)

> 1. sender에서 프로토콜을 생성한다
> 2. sender에서 1번 프로토콜을 타입으로 하는 인스턴스 생성
> 3. sender에서 위임할 일을 2번 인스턴스를 통해 시킨다
> 4. receiver는 1번 프로토콜을 채택하고 내부를 구현한다
> 5. receiver에서는 2번에서 만든 sender의 인스턴스에 본인을 갖다넣어준다
> 6. 실행하면 끝

사실 데이터 전달이나 특정 행위를 시키는 것이나 동일하다고 볼 수 있다. 프로토콜에서 데이터를 전달할 수 있게 만들어주냐 아니면 특정 행위만 시키도록 하는 것인지에 따른 차이가 있는 것 같다. 

아무튼 두 방법에서 이야기하는 delegation이란 같은 방법이라는 것! 그게 내 생각이다. 

---

**Ref**
[Protocol](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html)
[Initialization](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html#ID231)
[Functions](https://docs.swift.org/swift-book/LanguageGuide/Functions.html)
야곰의 재미난 컴퓨터 이야기 2편


