
# Responder Chain, Touch/Press/Gesture, DispatchGroup wait/notify, private extension

## 211227_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. Responder Chain

앱은 `reponder 객체`를 사용하여 이벤트를 수신하고 처리한다. reponder 객체는 `UIResponder` 클래스의 인스턴스로, 하위 클래스인 `UIView`, `UIViewController`, 그리고 `UIApplication`도 포함한다. responder들은 이벤트 데이터를 받고 무조건 이벤트를 처리하거나 다른 responder 객체로 이를 보내줘야 한다. 만약 앱이 이벤트를 받게 되면, `UIKit`은 자동으로 이벤트에 가장 적합한 responder 객체를 가리키는데 이를 `first responder`라고 한다. 

처리되지 않은 이벤트들은 responder로부터 앱내 responder 객체들의 동적 구성인 활성화된 responder chain내 다른 responder에게 전달된다. 그림을 보다싶이 앱의 reponder들로는 `UILabel`, `UITextField`, `UIButton`, 두 개의 `UIView`가 있다. 다이어그램은 이벤트들이 responder chain에 따라 responder에서 다음 responder로 어떻게 이동하는지 보여준다.

![](https://docs-assets.developer.apple.com/published/7c21d852b9/f17df5bc-d80b-4e17-81cf-4277b1e0f6e4.png)

만약 `UITextField`가 이벤트를 처리하지 않는다면 `UIKit`은 `UITextField`의 부모인 `UIView` 객체에게 이벤트를 전달한 후, `UIWindow`의 root view로 보낸다. root view에서 이벤트를 `UIWindow`로 보내기 전에 responder chain이 소유하고 있는 `ViewController`로 우회한다. 만약 `UIWindow`도 이벤트를 처리하지 못하면 `UIKit`은 `UIApplication` 객체에게 보내고, 해당 객체가 `UIResponder`의 인스턴스이고 아직 responder chain에 속하지 않은경우 app delegate에게 전달할 수 있다. 

즉 해당 타입이 이벤트를 처리하지 않을 경우, 다음 responder(= 부모)에게 계속 넘기게 되는 것이다. 

#### 이벤트의 first responder 결정하기 

`UIKit`은 해당 이벤트의 유형에 따라 이벤트의 first responder로 객체를 지정한다. 이벤트 유형은 다음과 같다. 

|이벤트 종류|first responder|
|---|---|
|Touch 이벤트|터치가 발생한 뷰|
|Press 이벤트|focus를 가지는 객체|
|Shake-motion 이벤트| 사용자(or UIKit)이 지정하는 객체|
|Remote-control 이벤트| 사용자(or UIKit)이 지정하는 객체|
|Editing 메뉴 메시지| 사용자(or UIKit)이 지정하는 객체|


> 가속도(accelerometers), 평형(gyroscopes), 자력(magnetometer) 관련한 모션 이벤트는 responder chain을 따르지 않는다. 대신에, Core Motion이 이러한 이벤트를 지정된 객체에 바로 전달해준다. 

제어는 액션 메시지를 사용하여 관련된 타겟 객체와 직접적으로 소통한다. 사용자가 제어(control)과 상호작용하게 되면, 제어는 타겟 객체에게 액션 메시지를 보낸다. 액션 메시지는 이벤트가 아니지만 responder chain의 이점은 취하고 있다. 만약 타겟 객체의 제어가 `nil`인 경우, `UIKit`은 타겟 객체부터 시작해서 적합한 액션 메서드를 구현하는 객체를 찾을 때 까지 responder chain을 가로지르게 된다. 예를 들어, `UIKit` 편집 메뉴는 `cut()`, `copy()`, `paste()` 메서드를 구현하는 responder 객체를 찾기 위해 이러한 행위를 사용한다.

제스처 인식자는 touch나 press 이벤트를 뷰보다 먼저 받는다. 만약 뷰의 제스처 인식자가 일련의 터치를 인식하지 못하는 경우, `UIKit`은 터치들을 뷰로 보내게 된다. 만약 뷰도 터치들을 처리하지 못하는 경우, `UIKit`은 이를 responder chain으로 넘기게 된다. 

#### 터치 이벤트가 포함된 responder 확인하기

 `UIKit`은 뷰 기반의 hit-testing을 통해 어디서 터치 이벤트가 발생했는지 확인한다. 특히,  `UIKit`은 뷰 계층에 속하는 뷰 객체의 bounds와 터치 영역을 비교한다. `UIView`의 `hitTest()`메서드는 뷰 계층을 가로지르는데(횡단), 특정 터치를 포함하는 가장 깊은 하위 뷰를 찾는다. 이 하위 뷰가 터치 이벤트에 대한 first responder가 되는 것이다. 

> 만약 터치 영역이 뷰의 bounds 밖이었다면, `hitTest()` 메서드는 뷰와 모든 하위 뷰들을 무시한다. 따라서 뷰의 `clipsToBound` 속성이 false이면, 터치를 포함하는 경우에도 뷰의 bounds를 벗어난 하위 뷰는 반환되지 않는다. 

![](https://i.stack.imgur.com/ekj9N.png)
> 1번 이미지 : `clipsToBounds = true`
> 2번 이미지 : `clipsToBounds = false`

터치가 발생하면, `UIKit`은 `UITouch` 객체를 만들고 뷰와 연결한다. 터치의 위치나 다른 파라미터가 변경되면 `UIKit`은 동일한 `UITouch` 객체를 새 정보로 업데이트한다. 변경되지 않는 속성은 뷰 쁀이다. (터치 위치가 원래 뷰 밖으로 이동해도 터치 뷰 속성의 값은 변경되지 않는다) 터치가 끝나면 `UIKit`은 `UITouch` 객체를 해제(release)한다.

#### Responder Chain 변경하기

responder 객체의 프로퍼티인 `next`를 재정의하여 responder chain을 변경할 수 있다. 이 때 `next` responder는 반환되는 객체이다. 

많은 `UIKit` 클래스가 이미 이 속성을 재정의하고 다음을 포함한 특정 객체를 반환한다. 

- `UIVIew` 객체. 뷰가 뷰 컨트롤러의 root 뷰라면, 다음 responder는 window 객체이다. 
- `UIViewController` 객체.
	- 뷰 컨트롤러의 뷰가 window의 root 뷰라면, 다음 reponder는 window 객체이다. 
	- 만약 뷰 컨트롤러가 다른 뷰 컨트롤러에 의해 띄워졌다면, 다음 responder는 띄우는 뷰 컨트롤러가 된다. 
- `UIWindow` 객체. `UIWindow`의 다음 responder는 `UIApplication` 객체이다.
- `UIApplication` 객체. 다음 responder는 app delegate이지만, app delegate가 `UIResponder`의 인스턴스이고, 뷰 혹은 뷰 컨트롤러 혹은 앱 객체 자체가 아닌 경우에만 해당된다. 

> Shake Motion
>  - 모든 뷰에서 처리하도록 만들 필요는 없다
>  - 상위의 한 객체에서 처리하는 것이 좋다
>  - 주로 app delegate에서 처리를 해준다.
>  - 만약 뷰 내에 처리해야 할 shake 액션이 있다면, 이를 먼저 처리하고 최상위 app delegate에서 shake 액션을 최종 처리해준다!


### 2. Touches, Presses 그리고 Gestures

앱에서 해당 코드를 재사용할 수 있도록 앱의 이벤트 처리 로직을 제스처 인식기에 캡슐화한다. 

표준 UIKit 뷰 및 컨트롤을 사용하여 앱을 구축하면 UIKit이 자동으로 터치 이벤트(멀티터치 이벤트 포함)를 처리한다. 하지만 사용자 정의 뷰를 사용하여 내용을 표시하는 경우 뷰에서 발생하는 모든 터치 이벤트를 처리해야 합니다. 터치 이벤트를 직접 처리하는 방법에는 두 가지가 있다.

- 제스처 인식기를 사용하여 터치를 추적
- UIView 하위 클래스에서 터치를 직접 추적

이제 하나씩 알아보자. 

#### Handling UIKit Gestures

제스처 인식기를 사용하면 간단하게 터치를 처리할 수 있고 일관적인 유저 경험을 줄 수 있다. 어떠한 뷰에도 하나 이상의 제스처를 연결할 수 있다. 제스처 인식기는 해당 뷰에 대한 수신 이벤트를 처리 및 해석하고 알려진 패턴과 일치시키는 데 필요한 모든 논리를 캡슐화한다. 일치하는 항목이 탐지되면 제스처 인식기는 할당된 대상 객체(뷰 컨트롤러, 뷰 자체, 다른 앱 내의 객체)에게 알린다. 

제스처 인식기는 타겟-액션 디자인 패턴을 사용하여 알림을 보낸다. `UITapGestureRecognizer` 객체는 한 손가락으로 뷰를 탭한 것을 탐지할 때, 뷰 컨트롤러의 액션 메서드를 호출하여 터치에 대한 반응을 제공한다.

> 타겟-액션도 가능하지만 delegate pattern으로도 처리가능하다.


![](https://docs-assets.developer.apple.com/published/7c21d852b9/0c8c5e29-c846-4a16-988b-3d809eafbb6b.png)

제스처 인식기는 두 개의 타입이 있다. 비연속적(discrete)이고 연속적인(continuous) 특성으로 나눌 수 있다. 비연속적인 제스처 인식기는 제스처가 인식된 이후 딱 한 번만 액션 메서드를 호출한다. 초기 인식 기준이 충족되면 연속 제스처 인식기는 동작의 이벤트 정보가 변경될 때 마다 사용자에게 알려주는 수행을 여러번 반복한다. 예를 들어서 `UIPanGestureRecognizer`는 터치 위치가 변경될 때마다 액션 메서드를 호출한다. 

Interface Builder에는 각 표준 UIKit 제스처 인식기에 대한 객체가 포함되어 있다. 또한 사용자 지정 `UIGestureRecognizer` 하위 클래스를 나타내는 데 사용할 수 있는 사용자 지정 제스처 인식기 객체도 포함되어 있다.

##### 제스처 인식기 구성하기 

제스처 인식기를 구성하려면 다음과 같이 수행하면 된다. 

1. 스토리보드에서 제스처 인식기를 끌어다가 view에 놓는다. 
2. 제스처가 인식되었을 때 수행될 액션 메서드를 구현한다.
3. 액션 메서드와 제스처 인식기를 연결한다. 

우클릭을 해서 제스처 인식기와 인터페이스 빌더를 연결할 수 있고, 적합한 액션 메서드 또한 연결할 수 있다. 물론 코드로만 구현할 수도 있는데, 이 때는 `addTarget()` 메서드를 사용한다. 

```swift
@IBAction func myActionMethod(_ sender: UIGestureRecognizer)
```

제스처 인식기와 연결된 액션 메서드는 해당 제스처에 대한 앱의 응답을 제공한다. 비연속적인 제스처의 경우 액션 메서드는 버튼과 유사하다. 액션 메서드가 호출되면 해당 제스처에 적합한 작업을 수행한다. 연속적인 제스처의 경우 동작 방법은 제스처 인식에 응답할 수 있지만 제스처가 인식되기 전에 이벤트를 추적할 수도 있다. 추적 이벤트를 사용하면 보다 상호적인 경험을 만들 수있다. 예를 들어 `UIPanGestureRecognizer`객체의 업데이트를 통해 앱의 콘텐츠 위치를 변경해 줄 수 있다. 

제스처 인식기의 상태(state) 속성은 객체의 현재 인식 상태를 의미한다. 연속적인 제스처의 경우 제스처 인식기는 `UIGestureRecognizer.state.began`에서 `UIGestureRecognizer.state.changed`로 `UIGestureRecognizer.state.ended`로, 혹은 `UIGestureRecognizer.state.cancelled`로 업데이트 될 수 있다. 액션 메서드는 이 프로퍼티를 사용하여 적합한 액션을 결정한다. 예를 들어, began과 changed 상태를 사용하여 컨텐츠의 일시적인 변화를 나타낼 수 있고, ended 상태를 사용하여 해당 변화가 영구적임을 나타내고, cancelled 상태를 통해 변화를 제거할 수 있다. 항상 액션을 취하기 전에 제스처 인식기의 state 프로퍼티를 확인하는 것이 중요하다. 


#### Handling Touches in Your View

터치를 처리하는 것이 뷰의 컨텐츠에 복잡하게 연결된 경우, 뷰의 하위 클래스에서 직접 터치 이벤트를 사용한다.

앞서 본 것 처럼, 만약 사용자 정의 뷰와 함께 제스처 인식기를 사용할 계획이 없다면, 뷰 자체에서 터치 이벤트를 바로 처리해줄 수 있다. 왜냐하면 뷰들은 responders이기 때문이며, 다른 타입의 이벤트들 뿐만 아니라 멀티 터치도 처리할 수 있다. `UIKit`은 뷰에서 발생한 터치 이벤트를 확인하기 위해 `touchesBegan()`, `touchesMoved()`나 `touchesEnded()` 메서드를 사용한다. 이 메서드들은 사용자 정의 뷰에서 재정의 가능하고 터치 이벤트들에 대해 반응을 제공할 때 사용할 수 있다. 

이 메서드들을 뷰에서 재정의하여 터치의 단계에 따라 이벤트를 처리해 줄 수 있다. 아래의 그림을 보면 터치의 단계를 나타내고 있다. 손가락이 스크린을 너치하면 `UIKit`은 `UITouch` 객체를 생성하고, 터치 위치를 적절한 지점으로 설정한 다음, `phase` 속성을 `UITouch.Phase.began`으로 설정해준다. 동일한 손가락이 스크린을 이동하게 되면, `UIKit`은 터치 위치와 `phase` 프로퍼티를 `UITouch.Phase.moved`로 변경해준다. 사용자가 손가락을 스크린에서 떼어내게 되면 `UIKit`은 `phase` 프로퍼티를 `UITouch.Phase.ended`로 바꾸고 일련의 터치를 종료시킨다. 

![](https://docs-assets.developer.apple.com/published/7c21d852b9/08b952fe-6f46-41eb-8b8a-4830c1d48842.png)

유사하게, 시스템은 언제든지 진행 중인 터치를 취소시킬 수 있다. 예를 들어 터치하고 있는데 전화가 온다거나 하는 경우이다. 이 때 `UIKit`은 `touchesCancelled()` 메서드를 호출하여 이를 알리게 된다. 이 메서드를 통해 뷰의 데이터 구조를 필요에 따라 정리할 수 있게 된다. 

`UIKit`은 화면에 닿는 각각의 손가락에 대해  `UITouch` 객체를 만든다. 터치 자체는 현재 `UIEvent` 객체와 함께 전달된다. `UIKit`은 손가락 터치와 애플펜슬 터치를 구분하여 각각 다르게 다룰 수 있다. 

> 기본 구성에서 view는 두 개 이상의 손가락이 view를 터치하는 경우에도 이벤트와 관련된 첫 번째 `UITouch` 객체만 수신한다. 추가 터치를 수신하려면 view의 `isMultipleTouchEnabled`를 true로 설정해야한다. 또한 인터페이스 빌더에서 attributes inspector를 사용하여 이 속성의 값을 바꿔줄 수도 있다. 

> #### Tip
> `touchesBegan()`등 메서드는 `UIresponder`의 메서드이나 실제로는 `UIview`클래스를 상속 받은 클래스에 재정의하여 사용한다. 

> #### Tip
> press는 물리 버튼을 말한다. 

> #### Tip
> 터치 되었다고 해서 해당 이벤트에 respond한 것은 아니다. 터치만 인지가 된 것이지 아무 처리를 해주지 않은 상태이다. 즉 `touchesBegan()`에 출력이 되어도 respond한 것은 아니라는 것이다. 
> 계층의 중간에 이벤트를 처리할 수 있는 responder가 없으면, app delegate에서부터 하위로 `touchesBegan()`을 계속 출력한다. 
> `hitTest`의 동작원리와 유사한데 이와 관련이 있는지 알아봐야겠다!


### 3. DispatchGroup의 wait와 notify

DispatchGroup의 `wait()`와 `notify()`에 대해 알아보려고한다.

먼저 DispatchGroup이란 여러 일들을 그룹화 하여 하나의 유닛처럼 관리할 수 있도록 해준다. 그래서 여러 개의 workItems를 넣어줄 수도 있고 동일한 큐, 다른 큐에서 비동기적으로도 실행하도록 스케줄링 해줄 수 있다.

DispatchGroup의 모든 작업이 끝나게 되는 시점에 원하는 동작을 수행할 수 있는데, 이는 `wait()`와 `notify()`로 구현할 수 있다. 

> Thread를 직접적으로 정지시키는 `Thread.sleep`은 바람직하지 않은 방법이라고 한다. 해당 쓰레드를 정지시키게 되면, 다른 작업 또한 진행되지 못하여 자원이 비효율적으로 사용되고, 만약 main을 잠재우게 되면 UI업데이트에도 영향을 미치는 등 사이드 이펙트가 생길 수 있다. 

`wait()` 의 경우 synchronous 하게 동작하고, `notify()` 의 경우 asynchronous 하게 동작한다고 한다. 

> 1 . `notify(qos:flags:queue:execute:)` 

먼저 notify는 execute 블록에 들어온 코드를 특정 속성의 queue에 넣어 현재 그룹의 모든 작업이 실행을 마쳤을 때 동작한다. 

앞서 본 것처럼 notify는 비동기적으로 작동한다고 했다. 즉 group의 동작이 수행되고 있는 와중에도 다른 스레드의 작업이 수행될 수 있는 것이다. 

아래 코드를 보자. 

```swift
let group = DispatchGroup()
let queueImage = DispatchQueue(label: "com.image")
let queueVideo = DispatchQueue(label: "com.video")
queueImage.async(group: group) {
    print("image 시작")
    Thread.sleep(forTimeInterval: 1)
    print("image 끝")
}

queueImage.async(group: group) {
    print("video 시작")
    Thread.sleep(forTimeInterval: 1)
    print("video 끝")
}

group.notify(queue: .main) {
    print("all finished.")
}

print(123)
```

이 경우에 순서를 예측해보자. 

비동기적으로 관리해준다고 했으니 메인큐에 있는 `print(123)`
은 group의 작업이 마치지 않아도 실핼될 수 있을 것 같다. 

결과는 다음과 같다.

```swift
image 시작
123 // groupd의 작업이 끝나지 않았는데 다른 쓰레드에서 실행된다.
image 끝
video 시작
video 끝
all finished.
```

이렇게 된다면 `queueImage`에서 처리하고 있는 와중에 다른 코드가 불릴 수 있게 되는 것이다. 

> 2 . `wait()`

반면에 wait은 동기적으로 작동한다. 즉, group의 작업이 수행중이라면, 무조건 group의 작업이 모두 종료될 때 까지 아무런 다른 작업도 수행되면 안된다.

```swift
let group = DispatchGroup()
let queueImage = DispatchQueue(label: "com.image")
let queueVideo = DispatchQueue(label: "com.video")
queueImage.async(group: group) {
    print("image 시작")
    Thread.sleep(forTimeInterval: 1)
    print("image 끝")
}

queueImage.async(group: group) {
    print("video 시작")
    Thread.sleep(forTimeInterval: 1)
    print("video 끝")
}

group.wait()

print("all finished.")

print(123)
``` 

결과를 예상해보면 아까와는 달리 물리적인 순서 그대로 코드가 실행될 것만 같다. 

```swift
image 시작
image 끝
video 시작
video 끝
all finished.
123
```

결과를 보면 예측한대로, 순서대로 실행이 되고 있다. 또한 group의 작업이 수행 중일 때는 다른 쓰레드에서도 작업이 진행되지 않고 있다. 동기적으로 작동되어서 group의 작업이 끝나기만을 모든 쓰레드들이 보고 있기 때문이다. 

이에 wait이 지나고 다른 작업들이 수행되는 것이다. wait에는 timeout을 걸어서 원하는 시간만큼만 기다릴 수도 있다. 


### 4. extension 접근제어자 

특정한 부분을 위해 특정 기능을 만들게 되는 경우가 있는데, 이 때 extension을 통해 구현할 때가 있다. 이 때는 extension을 해주긴 하나, 접근제어자를 붙여서 해당 파일 내에서만 작동 가능하도록 만들어 줄 수 있다. 

동일한 파일 내에서 extension하게 되는 경우, 이 때 private과 fileprivate은 동일하게 취급된다. 

```swift
// A.file
private extension A {
	// ~~
}

fileprivate extension A {
	//
}
```

즉, private으로 하나 fileprivate으로 하나 동일하게 인식되는 것이다. 이 때 만큼은 동일하게 해당 파일이 scope가 되는 것이다. 

## 고민된 점 
- 앱에서 터치가 인식되고 제어되는 방법
- Thread.sleep하지 않고 시간 간격을 두는 방법

## 해결 방법 
- 터치를 처리하는 방법에 대한 문서 읽기 
- DispatchGroup의 wait 기능
---

**Ref**

https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events

https://developer.apple.com/documentation/uikit/touches_presses_and_gestures

https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/handling_touches_in_your_view

https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/handling_uikit_gestures

https://github.com/yagom-academy/ios-bank-manager/pull/118#discussion_r775415950

https://developer.apple.com/documentation/dispatch/dispatchgroup/2016090-wait

https://developer.apple.com/documentation/dispatch/dispatchgroup/2016066-notify

