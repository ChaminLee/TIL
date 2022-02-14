# Core Animation, Pull-Down button, UITableView Swipe Actions, BeautyAlert Todo

## 220214_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### !복습!

- 101 & 010 = 000
	- 두 개의 값이 1로 같거나 0으로 같아야 1이 나온다! 
- escaping closure임을 나타내는 방법
	- `(() ->())?` : 옵셔널 표기하면 기본적으로 escaping이라고 한다
	- `@escaping` attribute 속성 사용

### 1. Core Animation 

- frame
	- 상위 뷰의 좌표 시스템안에서 view의 위치와 크기를 나타낸다.
- bounds
	- 자신만의 좌표시스템 안에서 view의 위치와 크기를 나타낸다. 

frame을 변경하는 경우 액자+사진이 한 번에 움직여서 보여지는 컨텐츠에는 문제가 없다. 하지만 bounds의 좌표를 변경하는 경우, 액자 속 사진을 이동시키는 것이기에 컨텐츠의 보여지는 영역이 변경된다. 

https://stackoverflow.com/questions/1210047/cocoa-whats-the-difference-between-the-frame-and-the-bounds/28917673#28917673

Core Animation은 CALayer의 프로퍼티들을 직접 수정하지 않고 두 개의 layer tree를 관리한다. 

> Layer는 앱의 view를 대체하는 것은 아니다. 즉, layer 객체만을 기반으로 시각적인 인터페이스를 만들 수 없다. Layer는 view에 인프라를 제공하는 역할을 하는 것이다. 좀 더 구체적으로 말하자면, layer를 사용하면 view의 내용을 더 쉽고 효율적으로 그리고, 애니메이션을 적용할 수 있고, 작동 중에 높은 프레임률을 유지할 수 있다. 
>  하지만 layer는 하지 못하는게 많다. layer는 이벤트를 처리하거나, 컨텐츠를 그리거나, responder chain에 참여하거나, 다른 많은 작업을 수행하지 못한다. 이러한 이유로 모든 앱은 이러한 종류의 상호 작용을 처리하기 위해 하나 이상의 view를 가지고 있어야 한다! 

- model layer tree
	- `CATransaction`이 일어난 후 최종적인 layer 정보
- presentation layer tree
	- 현재 화면에 보여지는 layer

예시 중, 카드가 전환되는 애니메이션을 구현해보자. 

<img src="https://i.imgur.com/4v01Gw5.gif" width = 50%>

파란색 뷰의 뒤에 있던 초록색 뷰가 앞으로 나오는 애니메이션이다. 
이 동작을 구현하기 위해서 필요한 요소는 다음과 같다. 

- zPosition
	- 파란색, 초록색 뷰의 계층 순서
- position
	- 각각 양측으로 이동
- transform.rotation
	- 주어진 각만큼 rotate

파란색 뷰에 대한 애니메이션 코드를 먼저 보자.

```swift
func blueViewAnimation(completion: () -> ()) {
    let zPosition = CABasicAnimation()
    zPosition.keyPath = "zPosition"
    zPosition.fromValue = 1
    zPosition.toValue = -1
    zPosition.duration = 1.2

    let rotation = CAKeyframeAnimation()
    rotation.keyPath = "transform.rotation"
    rotation.values = [0, -0.03, 0]
    rotation.duration = 1.2
    rotation.timingFunctions = [CAMediaTimingFunction(name: .easeInEaseOut), CAMediaTimingFunction(name: .easeInEaseOut)]

    let position = CAKeyframeAnimation()
    position.keyPath = "position"
    position.values = [NSValue(cgPoint: .zero), NSValue(cgPoint: CGPoint(x: -110, y: -20)), NSValue(cgPoint: .zero)]
    position.timingFunctions = [CAMediaTimingFunction(name: .easeInEaseOut), CAMediaTimingFunction(name: .easeInEaseOut)]
    position.isAdditive = true
    position.duration = 1.2

    let group = CAAnimationGroup()
    group.animations = [zPosition, rotation, position]
    group.duration = 1.2

    blueView.layer.add(group, forKey: nil)

    completion()
}
```

간단한 애니메이션이라면 `CABasicAnimation()`을 이용해서 값만 바꿔주고, 일련의 값을 필요로 한다면 `CAKeyframeAnimation()`을 사용하여 값이 변해갈 과정을 수치로 나타내주면 된다. 

우선 파란색 뷰의 경우 z축으로 봤을 때, 앞에 있다가 뒤로 가야하기 때문에 1 -> -1로 변경하도록 설정해준다. rotation의 경우 조금만 비틀어주기 위해 값을 주고 다시 복귀 해야하기 때문에 앞뒤로 0을 붙여준다. 애니메이션은 자연스럽게 표시하기 위해 easeInEaseOut을 사용한다. 

position의 경우 파란색 뷰는 좌측으로 조금 이동해야 하기 때문에 NSValue와 CGPoint의 값을 활용하여 좌측으로 잠시 이동하도록 해준다. 

