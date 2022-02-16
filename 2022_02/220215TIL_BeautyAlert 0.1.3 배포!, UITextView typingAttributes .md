# BeautyAlert 0.1.3 배포!, UITextView typingAttributes 

## 220215_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. BeautyAlert 0.1.3 배포!

#### iPad 지원 

iPad에서도 alert를 쓸 수 있게 alert의 크기를 스크린에 상대적인 비율을 정해두어 기기마다 다른 크기로 보여지도록 구현했다. 

```swift
private enum UserDevice {
    static let deviceIdiom = UIScreen.main.traitCollection.userInterfaceIdiom
}
```

위 코드를 통해 device의 종류를 구하고, 아래와 같이 분기처리하여 값을 다르게 주었다. 

```swift
var widthRatio: Double = 0.0
        
switch UserDevice.deviceIdiom {
case .phone:
    widthRatio = 0.8
case .pad:
    widthRatio = 0.3
default:
    break
}
```

### 2. UITextView typingAttributes

메모 앱 같이 textView 내에서 첫 번째 줄은 타이틀, 그 다음은 내용으로 인식이 되는데, 폰트가 다르게 적용된 것을 볼 수 있다. 

이는 UITextViewDelegate의 메서드인`shouldChangeTextIn()` 를 통해 구현해볼 수 있다. 

```swift
func textView(_ textView: UITextView, shouldChangeTextIn range: NSRange, replacementText text: String) -> Bool {
    let textAsNSString = textView.text as NSString
    let replacedString = textAsNSString.replacingCharacters(in: range, with: text) as NSString
    let titleRange = replacedString.range(of: .lineBreak)

    if titleRange.location > range.location {
        textView.typingAttributes = TextAttribute.title
    } else {
        textView.typingAttributes = TextAttribute.body
    }

    return true
}
```
 
 우선 textView의 텍스트를 꺼내서 `NSString`으로 캐스팅 해준다. 
그리고 `replacingCharacters()`을 통해 범위 내의 텍스트를 주어진 텍스트로 변경해준다. 사실 변경이라기 보다는 `shouldChangeTextIn()`에서 추출하는 textView의 text는 현재 입력하기 전 텍스트를 보여주기에 미리 반영해두는 것 뿐이다!

이제 이 텍스트를 가지고, `range()` 메서드를 통해 문자열 내에서 지정된 문자열(줄바꿈)이 처음 나타나는 범위를 찾아 반환해준다. 

이 범위가 이제 제목을 나타내게 될 것 이다. 

줄바꿈을 하지 않고 계속 입력하게 되면 줄바꿈을 찾지 못하기에 다음과 같은 (titleRange.location, range.location)을 구할 수 있다. 

![](https://i.imgur.com/iflqkXz.png)

줄바꿈을 하기 전까지는 titleRange.location가 range.location보다 크기 때문에 로직에 따라 제목으로 인식되는 것이다. 

예를 들어서 7자리를 첫 줄에 입력하고 줄바꿈하게 되면 titleRange.location은 7이 된다. 

![](https://i.imgur.com/TpJmmCc.png)

딱 줄바꿈을 하면 동일해지는 순간이 오며, 그 다음에 입력하는 순간 range.location이 더 커지기 때문에 내용으로 인식될 수 있는 것이다. 

추가적으로 typingAttributes를 활용했기 때문에 사용자가 입력하는 새 텍스트에 적용할 속성을 부여할 수 있었다!



## 고민된 점 
- 각 기기별로 기능을 지원하기 위해 고려해야할 점 
- textView에 입력할 때 제목과 내용을 폰트로 구분하는 방법

## 해결 방법 
- 기기 마다 비율을 다르게 설정하거나, 분기처리하여 다른 로직을 사용하도록 구현하면 될 것 같다!
- range의 location과 typingAttributes 활용!
---

**Ref**

https://hartl.co/2015/05/21/Highlight-first-line.html
