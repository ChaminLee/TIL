
# 커스텀 UIStackView, OpenSource Library Day 3

## 220125_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 커스텀 UIStackView 

> 0. `LayoutMargin`

stackView 내에 마진을 두는 역할을 해준다. 
```swift
stackView.isLayoutMarginsRelativeArrangement = true

stackView.layoutMargins = UIEdgeInsets(top: 0, left: 15, bottom: 0, right: 15)
```

우선 stackView는 arranged된 subview를 관리하기 때문에, 하위 뷰의 위치를 조정할 때 layoutMargin을 따를 것인지를 나타내는 `isLayoutMarginsRelativeArrangement` 프로퍼티에 true의 값을 준다. 

그 이후 `.layoutMargin`에 `UIEdgeInsets()`값을 넣어 마진을 주면 된다!

> 1. `semanticContentAttribute`

stackView에 쌓이는 순서(방향)을 정해주는 역할을 한다. 

기본적으로 horizontal stackview에 새로운 하위 뷰들을 추가해주게 되면 `왼쪽 -> 오른쪽`의 방향으로 추가된다. 

이 방향을 바꿔주고 싶을 경우 위 프로퍼티를 사용해주면 된다. 

```swift
stackView.semanticContentAttribute = .forceLeftToRight
```

`forceLeftToRight`는 `왼쪽 -> 오른쪽`이고, `forceRightToLeft`는 `오른쪽 -> 왼쪽`순으로 추가된다. 

> 지금은 stackView지만, UIButton에 이미지와 텍스트의 순서를 바꾸고 싶은 경우에도 위 프로퍼티를 사용하면 된다고 한다!

> 2. `setCustomSpacing()`

```swift
func setCustomSpacing(_ spacing: CGFloat, after arrangedSubview: UIView)
```

after에 전달되는 view이후로부터 spacing 만큼의 마진을 줄 수 있다. 기존의 UIStackView의 `.spacing`이 있더라도 위 속성이 우선적으로 적용된다. 

```swift
let stackView = UIStackView()
let label1 = UILabel()
let label2 = UILabel()
let label3 = UILabel()
[label1, label2, label3].forEach {
    stackView.addArrangedSubview($0)
}

stackView.setCustomSpacing(20, after: label2)
```

위처럼 하면 label2이후 부터는 마진이 20이 된다!

### OpenSource Library Day 3

시간을 정말 짬내서 하고 있다... (너무 조금씩 발전하는 중...)

오늘은 뷰 계층을 다시 좀 점검하고 alert에 message를 넣어줄 수 있도록 기능을 추가하였다. 

진행현황을 대략 파악해 볼 수 있도록 현재 지원하는 두 타입의 alert를 첨부한다!

|cancel/confirm button|confirm button|
|:---:|:---:|
|![](https://i.imgur.com/uA4tYRy.png)|![](https://i.imgur.com/7Pin5Fn.png)|



현재까지 커스텀 가능한 부분은 아래와 같다!

- Alert
	- Title
		- titleText
		- textColor
	- Message
		- messageText
		- textColor
	- backgroundColor
- AlertAction
	- cancel	
		- titleText
		- titleColor	
		- backgroundColor
	- confirm
		- titleText
		- titleColor
		- backgroundColor

우선 색상, 문자를 입맛에 맞게 사용할 수 있도록 하는데 초점을 맞췄다!

## 고민된 점 
- StackView는 만능인가...
- Alert를 어떻게 하면 더 커스텀하고 이쁘게 쓸 수 있을까...

## 해결 방법 
- StackView의 프로퍼티들 찾아보기!
- 여러 디자인 레퍼런스 찾아보기
---

**Ref**

https://developer.apple.com/documentation/uikit/uiview/1622566-layoutmargins

https://developer.apple.com/documentation/uikit/uiview/1622461-semanticcontentattribute

https://developer.apple.com/documentation/uikit/uistackview/2866023-setcustomspacing