`isAdditive`의 경우 현재 위치를 기준으로 상대적으로 애니메이션을 사용할 것인지 여부를 나타낸다. `true`를 하면 상대적인 위치를 기준으로 한다! 문서를 보면 true로 지정시, 애니메이션으로 지정한 값이 속성의 현재 render tree 값에 추가되어, 새 render tree 값을 생성한다고 한다. 

> render tree 내의 객체들은 실제로 애니메이션을 수행하는 Core Animation의 전용 객체이다. 

대략 이해해보자면, 애니메이션이 수행되던 트리에 추가되어 현재 트리에 상대적으로... 그러니까 맥락이 이어진다는 것이니, 현재 위치에서 애니메이션이 적용된다고 봐도 될 것 같다. `false`로 주게 되면, 현재 위치에서 만큼 이동하는 것이 아니라, 원치 않은 위치에서 이동하는 모습을 보게 될 수 있다. 

초록색 뷰의 애니메이션 코드도 비슷하다. 

```swift
func greenViewAnimation() {
    let zPosition = CABasicAnimation()
    zPosition.keyPath = "zPosition"
    zPosition.fromValue = -1
    zPosition.toValue = 1
    zPosition.duration = 1.2

    let rotation = CAKeyframeAnimation()
    rotation.keyPath = "transform.rotation"
    rotation.values = [0, 0.14, 0]
    rotation.duration = 1.2
    rotation.isAdditive = true
    rotation.timingFunctions = [CAMediaTimingFunction(name: .easeInEaseOut), CAMediaTimingFunction(name: .easeInEaseOut)]

    let position = CAKeyframeAnimation()
    position.keyPath = "position"
    position.values = [NSValue(cgPoint: .zero), NSValue(cgPoint: CGPoint(x: 110, y: -20)), NSValue(cgPoint: .zero)]
    position.timingFunctions = [CAMediaTimingFunction(name: .easeInEaseOut), CAMediaTimingFunction(name: .easeInEaseOut)]
    position.isAdditive = true
    position.duration = 1.2

    let group = CAAnimationGroup()
    group.animations = [zPosition, rotation, position]
    group.duration = 1.2

    greenView.layer.add(group, forKey: nil)
}
```

애니메이션을 추가하는 `add(_, forKey: )`에서 Key는 문자열로, 아무거나 넣어도 된다! 옵셔널 타입이기에 nil을 넣어도 작동한다.

#### Layer coordinate System

다시 Core Animation Programming guide 문서로 돌아와서 좌표계를 살펴보자. 

layer는 point-기반의 좌표계, unit 좌표계 두 가지를 사용한다고 한다. 

기본적인 layer의 기하학적인(position 기반) 좌표계는 다음과 같다. 

![](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/layer_coords_bounds_2x.png)

중앙이 `position`이다. `position`이란 상위 레이어의 좌표 공간에서의 layer의 위치이다. `position` 값은 포인트 단위로 지정되며 항상 `anchorPoint` 프로퍼티의 값을 기준으로 지정된다. 독립적인 layer의 기본 `position`은 (0.0, 0.0)이다. frame 프로퍼티를 변경하면 `position`의 값 또한 변경된다. 

그렇다면 `anchorPoint`는 무엇인가...? `anchorPoint`는 layer의 bounds 직사각형의 anchorpoint를 의미한다. 이 `anchorPoint`는 unit 좌표 공간을 사용하여 표시된다. 

아래 그림은 unit 좌표 시스템이다. 
![](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/layer_coords_unit_2x.png)

기본 값은 (0.5, 0.5)로 layer의 bounds 직사각형의 중앙이다. 뷰에 대한 모든 기하하적 조작은 지정된 점에 대해 발생한다. 예를 들어 회전 변형을 기본 anchorPoint와 함께layer에 주려고 하면, layer는 anchorPoint를 중심으로 회전한다. anchorPoint를 다른 위치로 변경하면 layer가 새로운 지점을 중심으로 회전하게 된다. 

![](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreAnimation_guide/Art/layer_coords_anchorpoint_transform_2x.png)

> `isRemoveOnCompletion = false`를 설정하면 애니메이션의 목적지 위치를 변경하지 않아도 되어서 자동으로 업데이트 됨 

### 2. Pull-Down button

iOS 14 이상부터 버튼은 사용자가 선택할 수 있는 아이템이나 동작들을 나열하는 pull-down menu를 보여줄 수 있다! 주로 현재 맥락에 유용한 동작들의 목록을 나타내는데 사용된다고 한다. 이를 menu라고 부르기도 한다. 

이러한 pull-down menu는 action sheet, context menu, popover에 비교해 몇 가지 이점을 제공한다. 

- menu는 버튼과 가까운 위치에서 보여진다. 그래서 사용자가 즉시 menu의 항목들과 수행하는 동작들간의 관계를 이해하기 쉽다. 
- 동작들을 나열하는 것 뿐만 아니라, 기본 동작에 영향을 미치는 selections를 제공할 수 있다. 
- menus는 view에 빠르게 애니메이션화되며 화면이 나타날 때 화면을 흐리게 하지 않아 전환과 전체적인 느낌을 가볍게 느낄 수 있다. 

