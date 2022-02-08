
# DateFormatter, ScrollView contentOffset VS setContentOffset, preferredSplitBehavior

## 220208_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. DateFormatter

DateFormatter의 인스턴스는 NSDate 객체를 문자열로 나타내기도 하고, 날짜 및 시간의 문자열을 NSDate 객체로 변환하기도 한다. 

사용자들에게 날짜를 보여주기 위해 `dateStyle`과 `timeStyle` 프로퍼티를 필요에 따라 사용할 수 있다. 

만약 날짜만 보여주고 싶다면 timeStyle을 `.none`으로 두고, 시간만 보여주고 싶을 경우 dateStyle을 `.none`으로 두는 등의 방식으로 구현할 수 있다. 

또한 각각 short/medium/long의 프로퍼티도 존재하는데, 날짜/시간을 길게 늘여서 표현할 것인지 짧고 간결하게 나타낼 것인지를 나타내준다. 

```swift
let dateFormatter = DateFormatter()
dateFormatter.dateStyle = .medium
dateFormatter.timeStyle = .none
 
let date = Date(timeIntervalSinceReferenceDate: 118800)
 
// US English Locale (en_US)
dateFormatter.locale = Locale(identifier: "en_US")
print(dateFormatter.string(from: date)) // Jan 2, 2001
 
// French Locale (fr_FR)
dateFormatter.locale = Locale(identifier: "fr_FR")
print(dateFormatter.string(from: date)) // 2 janv. 2001
 
// Japanese Locale (ja_JP)
dateFormatter.locale = Locale(identifier: "ja_JP")
print(dateFormatter.string(from: date)) // 2001/01/02
```

이런식으로 language code와 country code를 조합한 identifier를 넣어서 각 지역/언어에 맞게 날짜/시간을 나타낼 수 있다. 

`_`를 기준으로 좌측이 language code, 우측이 country code이다. 

만약 현재 기기의 설정을 기반에 따라 변경하고자 한다면 다음의 코드를 추가해주면 된다. 

```swift
dateFormatter.locale = Locale.current
```

위처럼 TimeInterval 값을 날짜로 변환하기 위해 `Date(timeIntervalSinceReferenceDate: )`를 사용한다. 

이 메서드의 경우 2001년 1월 1일 00:00:00 UTC를 기준으로 한 date 차이를 알려준다. 즉 값이 너무 큰 경우 현재가 아닌 미래(?)가 될 수도 있다는 것이다. 

만약 미래가 나왔다면... 더 이전의 기준값을 가지고 비교할 필요가 있다. 

`Date(timeIntervalSince1970: )`를 이용하면 1970년 1월 1일 00:00:00 UTC를 기준으로 할 수 있다!

위 두 메서드 모두 숫자가 마이너스인 경우는 기준일자보다 앞선 일자를 나타낸다고 한다. 

아무튼 고려해서 잘 써줘야 할 것 같다! (어디선가 Int만 떨어진다면 어떤 기준 date를 통해서 얻어온 것인지 사전 확인이 필요할 것 같다..)

### 2. ScrollView contentOffset, setContentOffset

지성이 질문해주셔서 들여다보게 되었다. 

scrollview는 내부 요소들을 담는 content view와 scroll view 자체를 가지고 있다. 

content view가 scrollview보다 커지게 되면, 그 때 스크롤이 가능해지는 것이다. 이 때 content의 offset을 지정할 수도 있는데, 대표적인 방법이 `contentOffset` 인스턴스 프로퍼티에 CGPoint 값을 넣거나, `setContentOffset(CGPoint, animate: )`를 통해 지정하는 것이다. 

#### `contentOffset` 

Content view의 원점이 scroll view의 원점에서부터 가지는 간격(offset)을 의미한다. 

```swift
var contentOffset: CGPoint { get set }
```

간단하게 보자면, 액자와 액자 속 사진의 마진인 것이다. 

