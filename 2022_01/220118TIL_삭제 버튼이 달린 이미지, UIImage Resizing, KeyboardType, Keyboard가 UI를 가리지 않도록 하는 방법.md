
# 삭제 버튼이 달린 이미지, UIImage Resizing, KeyboardType, Keyboard가 UI를 가리지 않도록 하는 방법

## 220118_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. 삭제 버튼이 달린 이미지 만들기 

시중의 앱을 보면, 이미지를 추가한 이후에 사용자가 선택적으로 이미지를 제거할 수 있게 하는 모습을 볼 수 있다. 

구현을 위한 아이디어는 간단하다. 

`UIView`를 하나 만들고, 내부에서 `UIImageView`와 `UIButton`을 사용하여 레이아웃만 잘 잡아주면 원하는 뷰를 만들 수 있다. 

![](https://i.imgur.com/JNJHVwL.png)

예를 들어서 위 모얄 처럼 우측 상단에 삭제 버튼이 달린 뷰를 만든다고 하면 레이아웃은 다음과 같이 구성해볼 수 있다. 

![](https://i.imgur.com/3veBzJT.png)

위 처럼 레이아웃을 잡아주면 마치 버튼이 이미지 위에 걸쳐있는 듯한 느낌을 줄 수 있는 것이다. 

코드를 보자!

```swift
class ProductImageCustomView: UIView {
    override init(frame: CGRect) {
        super.init(frame: frame)
        configUI()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }
    
    // 이미지
    let productImageView: UIImageView = {
        let imageView = UIImageView()
        imageView.contentMode = .scaleAspectFit
        return imageView
    }()
    
    // 삭제 버튼
    private lazy var deleteButton: UIButton = {
        let button = UIButton(frame: CGRect(origin: .zero, size: CGSize(width: 20, height: 20)))
        let icon = UIImage(systemName: "minus.circle.fill")?.resizeImageTo(size: CGSize(width: 20, height: 20))
        button.setImage(icon, for: .normal)
        button.tintColor = .black
        button.backgroundColor = .white
        button.layer.masksToBounds = true
        button.layer.cornerRadius = button.frame.width / 2
        button.addTarget(self, action: #selector(removeProductImageView), for: .touchUpInside)
        return button
    }()
    
    // 이미지 삭제 버튼 클릭시 superview에서 UIView가 제거되도록
    @objc func removeProductImageView() {
        self.removeFromSuperview()
        NotificationCenter.default.post(name: .imageRemoved, object: nil)
    }
    
    // 레이아웃 설정 
    private func configUI() {
        [productImageView, deleteButton].forEach {
            self.addSubview($0)
            $0.translatesAutoresizingMaskIntoConstraints = false
        }
        
        NSLayoutConstraint.activate([
            productImageView.topAnchor.constraint(equalTo: self.topAnchor, constant: 10),
            productImageView.leadingAnchor.constraint(equalTo: self.leadingAnchor, constant: 10),
            productImageView.trailingAnchor.constraint(equalTo: self.trailingAnchor, constant: -10),
            productImageView.bottomAnchor.constraint(equalTo: self.bottomAnchor, constant: -10),
            
            deleteButton.topAnchor.constraint(equalTo: self.topAnchor),
            deleteButton.trailingAnchor.constraint(equalTo: self.trailingAnchor),
            deleteButton.widthAnchor.constraint(equalToConstant: 20),
            deleteButton.heightAnchor.constraint(equalToConstant: 20)
        ])
    }
}
```
크게 설명할 부분이 없을 정도로 `UIView`를 커스텀하게 사용한 것이라 코드를 따라 쉽게 이해해볼 수 있을 것 같다. 

### 2. `UIImage` resizing

`UIImage`가 너무 큰 경우 보통은 auto layout을 통해 너비,높이를 잡아주거나 top,leading,trailing,bottom을 주어서 정해진 마진을 지키면서 이미지가 들어가도록 구현한 경험이 있을 것이다. 

만약 이미지 자체의 크기를 조절하고 싶다면 위 방법으로는 해결이 되지 않을 것이다. 이에 사용할 수 있는 방식이 이미지 리사이징이다. 

이미지 리사이징을 위해 사용되는 방법은 여러가지가 있지만 이번에는 `UIGraphicsBeginImageContextWithOptions(_:_:_:)`를 사용하는 방식을 알아보자. 

간략하게 순서를 먼저 알아보자. 

1. `UIGraphicsBeginImageContextWithOptions(_:_:_:)`로 비트맵 만들기 
2. `UIView.draw()`로 전달받은 사이즈만큼 다시 그려주기 
3. `UIGraphicsGetImageFromCurrentImageContext()
` 메서드를 통해 크기가 조정된 이미지를 받아오기 
4. `UIGraphicsEndImageContext()`를 호출하여 비트맵을 그려주는 환경을 치워주기!

####  1. `UIGraphicsBeginImageContextWithOptions(_:_:_:)`

```swift
func UIGraphicsBeginImageContextWithOptions(_ size: CGSize, 
                                          _ opaque: Bool, 
                                          _ scale: CGFloat)
```

특정 옵션의 비트맵 기반의 그래픽 배경을 만들어주는 역할을 한다. 총 세 가지의 파라미터가 있다. 

- `size`
    - 새로운 비트맵 배경(context)의 크기를 나타낸다. 
    - 이는 `UIGraphicsGetImageFromCurrentImageContext()` 메서드를 호출하여 반환할 이미지의 크기를 나타내기도 한다. 
    - 비트맵의 크기를 픽셀단위로 가져오려면, 이미지의 너비와 높이에 `scale` 파라미터로 오게 될 값을 곱해주면 된다. 
    - 즉 `너비 * 높이 * scale`을 의미하는 것이다. 
- `opaque`
    - 비트맵이 불투명한지 여부를 나타내는 Bool 타입의 값이다. 
    - 비트맵이 완전히 불투명하다는 것을 알고 있다면 값을`true`로 하여 alpha 채널을 무시하고 비트맵의 저장소를 최적화 할 수 있다. 
    - `false`로 할 경우, 비트맵에 부분적으로 투명한 픽셀을 처리할 수 있는 alpha 채널이 포함되어야 한다는 것을 의미한다. 
- `scale`
    - 비트맵에 적용할 축적 비율(scale factor)이다. 
    - 값을 0.0으로 주면 scale factor가 기기의 main screen의 scale factor로 설정된다. 
    - 기기마다 스케일 값은 다른데, 스케일 값은 `UIScreen.main.scale`로도 확인해 볼 수 있다.


이 메서드를 통해 생성된 배경(context)가 현재 배경이며, `UIGraphicsGetImageFromCurrentImageContext()` 메서드를 호출하여 배경의 현재 내용을 기반으로 이미지 객체를 얻을 수 있다. 그리고 배경의 수정을 마치면 무조건 `UIGraphicsEndImageContext()` 메서드를 호출해서 비트맵을 그리는 환경을 정리하고, 배경 스택(context stack)의 맨 위에서 그래픽 배경(graphic context)를 제거해야한다. 스택에서 이러한 타입의 배경을 제거하는데 `UIGraphicsPopContext()`메서드를 사용하면 안된다고 한다. 

만약 다른 그래픽 배경을 가져오고 싶다면 push/pop을 하여 배경을 변경할 수 있다. 또한 `UIGraphicsGetCurrentContext()` 메서드를 사용하여 비트맵 배경을 가져올 수 있다. 

마지막으로 이 메서드의 경우 앱의 모든 쓰레드에서 호출할 수 있다고 하니 참고하자. 

#### 2. `draw(in: )`

```swift
func draw(in rect: CGRect)
```

전체 이미지를 특정한 사각형 내에 그려주어 필요에 따라 크기를 조정해주는 역할을 한다. 

파라미터는 1개로 간단하다.

- `rect`
    - 이미지를 그려줄 사각형(그래픽 context의 좌표계에서의 사각형!)을 의미한다. 

이 메서드는 설정된 이미지의 방향을 고려하여 현재 그래픽 배경에 전체 이미지를 그려준다. 기본 좌표계에서 이미지는 지정된 사각형의 원점 오른쪽 아래쪽에 위치한다. 하지만 이 방법은 현재 그래픽 배경에 적용된 모든 변환들을 고려해준다. 

이 메서드는 `CGBlendMode.normal blend` 모드를 사용하여 이미지를 불투명하게 그려준다. 

이제 이미지를 그렸으니 얻어오기만 하면 된다. 

#### 3. `UIGraphicsGetImageFromCurrentImageContext`

```swift
func UIGraphicsGetImageFromCurrentImageContext() -> UIImage?
```

옵셔널 타입의 `UIImage`를 반환해주는 메서드이다. 이제 크기가 조정된 이미지를 받아줄 수 있는 것이다. 조금 더 자세히 이야기해보면, 현재 비트맵 그래픽 배경의 내용을 포함하는 이미지 객체를 반환해주는 것이다. 

다만 비트맵 기반 배경이 현재 그래픽 배경인 경우에만 이 메서드를 호출해야 한다. 즉, 앞에서 그래픽 배경(context)를 설정해주고 나서 호출해줘야 한다는 것이다. 그렇지 않고 현재 배경이 nil이거나 아직 `UIGraphicsBeginImageContext(_:)` 메서드 호출을 통해 생성되지 않은 경우에 위 메서드는 nil을 반환하게 된다. 

이 메서드 또한 앱의 모든 쓰레드에서 호출될 수 있다! 

`UIGraphicsBeginImageContext(_:)`는 또 뭔가..싶긴한데 아까 1번 과정에서 본 메서드랑 굉장히 유사하게 생긴 것을 알 수 있다. 

실제로도 문서를 보면 `UIGraphicsBeginImageContextWithOptions(_:_:_:)`의 opaque가 false이고, scale은 1.0으로 세팅된 메서드를 호출하는 것과 같다고 한다. 그래서인지 `CGRect`만 받도록 아래와 같이 구현되어 있다.

```swift
func UIGraphicsBeginImageContext(_ size: CGSize)
```

#### 4. `UIGraphicsEndImageContext()`

```swift
func UIGraphicsEndImageContext()
```

스택의 맨 위에서 현재 비트맵 기반 그래픽 배경을 제거하는 역할을 한다. 즉, 다 그리고 이미지를 사용하고나면 제거해주는 것이다. 

이 메서드를 사용하여 `UIGraphicsBeginImageContext(_:)` 메서드로 생성된 환경을 정리하고, 스택 상단에서 해당 비트맵 기반 그래픽 배경을 제거할 수 있다. 현재 배경이 `UIGraphicsBeginImageContext(_:)` 메서드를 통해 생성되지 않은 경우 이 메서드는 아무것도 수행하지 않는다! 

이 메서드 또한 앱의 모든 쓰레드에서 호출될 수 있다. 

#### 5. 구현 코드

설명이 조금 길었지만... 위 내용을 이해했다면 코드는 안봐도 구현할 수 있을 것이다. 

`UIView`를 extension하여 위 메서드들을 사용해서 이미지를 리사이징하는 방법을 구현해보자!

```swift
extension UIImage {
    func resizeImageTo(size: CGSize) -> UIImage? {
        // 비트맵 생성
        UIGraphicsBeginImageContextWithOptions(size, false, 0.0)
        // 비트맵 그래픽 배경에 이미지 다시 그리기
        self.draw(in: CGRect(origin: CGPoint.zero, size: size))
        // 현재 비트맵 그래픽 배경에서 이미지 가져오기
        guard let resizedImage = UIGraphicsGetImageFromCurrentImageContext() else {
            return nil
        }
        // 비트맵 환경 제거 
        UIGraphicsEndImageContext()
        // 크기가 조정된 이미지 반환
        return resizedImage
    }
}
```

그냥 앞선 설명을 그대로 코드로 갖다 넣은 정도이다! 
이제 입맛에 맞게 이미지 크기를 조정해서 사용해 볼 수 있을 것이다.

이미지 크기 자체가 줄어든다는 장점도 있겠지만, 이미지 자체가 줄어들기에 이미지 용량 또한 줄어든다는 것도 장점으로 꼽을 수 있을 것 같다.



### 3. iOS KeyboardType

![](https://miro.medium.com/max/23130/1*v3li23Q8UkrTFFsLXU62Wg.png)

위 이미지를 보다시피 iOS 환경에서는 여러 형태의 키보드를 제공하고 있다. 
`UITextField`, `UITextView`등에서 직접적으로 텍스트를 써야하는 경우가 키보드를 써야하는 대표적인 상황이다. 

`UITextField`나 `UITextView`에 키보드 타입을 설정하는 코드는 간단하다!

```swift
// UITextField
someTextField.keyboardType = .emailAddress
// UITextView
someTextView.keyboardType = .twitter
```

### 4. 키보드가 UI를 가리지 않도록 하는 방법

`UITextField`나 `UITextView`를 탭하는 경우 키보드가 올라오게 될 것이다. 키보드는 대략 뷰의 절반정도 크기로 올라오게 되는데, 이로 인해 `UITextField`나 `UITextView`를 가리게 되는 경우가 생긴다. 

즉, 내가 입력하고 있는 값이 키보드에 가려져서 보이지 않게 되는 것이다. 이를 해결해주기 위해서는 키보드가 띄워질 때 터치된 뷰가 키보드 높이에 대해 반응하여 뷰를 올려주는 행위가 필요하다. 

우선 `UITextField` 혹은 `UITextView`가 속해있는 뷰가 `UIScrollView`라고 가정해보자. 

문제를 해결하기 위한 순서를 먼저 살펴보자. 

1. 키보드가 올라올 때/ 내려갈 때 오는 알림을 받아주기 위해 등록해준다. 
2. 키보드 등장/사라짐에 따라 실행할 `@objc` 메서드를 생성해준다. 

끝이다!

생각보다 간단하게 구현 가능한데, 살펴보자. 

#### 1. NotificationCenter addObserver

```swift
func addKeyboardNotification() {
    NotificationCenter.default.addObserver(self, selector: #selector(keyboardWillShow), name: UIResponder.keyboardWillShowNotification, object: nil)
    NotificationCenter.default.addObserver(self, selector: #selector(keyboardWillHide), name: UIResponder.keyboardWillHideNotification, object: nil)
}
```

앱에서 기본적으로 `keyboardWillShowNotification`와 `keyboardWillHideNotification`을 제공한다. 즉 키보드가 뜨게 되면 `keyboardWillShowNotification`로 notification이 post가 될 것이기에, 이에 대응할 메서드를 만들어주면 되는 것이다. 키보드가 사라지는 경우도 마찬가지이다! 

이 메서드를 ViewController의 `viewDidLoad()`에서 호출을 미리 해주자.

#### 2. 키보드 등장/사라짐에 따른 메서드 구현 

우선 키보드가 떴을 때 실행해 줄 메서드를 먼저 구현해보자. 

```swift
extension UIView {
    var firstResponder: UIView? {
        guard !isFirstResponder else {
            return self
        }

        for subview in subviews {
            if let firstResponder = subview.firstResponder {
                return firstResponder
            }
        }

        return nil
    }
}


@objc func keyboardWillShow(_ sender: Notification) {
    guard let info = sender.userInfo else {
        return
    }
    let userInfo = info as NSDictionary
    guard let keyboardFrame = userInfo.value(forKey: UIResponder.keyboardFrameEndUserInfoKey) as? NSValue else {
        return
    }

    let keyboardRect = keyboardFrame.cgRectValue
    productScrollView.contentInset.bottom = keyboardRect.height

    if let firstResponder = self.view.firstResponder,
       let textView = firstResponder as? UITextView {
        productScrollView.scrollRectToVisible(textView.frame, animated: true)
    }
}
```

우선 firstResponder를 찾기 위해 `UIView`를 extension해준다. 이는 현재로서는 거의 textView를 위한 기능이지만... 우선 구현해보자. 

파라미터에 `Notification`이 위치한 것을 볼 수 있는데, 이는 Notification이 post할 때 키보드에 대한 정보도 담아서 보내주기 때문에 이를 받아주기 위해서 구현해둔 것이다. 

하나씩 살펴보면, 우선 userInfo에서 데이터를 꺼내준다. 그 이후 `keyboardFrameEndUserInfoKey`를 키값으로 키보드 정보를 찾게되는데, 이름을 보다싶이 키보드가 다 뜨고 나서의 frame을 얻을 수 있다. 키보드가 뜨기 시작하거나, 뜨는 중이면 높이가 완전하지 않기 때문에 다 뜨고 나서의 키보드 정보를 얻어오는 것이다. 

그리고는 키보드 프레임의 정보에서 높이까지도 얻어올 수 있다. 

이제 `UIScrollView`의 contentInset의 bottom에 키보드 높이만큼 값을 할당해준다. contentInset은 스크롤 뷰의 모서리로부터 컨텐츠 뷰의 inset을 나타내는 사용자 거리이다. 이 프로퍼티를 사용하여 컨텐츠와 컨텐츠 뷰 가장자리 사이의 공간을 확장할 수 있다. 기본적으로 UIKit은 겹치는 bars를 고려하여 컨텐츠 inset을 자동으로 조정해준다.

textView에 대한 코드의 경우, 먼저 firstResponder가 `UITextView` 타입인지를 보고 scrollView를 이동시켜준다. `scrollToRect` 메서드는 기기에서 보이도록 특정 영역을 스크롤하는 역할을 한다. 

키보드를 내리는 경우는 간단하다. 

```swift
@objc func keyboardWillHide(_ sender: Notification) {
    productScrollView.contentInset = .zero
}
```
그냥 scrollView를 다시 원점으로 되돌려주면 되는 것이다!

## 고민된 점 
- UIImage 크기 조절은 어떻게 할까?
- 키보드에 UI가 가려지지 않게 하려면 어떻게 해야할까?
- 커스텀 UI를 어떻게하면 더 잘 만들 수 있을까?

## 해결 방법 
- 고민 후 직접 구현 및 공식 문서 탐방!!

---

**Ref**

https://betterprogramming.pub/12-shades-of-keyboard-types-in-ios-a413cf93bf4f

https://developer.apple.com/documentation/uikit/uikeyboardtype

https://developer.apple.com/documentation/uikit/1623922-uigraphicsbeginimagecontext

https://developer.apple.com/documentation/uikit/1623912-uigraphicsbeginimagecontextwitho

https://developer.apple.com/documentation/uikit/1623924-uigraphicsgetimagefromcurrentima

https://developer.apple.com/documentation/uikit/1623933-uigraphicsendimagecontext

https://seizze.github.io/2019/11/17/iOS%EC%97%90%EC%84%9C-%ED%82%A4%EB%B3%B4%EB%93%9C%EC%97%90-%EB%8F%99%EC%A0%81%EC%9D%B8-%EC%8A%A4%ED%81%AC%EB%A1%A4%EB%B7%B0-%EB%A7%8C%EB%93%A4%EA%B8%B0.html