또한 동작과 직접적으로 관련된 옵션들을 나타내기 위해서 menu를 사용하기를 권장한다. 이에 버튼들을 인터페이스에 추가하지 않아도 나열할 수 있는 것이다. 정렬이 필요할 때는 정렬의 속성을, add를 탭하면 원하는 아이템을 제공할 수 있다. 

이러한 pull-down 버튼을 보여주기 위해 더보기 버튼을 추가하는 것이 좋다고 한다. 추가적으로 pull-down의 항목 중 하나가 어떠한 것을 삭제할 수 있을 때, 의도를 확인하기 위해 한 번 더 물을 것을 권장한다. 이에 사용자의 데이터가 유실되는 것을 피할 수 있다. 또한 사용자의 이해를 위해 타이틀을 추가하거나, 적절한 아이콘을 추가하여 텍스트와 함께 이해를 도울 수 있다고 한다! 

구현은 간단히 해 볼 수 있다. 

```swift
let moreOptionButton = UIBarButtonItem(image: UIImage(systemName: "ellipsis.circle"), style: .plain, target: self, action: nil)
let shareAction = UIAction(title: "공유", image: UIImage(systemName: "square.and.arrow.up.fill")) { _ in
    // 공유 기능
}
let deleteAction = UIAction(title: "삭제", image: UIImage(systemName: "trash.fill"), attributes: .destructive) { _ in
    // 삭제 기능
}
let optionMenu = UIMenu(options: .displayInline, children: [shareAction, deleteAction])
moreOptionButton.menu = optionMenu
```

결과는 다음과 같다

![](https://i.imgur.com/d2n7w9T.png)

actionSheet보다는 간결하고 의미전달에도 부족함이 없는 것 같다! 

### 3. UITableView Swipe Action 

UITableView의 UITableViewDelegate 프로토콜을 채택하여 swipe와 관련한 메서드를 구현할 수 있다. 

alertViewController에 UIAlertAction을 추가한 것 처럼, `UISwipeActionConfiguration`에 `UIContextualAction` 객체를 생성하고 추가해주면 된다. 

간단하니 바로 코드로 보자.

```swift
override func tableView(_ tableView: UITableView, trailingSwipeActionsConfigurationForRowAt indexPath: IndexPath) -> UISwipeActionsConfiguration? {
    let deleteAction = UIContextualAction(style: .destructive, title: nil) { _, _, completionHandler in
        // 삭제 기능
        completionHandler(true)
    }
    deleteAction.image = UIImage(systemName: "trash.fill")
    deleteAction.backgroundColor = .systemRed

    let shareAction = UIContextualAction(style: .normal, title: nil) { _, _, completionHandler in
        // 공유 기능
        completionHandler(true)
    }
    shareAction.image = UIImage(systemName: "square.and.arrow.up.fill")
    shareAction.backgroundColor = .systemIndigo

    let swipeActions = UISwipeActionsConfiguration(actions: [deleteAction, shareAction])
    swipeActions.performsFirstActionWithFullSwipe = false
    return swipeActions
}
```

스와이프를 쭉 하다보면 자동으로 첫 번째 action이 실행되게 된다. 이를 방지하기 위해 `performsFirstActionWithFullSwipe` 옵션을 false로 주었다.

swipe actions를 추가해줄 때에는 반대로 들어간다고 생각하면 편하다. 
즉 [A, B]로 넣어주면 stack처럼 뒤에서 부터 뽑혀나가기 때문에 실제로 화면에서는 [B, A] 순서로 위치한 것을 확인할 수 있다. 

![](https://i.imgur.com/Oi8WvIm.gif)

### 4. BeautyAlert ToDo

BeautyAlert에 반영해볼 만한 기능들인데... 날 잡고 반영해봐야겠다.
시간이 오래 걸릴만한 건 아니지만... 그냥 시간 내는게 쉽지 않다ㅠ

- 코드
	- 타이틀/컨텐츠 줄 수 제한 제거 
- 문서
	- 1button / 2buttons case 변경 
	- 색상 커스터마이징 변경 
- 크기
	- iPad 대응
	- iPhone에서의 상대적인 크기 척도 설정 요구

## 고민된 점 
- Core Animation을 주는 방법, 커스텀 애니메이션을 만드는 방법?
- action sheet? pull-down button?

## 해결 방법 
- 공식 문서 및 HIG 읽고 구현해보기! 
---

**Ref**

https://developer.apple.com/documentation/quartzcore/calayer/1410744-presentationlayer

https://developer.apple.com/documentation/quartzcore/calayer/1410853-modellayer

http://minsone.github.io/mac/ios/coreanimation-implicit-animation

https://www.objc.io/issues/12-animations/animations-explained/

https://developer.apple.com/documentation/quartzcore/calayer/1410817-anchorpoint

https://developer.apple.com/design/human-interface-guidelines/ios/controls/buttons/#:~:text=displays%20every%20item.-,Pull%2DDown%20Buttons,-A%20pull%2Ddown
