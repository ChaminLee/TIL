# Core Graphics, OS, BeautyAlert Pod 배포

## 220210_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용
https://developer.apple.com/documentation/uikit/uiviewcontroller/1621432-showdetailviewcontroller

### 1. Core Graphics

커스텀 뷰란 무엇일까 이전에 생각했을 때에는 여러 UI 요소를 입맛에 맞게 배치하여 사용하는 것이라고 생각했었다. 

하지만 찐 커스텀뷰란 직접 어떠한 모양을 그리는 것이라고 한다. A점에서 B점까지 선을 그리거나, 원을 그려주는 등의 행위를 말하는 것 같다. 

이를 이전에 드로잉 사이클에서 봤던 `draw(_ :)`메서드를 재정의하여 구현해보자! 

> `draw(_ :)`는 필요한 경우에만 오버라이드 해야한다. 안쓰는데 재정의하는 것은 오히려 역효과가 생길 수 있다고 한다. 

시작하기 전에 앞서서 attributes들에 대해 간략하게 짚어보고 넘어가자. 

- `@IBDesignable`: 코드를 통해 구성된 UI요소가 스토리보드에 계속 동기화 되도록 한다. 
- `@IBInspectable`: 특정 요소의 값을 스토리보드의 UI에 나타내서 설정할 수 있도록 해준다. 

자 그러면 간단하게 `+` 버튼을 만들어서 `CGAffineTransform`을 이용하여 회전시키는 것 까지 살펴보자. 

우선 버튼을 그려보자.

```swift
@IBDesignable
class PlusButton: UIButton {
    override func draw(_ rect: CGRect) {
        guard let context = UIGraphicsGetCurrentContext() else {
            return
        }
        
        let height = bounds.height
        let width = bounds.width
        
        let circleRect = bounds.insetBy(dx: width * 0.05, dy: height * 0.05)
        
        // 원 그리기
        context.beginPath()
        context.setLineWidth(10)
        context.setFillColor(UIColor.green.cgColor)
        context.setStrokeColor(UIColor.black.cgColor)
        context.addEllipse(in: circleRect)
        context.drawPath(using: .fillStroke)
        context.closePath()
        
        // + 모양 그리기 
        context.beginPath()
        context.setLineCap(.round)
        context.addLines(between: [CGPoint(x: width / 2, y: height * 0.2), CGPoint(x: width / 2, y: height * 0.8)])
        context.addLines(between: [CGPoint(x: width * 0.2, y: height / 2), CGPoint(x: width * 0.8, y: height / 2)])
        context.drawPath(using: .stroke)
        context.closePath()
    }
}
```

그리기 시작과 끝에는 `beginPath()`와 `closePath()`를 호출해줘야 한다. 

그리고 원을 그릴 때는 `addElipse()`를 사용하여 `CGRect` 값을 주면 해당 범위 내에 그릴 수 있다. 

선을 그릴 때에는 `addLine()`을 사용하거나, `addLines()`를 통해 두 지점을 한 번에 줄 수도 있다. 

아무튼 선이나 원 등의 요소를 추가해주고 난 뒤, `drawPath()`를 호출하여 외곽선만 그릴 것인지, 내부를 채울 것인지 등 옵션을 주어 그릴 수 있게 된다! 

> `setLineCap()`의 옵션을 통해 선의 끝을 둥글게 할 지, 각지게 만들지 정할 수 있다. 
> `setLineJoin()`은 선이 겹치는 경우에 둥글게 할지, 각지게 만들지 정할 수 있다. 

`UIButton`을 회전시키는 것은 간단하다. 
`CGAffineTransform(rotateAngle: )`을 활용하여 애니메이션과 함께 돌려주기만 하면 된다. 

```swift
// 45도 만큼 회전 
UIView.animate(withDuration: 0.2) {
    self.plusButton.transform = CGAffineTransform(rotationAngle: .pi / 4)
}

// 원상복귀
UIView.animate(withDuration: 0.2) {
    self.plusButton.transform = .identity
}
```

`.identity`를 쓰면 다시 원래 모양대로 돌아올 수 있다. 

결과물을 보자!

![](https://i.imgur.com/QjrvZsi.gif)

당근마켓이나 토스 등 앱에서 보면 버튼 터치할 때에 버튼 자체가 돌아가거나, 유사한 애니메이션이 있는데 이를 활용해서 구현해볼 수 있을 것 같다! 
 
### 2. OS

매주 2강씩 들은지 10주차가 되어간다...! 이렇게 주에 조금이나마 시간을 내어 듣다보니 어느새 완강까지 3주 정도 남은 것 같다. 

면접을 대비하기 위함도 있지만 운영 체제가 어떻게 동작하는지 살펴보면 iOS 개발하는데 있어서도 좋은 양분이 될 것이라고 생각한다! 

오늘은 memory management의 방법에 대한 강의를 들었다! 

[Memory Management 3](https://github.com/ChaminLee/CS_Study/blob/main/OS/20.%20Memory%20Management%203.md)

### 3. BeautyAlert Pod 배포

재미삼아 만들었던 BeautyAlert를 배포하였다! 

![](https://i.imgur.com/vmfzZZd.png)

기능을 더 추가해보려다가, 시간적 여유가 되지 않아 이 정도에서만 마무리하고, 다음번에 조금 더 완성도 있는 라이브러리를 구현해보면 좋겠다는 생각을 했다. 

[BeautyAlert](https://github.com/ChaminLee/BeautyAlert)

나라도 편리하게 사용하면 좋을 것 같다! 
## 고민된 점 
- 진정한 커스텀뷰란?
- UI요소를 회전시키는 방법

## 해결 방법 
- Core Graphics를 활용한 그리기
- CGAffineTransform을 활용한 회전 
---

**Ref**

야곰의 활동학습 

https://github.com/ChaminLee/CS_Study/blob/main/OS/20.%20Memory%20Management%203.md
