# Scene을 이해해보고 App Life Cycle 뿌수기

## 211126_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Multi Scene이란?

하나의 앱에서 여러 scene을(여러 개의 독립적인 실행) 띄울 수 있다. 

화면을 분할하는 splitView와는 다르다!

### 버전에 따른 SceneDelegate & AppDelegate

SceneDelegate는 iOS 13 이후 버전부터 지원하기 때문에 빌드 타겟을 확인하는 것이 중요하다. 기본적으로 Xcode에서 새 프로젝트를 생성하게 되면 AppDelegate와 SceneDelegate 파일이 생기게 된다. 

왜냐? [앞서 봤던 것](https://github.com/ChaminLee/TIL/blob/main/2021_11/211123TIL_%EC%95%B1%EC%9D%98%20%EC%83%9D%EB%AA%85%EC%A3%BC%EA%B8%B0%EB%A5%BC%20%EA%B4%80%EB%A6%AC%ED%95%B4%EB%B3%B4%EC%9E%90!%20(App%20state%2C%20App%20Delegate%2C%20Scene%20Delegate).md) 처럼 scene을 지원하게 되면서 AppDelegate는 Process Life Cycle, Session Life Cycle을 관리하고, SceneDelegate가 UI Life Cycle을 관리하게 되었기 때문이다.

이에 iOS 13 이상에서는 역할이 분담되지만, iOS 12 이하의 버전에서는 모두 App Delegate가 담당하고 있다. 이 때문에 만약 scene이 있는 버전, 없는 버전을 모두 지원해주고자 한다면 기본적으로 Scene Delegate에서 제공하는 Life Cycle 코드 말고도, App Delegate 내에 Life Cycle 코드를 적어줘야 한다! 

대신 코드를 양쪽에 적어줌과 동시에 Scene Delegate 파일의 클래스 위에 `@available(iOS 13.0, *)`를 적어줌으로써 iOS 13 이상의 버전에서 빌드가 되도록 만들어 줄 수 있다. 그렇게 되면 iOS 12 이하의 버전에서는 Scene Delegate가 작동하지 않는다~!

### UIWindow?

> UIWindow란 앱의 배경이 되는 UI이며, 위의 view에 이벤트를 보내주는 객체이다. 

이오 같이 배경이 되는 역할이기에, iOS 앱은 적어도 하나 이상의 `UIWindow`를 갖게 된다.

구현부를 살펴보자. 

```swift
@MainActor class UIWindow : UIView
```

`@MainActor`란 해당 클래스가 메인 쓰레드에서 실행됨을 나타낸다고 한다. `UIWindow`는 UIView를 상속받고 있는 클래스로서, 어떻게 보면 그냥 뷰라고 봐도 될 것 같다. 

window는 이벤트를 처리하기 위해, 기본적인 앱의 작동과 같은 다른 작업들을 수행하기 위해 view controller들과 작업한다. UIKit은 대부분 window 관련한 상호 작용을 처리하며, 많은 앱 동작을 구현하기 위해 필요한 경우 다른 객체들과 함깨 작업하기도 한다. 

다음과 같은 작업 수행해야 하는 경우 windows를 사용한다. 

- 앱의 컨텐츠를 보여주기 위해 메인 window를 제공
- 추가 컨텐츠를 보여주기 위해 추가 window를 만드는 경우 

보통은 Xcode가 앱의 main window를 제공해준다. 새로운 iOS 프로젝트는 앱의 view를 정의하기 위해 스토리보드를 사용한다. 스토리보드는 Xcode가 템플릿으로 자동으로 제공해주는 app delegate내 window 프로퍼티의 존재를 필요로 한다. 만약 앱이 스토리보드를 사용하고 있지 않다면, window를 스스로 만들어야 한다!

대다수의 앱은 기기의 메인 스크린에 앱 컨텐츠를 보여주기 위해 하나의 window를 필요로 한다. 기기의 메인 스크린에 추가적인 window를 만들 수 있지만, 추가적인 windows는 공통적으로 외부 스크린에 컨텐츠를 보여주는데 사용된다. 

![](https://docs-assets.developer.apple.com/published/58983ac3d2/5aa9566f-0275-4f33-a88e-50daf059cc80.png)

사용자는 AirPlay이나 케이블을 통해 위 사진처럼 외부 기기를 연결하여 추가적인 스크린을 보여줄 수 있다. 각 추가적인 스크린들은 새로운 공간을 나타내고, 앱의 컨텐츠를 보여줄 수 있고, UIScreen 객체를 사용하여 관리될 수 있다! 위 사진처럼 아이폰 화면에는 컨트롤러를 연결된 디스플레이에는 게임 화면을 보여줄 수 있게 된다. 

`UIWindow` 객체는 다음과 같은 일부 작업에도 사용할 수 있다. 
- 다른 window를 기준으로 window의 가시성에 영향을 주는 window의 z축 설정 (화면을 계속 쌓는 행위)
- window를 표시하고 키보드 이벤트의 대상의 지정
- 좌표값을 window의 좌표계로 변환하고, window의 좌표계에서 다시 변환
- window의 root viewcontroller 변경
- window가 표시되는 화면 변경 
 
 windows 자체는 시각적인 부분을 스스로 가지고 있지 않다. 대신에 window는 하나 혹은 그 이상의 view들을 가질 수 있으며, 이는 window의 root view controller에 의해 관리된다. 스토리보드 사용시 인터페이스에 적절한 view를 만들기만 하면 root viewcontroller를 구성할 수 있게 된다. 

그리고 `UIWindow`의 하위 클래스는 거의 필요하지 않다. window에서 구현할 수 있는 동작의 종류는 일반적으로 상위 수준인 view controller에서 더 쉽게 구현할 수 있다. window의 키 상태가 변경될 때 특정 동작을 구현하기 위해 `becomeKey()`나 `resignKey()` 메서드를 재정의(override)하는 것이 하위 클래스 중를 필요로 하는 순간 중 하나이다. 즉 키가 되는 window를 정해주기 위한 메서드를 부르기 위해 하위 클래스가 필요하다는 말 같다. 
이는 아래의 키보드 관련 예시를 보고 이해해 볼 수 있다.

터치 이벤트는 발생한 window로 전달되는 반면, 관련 좌표 값이 없는 이벤트는 키(key) window에 전달된다. 다만 한 번에 하나의 window만 키(key) window가 될 수 있으며, 이를 위해 `isKeyWindow` 프로퍼티를 사용하여 상태를 확인할 수 있다. 대부분의 앱의 경우, 메인 window가 key가 되는 window이지만, `UIKit`은 필요에 따라 다른 window를 지정할 수 있다. 

이제 어떤 window가 키인지 알아야 하는데, 이 때 `didBecomeKeyNotification`과 `didResignKeyNotification` 알림을 통해 관찰한다. 시스템은 키 window의 변경에 따라 이러한 알림을 보내게 된다. window를 강제로 키가 되게 하거나, 키 window의 키 역할을 그만두게(resign)하려면 이 클래스(`UIWindow`)의 적절한 메서드를 호출해주면 된다!





### + WWDC19_Architecting Your App for Multiple Windows

하나의 앱을 Split View를 통해 띄웠을 때 각 Scene들을 동기화해주는 부분에 이해가 가지 않던 부분이 있었다. 

처음에는 앞서 설명되었던 `App Delegate`나 `Scene Delegate`
를 통해 동기화 문제를 해결하는 건가? 생각했는데 아니었다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbJ0lnM%2FbtqM9Ospe1u%2FLXKhReOKHh6aKDuI6jv761%2Fimg.jpg)

단순하게 생각해보면 NotificationCenter를 사용하여 이벤트를 주고 받았는데, Notification은 Scene 단위(View Controller)에서 오가는게 아니라, 더 상위의 App단위(ModelController)로 전달이 된다는 것을 알 수 있다.

그러니까 정리해보자면 특정 Scene에서 발생한 이벤트를 다른 Scene에서도 동기화시켜주기 위해 중앙의 역할을 하는 App에게 보내주어, 다른 Scene은 App을 통해 동기화를 정상적으로 할 수 있는 것이다. 

App Delegate는 앱 전반을 관리하고, Scene Delegate는 화면 단위를 관리한다는 개념을 두고 생각해보면 이해가 빠를 듯 하다. 

### App Life Cycle

scene이 생기는 것은 프로세스 자체가 하나 더 생기는 것이 아니라, 프로세스가 관리하는 화면이 하나 더 늘어난 것이다!

App Life Cycle이 Process LifeCycle과 연관이 있을거라고 생각한다! 

메모리와 프로세스 관점에서 봤을 때 [Process Scheduling](https://leechamin.tistory.com/520?category=1012929)를 참고해 볼 수 있을 것 같다. 

더 자세한 건 아래서 Scence/App based의 차이를 보며 대조해보면 이해가 쉬울 듯 하다. 

### Scene based와 App based Life Cycle 차이

### Scene based

![](https://media.vlpt.us/images/minni/post/c9384e68-d8da-4b6e-b6bc-4f10ec9829ba/image.png)

1. UIKit이 scene을 만들어서 `Unattached` 상태로 만들어준다. 
2. 사용자가 앱을 눌러 키면 launch screen이 뜨는 `foreground-inactive` 상태로 이동한다. 혹은 켜지는 도중 홈, 잠금버튼 클릭 및 다른 앱으로 이동하여 `background`로 이동할 수 있다.
3. 시스템은 launch screen이 끝나면 자동으로 UI Event를 받을 수 있는 상태(`foreground-active`)의 메인 화면으로 이동시켜준다
4. 사용자가 멀티태스킹, 홈버튼 누르기(active -> inactive -> background) 등을 통해 `foreground-inactive`상태로 변경시켜줄 수 있다. 
5. 사용자가 홈버튼, 잠금 버튼, 다른 앱으로 이동을 통해 `background` 상태로 변경시킬 수 있다. 
6. `background` 상태의 앱이 시스템 판단하에 대기 상태인 `suspended` 상태로 변경될 수 있다. 
7. `background` 상태의 앱이 일정 시간이 지나 시스템에 의해 다시 `unattached` 상태로 이동할 수 있다. 

### App based 

![](https://media.vlpt.us/images/minni/post/63da97e7-d4dc-409b-8369-6250f519fcd8/image.png)

1. 현재 앱은 실행되지 않은 상태(`Not Running`)이다. 
2. 사용자가 앱을 클릭하면 자동으로 launch screen을 띄우고(`inactive`) 메인 화면이 뜨게 된다(`active`)

이 부분은 추가 설명이 필요할 듯 하다. 

![](https://miro.medium.com/max/700/1*0XS9grFLWcz6Quzdu5syGw.png)
위 사진은 app based 기반의 life cycle 중 `Not Running` -> `active`로 가는 일련의 과정을 나타낸다. 

유저가 앱 아이콘을 누르고, `main()` , `UIApplicationMain()` 등 메서드가 실행되고 초기화가 완료되면 `didFinishLaunchingWithOptions` 메서드가 호출된다. 이 때 이제 running(foreground) 상태에 접어들게 되는데, 앱을 활성화 되었음을 알리기 위해 `DidBecomeActive` 메서드가 호출된다. 이제 그 다음에는 이벤트 루프를 돌며 발생하는 이벤트들을 받고 처리하게 된다. 

이 과정이 app based life cycle에서 `Not  Running -> inactive -> active`의 과정을 나타내주지 않나 싶다. 

그렇다면 `Not  Running -> background`는 어떤 과정으로 발생될까?

![](https://miro.medium.com/max/700/1*5LKm3FR67tuGYEeh_eDgIA.png)

위 사진은 `Not  Running -> background`가 되는 과정을 나타내준다. 실행되는 부분은 동일하다. 다만 다른 부분은 running(foreground) 상태가 아닌 `background` 상태로 진입하게 된다. 이에 불리는 메서드 또한 `DidEnterBackground`인 것을 볼 수 있다. 

이제 여기서 실행이 허용되어 있다면 이벤트들을 모니터하며 기다려준다.  만약 앱 실행이 허용이 되어있지 않았다면 `Not  Running -> background -> Suspended`로 바로 빠지게 된다. 

3. `active` ~ `inactive`, `inactive` ~ `background`는 사용자에 의해 상태가 변경될 수 있다. 
4. `Suspended` 상태의 앱이 일정 시간이 지나면 시스템에 의해 다시 `Not Running` 상태를 거쳐서 실행되어야 한다.
5. 사용자가 `Suspended` 상태의 앱을 다시 실행시켜 `inactive` 상태로 만들어 줄 수 있다. 그리고는 바로 `active` 상태로 갈 것 같다!

즉 **실선**은 사용자의 이벤트에 의해 발생할 수 있는 것이고, **점선**은 사용자가 아닌 시스템에 의해 발생될 수 있는 부분을 말하는 것 같다. 

> #### Life Cycle에서 메모리와 프로세스의 관점
> 1. `Unattached`: 아직 scene이 연결되지 않은 상태, 시스템으로 부터 연결하라는 알림을 받기 전까지는 이 상태를 유지한다. 이에 메모리 상에는 올라와있고, 실행중인 상태라고 볼 수 있다. (프로세스 생명주기에 기반해서 보면 Job -> Created라고도 볼 수 있을 것 같다)
> 2. `Suspended`: background에 있지만 아무것도 실행되고 있지 않고, 대기한다. 이에 다시 등장할 가능성이 있기에 메모리에서는 해제되지 않고 대기만 한다. (자원이 할당되기까지 기다리는 suspended blocked 상태인 것 같다)
> 3. `Not Running`: 실행되지 않았거나, 종료된 상태를 말한다. 이에 메모리에도 없고 프로세스 관점에서 실행되는 건 없다. ( terminate 상태와 같은 듯 하다)

## 고민된 점 
- Scene based와 App based Life Cycle의 차이
- UIWindow의 역할? 
- WWDC Scene 동기화 방법에 대해
- App Life Cycle은 Process Life Cycle과 연관이 깊은가?

## 해결 방법 
- 문서나 영상을 통해 학습내용에 정리해보았고, 머릿속에도 정리해보았다! 
---

**Ref**

[UIWindow](https://developer.apple.com/documentation/uikit/uiwindow)
[WWDC19_Architecting Your App for Multiple Windows](https://developer.apple.com/videos/play/wwdc2019/258/)
[Displaying Content on a Connected Screen](https://developer.apple.com/documentation/uikit/windows_and_screens/displaying_content_on_a_connected_screen)
[iOS App Life Cycle](https://medium.com/@neroxiao/ios-app-life-cycle-ec1b31cee9dc)
[iOS Application Life Cycle](https://velog.io/@minni/iOS-Application-Life-Cycle)