![](https://i.imgur.com/dtrbQ9l.png)

scrollView가 액자, contentview가 사진인 것이다. 

#### `setContentOffset()` 

해당 메서드의 경우도 마찬가지이다. 

receiver의 원점에 대응되는 contentView의 원점이 가지는 offset를 설정하는 역할을 한다. 

문서에서 말하는 receiver란, contentView를 담고 있는 틀을 말하는 것 같다. 즉 scrollView가 될 수도 있고, contentview를 가지는 UITableViewCell이 될 수도 있을 것 같다. 

```swift
func setContentOffset(_ contentOffset: CGPoint, 
             animated: Bool)
```

해당 메서드의 경우는 offset 조정 뿐만 아니라, animation도 지원해준다. 

content view의 offset이 조정될 때 자연스럽게 스르륵 올라가도록 만들어주고 싶다면 해당 옵션을 true로 지정해주면 된다! 

예를 들어서 `UINavigationbar`를 가지는 view에 긴 텍스트를 가지는 `UITextView`를 올리게되면 content view의 offset이 초기에 (0,0)이 아닌 경우가 생긴다. 

![](https://i.imgur.com/qWZMu5C.png)

이 경우는 UITextView를 view의 safeAreaLayoutGuide의 상하좌우에 맞춰준 결과이다. 

레이아웃을 잡았음에도, content가 초기에 일부 가려지는 현상이 있다. 

이 때 초기에 contentOffset의 값을 확인해보면 `(0.0, 91.0)`이고, frame의 x,y 또한 `(0.0, 91.0)`이다. 

이에 추후에도 유의해야 할 점은 텍스트가 길어지면 contentOffset이 기대한대로 원점에 있지 않은 경우도 발생할 수 있다는 것이다!

그렇기에 시작하는 경우에는 contentView가 scrollView를 기준으로 offset이 (0.0, 0.0)이어야 한다. 이를 위해 contentOffset에 `.zero`를 주어서 초기 contentView의 offset을 (0.0, 0.0)으로 잡아줄 수 있다. 

![](https://i.imgur.com/mzRgmuP.png)

이 때 contentOffset의 값은 (0.0, 0.0)으로 원하는대로 보여지고 있다.

다만 특이한 점은 초기에 세팅을 위해 `setContentOffset()` 메서드를 사용하면 즉시 반영이 되지 않는 경우가 있다. 

왜인지 추측해보면,  `setContentOffset()` 메서드의 경우 다음 드로잉 사이클에 layout을 잡도록 예약해주는 것 같다는 생각이 든다. 

단순히 `contentOffset`의 값만 지정해주는 경우 현재 드로잉 사이클에 포함될 수 있으나,  `setContentOffset()` 같이 "offset을 조정해줘!" 같은 메서드는 다음 드로잉 사이클에 업데이트 될 것을 기본적으로 가정하는 것 같다. 

이에 즉시 레이아웃이 업데이트 되도록 하는 `layoutIfNeeded()`를 호출해주면 정상적으로 바로 동작하는 것을 볼 수 있다. 

정리해보자면 `contentOffset`를 사용하면 현재 사이클에 바로 적용이 되는 것이고! `setContentOffset()` 메서드를 쓴다면 드로잉 사이클을 고려해서 업데이트를 예약할 것인지, 즉시 반영하도록 해줄 것인지 고민하고 사용해야 할 것 같다!

### 3. preferredSplitBehavior

UISplitViewController를 portrait 모드에서 볼 때, 좌측 상단의 displayModeButton을 터치하면 primary view가 secondary view를 가리게 된다. 

이를 방지하고 `.oneBesideSecondary` 처럼 움직이게 해주려면 `preferredSplitBehavior`를 `.tile`로 세팅해주면 된다. 

`.tile`은 사이드바와 secondary view가 나란히 놓일 수 있도록 해주는 기능을 가지고 있다. 이에 primary view가 secondary view를 가리지 않고 보여지도록 구현할 수 있다!

## 고민된 점 
-  `contentOffset`과 `setContentOffset()`의 동작 차이 


## 해결 방법 
- LLDB를 통한 디버깅 및 뷰 드로잉 사이클과 연관지어 생각해보기!
---

**Ref**

https://developer.apple.com/documentation/foundation/dateformatter

https://developer.apple.com/documentation/foundation/nslocale/1643026-languagecode

https://developer.apple.com/documentation/foundation/nslocale/1643060-countrycode

https://developer.apple.com/documentation/uikit/uiscrollview/1619404-contentoffset

https://developer.apple.com/documentation/uikit/uiscrollview/1619400-setcontentoffset
