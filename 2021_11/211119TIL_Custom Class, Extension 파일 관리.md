# Custom Class, Extension 파일 관리

## 211119_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Custom Class 

`ViewController` 내에서 `UILabel`, `UIButton` 등 `UIView`의 요소를 생성해서 사용하는 경우가 있다. 

이 때 보통은 기본 타입을 활용해서 생성할 것 이다, 
`UILabel`을 예시로 보자! 

```swift
let customLabel = UILabel()
```

이제 해당 label에 속성을 줘야할 필요가 있을 경우 보통 아래와 같이 줄 수 있다. 

```swift
customLabel.text = "안녕하세요"
customLabel.textColor = .white
customLabel.font = UIFont(name: "Helvetica", size: 24)
```

`UILabel`이 하나인 경우 이렇게 기본 `UILabel`를 활용하여 속성값들을 줄 수 있지만 만약 `UILabel`이 많아지는 경우 어떻게 관리해야할까? 

```swift
let customLabel = UILabel()
let nameLabel = UILabel()
let priceLabel = UILabel()
let descriptionLabel = UILabel()
```

위와 같은 상황에서 모든 텍스트 컬러를 흰색으로 주려고 한다면 모든 텍스트에 일일이 텍스트 컬러를 줘야할까..?

```swift
// 하나하나 줘야하나...
customLabel.textColor = .white
nameLabel.textColor = .white
priceLabel.textColor = .white
descriptionLabel.textColor = .white
```

조금 더 개선해본다면 반복문, 고차함수를 통해 반복을 줄여볼 수는 있을 것이다. 

```swift
[customLabel, nameLabel, priceLabel, descriptionLabel].forEach {
	$0.textColor = .white
}
```

하지만 이런 방법도 일시적인 방법으로 보인다. 계속 `UILabel`이 추가되면 저 배열에 추가를 해줘야하는 수고로움이 있다. 

이렇게 반복되는, 공통되는 속성이 있다면 커스텀 클래스를 만들어 해결해볼 수 있다. 

```swift
class CustomLabel: UILabel {
    override init(frame: CGRect) {
        super.init(frame: frame)
    }
    
    init(text: String) {
        super.init(frame: CGRect.zero)
        self.text = text
        self.textColor = .white        
        self.font = UIFont(name: "Helvetica", size: 24)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

이렇게 커스텀 클래스를 만들고 활용하게 되면 불필요한 반복을 줄일 수 있고 통일성을 줄 수 있게 된다! 

```swift 
let prodNameLabel = CustomLabel(text: "맥북")
let priceLabel = CustomLabel(text: "2,000,000")
let sellerNameLabel = CustomLabel(text: "Apple")
```

이렇게 인스턴스를 생성하기만 하더라도 속성이 기본적으로 부여되면서, 위의 복잡한 과정을 쉽게 풀어볼 수 있다! 

## 고민된 점 
1. `extension` 파일 분리는 어떻게 하고? 네이밍은 어떻게 할까?

## 해결 방법 

> 1. `extensions` 폴더/파일 관리 및 네이밍 
 
`extension`을 파일로 분리해서 작성하는 경우 보통 `Extension`이라는 폴더 안에 파일을 따로 생성해서 관리를 한다. 이 때 파일 네이밍도 중요해질 수 있는데, 그냥 `extension`하는 타입을 적는 경우도 있지만 기능에 의거하여 네이밍을 해도 좋을 것 같다. 

예를 들어서 `extention String`내에서 어떠한 문자열을 분리하는 기능을 한다면 해당 파일명은 `String+split`이 될 수 있다. 기능을 네이밍에 담게 되면 추후에 추적하기도 편하고, 관리하기도 편할 것 같다! (네이밍의 이유도 생기는 느낌!)

---

**Ref**

[UI Custom](https://ios-development.tistory.com/43)

