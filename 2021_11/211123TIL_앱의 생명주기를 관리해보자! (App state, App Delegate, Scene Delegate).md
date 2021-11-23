# 앱의 생명주기를 관리해보자! (App state, App Delegate, Scene Delegate)

## 211123_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### App의 상태 종류

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FReETN%2Fbtra35cGBpA%2F7T7mkdivVdsvqS0l5QBkck%2Fimg.png)

foreground에 있을 때 메모리 및 기타 시스템 리소스에 높은 우선순위를 가지며, 시스템은 이러한 리소스를 사용할 수 있도록 필요에 따라 background 앱을 종료할 수 있다.

-   Not-Running : 실행되지 않았거나, 시스템에 의해 종료된 상태
-   Inactive : 실행 중이지만 이벤트를 받고 있지 않은 상태. 앱 실행 중 미리알림 또는 알럿이 화면을 덮어서 앱이 실질적으로 이벤트는 받지 못하는 상태 등을 의미함 (전화왔을 때, 알림창 등)
-   Active : 어플리케이션이 실질적으로 활동하고 있는 상태
-   Background : 백그라운드 상태에서 실질적인 동작을 하고 있는 상태. 백그라운드에서 음악을 재생하거나, 이동 경로를 트래킹하는 경우
-   Suspended : 백그라운드 상태에서 활동을 멈춘 상태. 빠른 재실행을 위해 메모리에 적재된 상태지만 실질적으로 동작하고 있지는 않음. 메모리가 부족할 때, 시스템이 강제 종료시킨다.

### App Life Cycle 관리하기 

앱이 foreground나 background에 있을 때, 시스템 알림에 반응하거나 시스템과 관련있는 다른 이벤트 들을 처리하는 것을 app life cycle내에서 관리하게 된다. 

app의 상태(state)가 바뀌게 되는 경우 `UIKit`은 적절한 delegate 객체의 매서드를 호출하여 사용자에게 알려준다.

- iOS 13 그리고 그 이후의 경우 `UISceneDelegate` 객체를 사용하여 scene 기반의 앱에서 발생하는 life-cycle 이벤트에 반응한다
- iOS 12 그리고 그 이전의 경우 `UIApplicationDelegate` 객체를 사용하여 life-cycle 이벤트들에 대해 반응한다. 

iOS 버전의 차이에 따라 불려지는 delegate가 다른 이유는 아래의 WWDC 세션 정리 내용을 참고하면 될 듯 하다. 간단하게 이야기하면 iOS 멀티 윈도우를 지원함에 따라 `UISceneDelegate`가 생겨났고, `UIApplicationDelegate`의 역할을 일부 대신한다고 봐도 될 것 같다. 

> 1 . Scene 기반의 Life-Cycle 이벤트들에 대한 반응

만약 앱이 scenes를 지원한다면 `UIKit`은 각각 life-cycle 이벤트들을 분리하여 전달할 것이다. scene이란 기기에서 실행되고 있는 앱의 UI를 하나의 인스턴스라고 볼 수 있다. 사용자는 각 앱에서 여러 개의 scene을 생성할 수 있고 각각 보여주거나 숨길 수 있다. 왜냐하면 각 scene은 각자의 life-cycle을 가지고 있기 때문에 각각의 실행 상태가 다를 수 있게 된다. 예를 들면, 하나의 scene은 foreground에 있고, 다른 하나의 scene은 background에 있거나 suspended 되었을 수 있다는 것을 의미한다. 

![](https://docs-assets.developer.apple.com/published/8e113a7266/scene-state~dark@2x.png)

위 그림은 scene에 대한 상태 전환을 설명하고 있다. 사용자 혹은 시스템이 새로운 scene을 앱에 요청한다면, `UIKit`은 이를 생성하고 unattached(미연결) 상태로 넣어준다. 사용자가 요청한 scene은 빠르게 foreground로 이동하며 화면에 나타나게 된다. 시스템이 요청한 scene의 경우 일반적으로 background로 이동하여 이벤트를 처리할 수 있게 이동시킨다. 예를 들어서 시스템은 location 이벤트를 처리하기 위해 background에 있는 scene을 실행시킬 수 있다. 

사용자가 앱의 UI를 제거(dismiss)하게 되면, `UIKit`
은 연관된 scene을 background 상태로 이동시키고, 언젠가는 suspended 상태로 이동시키게 된다. `UIKit`
은 언제든 background나 suspended 상태의 scene을 연결을 끊고 리소스를 회수하여 해당 scene을 unattached 상태로 되돌릴 수 있다. 

scene 전환을 사용하여 다음과 같은 작업을 수행할 수 있다. 
- `UIKit`이 scene을 app에 연결하려고 할 때, scene의 초기 UI를 구성하고 scene에 필요한 데이터를 로드한다
- foreground-active 상태로 전환할 때, UI를 구성하고 사용자와 상호작용할 준비한다. 
- foreground-active 상태를 벗어날 때, 데이터를 저장하고 앱의 동작을 정지(quiet)한다.
- background 상태로 진입할 때 중요한 작업을 마무리하고 메모리를 최대한 비우고, 앱 snapshot을 위해 준비한다. 
- scene의 연결이 해지될 때, scene과 관련된 공유자원을 처리한다.
- scene과 관련된 이벤트들 이외에도 `UIApplicationDelegate` 객체를 사용하여 앱의 시작에도 반응해야한다. 

> 2 . App 기반의 Life-Cycle 이벤트들에 대한 반응 

iOS 12 그리고 그 이전의 버전의 경우 위에서 설명한 scene을 지원하지 않는다. `UIKit`은 life-cycle 이벤트들을 `UIApplicationDelegate`객체에 전달해준다. app delegate는 별도의 화면에 표시되는 창을 포함하여 앱의 모든 화면(window)을 관리한다. 그 결과로 app 상태 전환은 외부 디스플레이 컨텐츠를 포함하여 앱의 모든 UI에 영향을 주게 된다. 

![](https://docs-assets.developer.apple.com/published/e6ac158845/app-state~dark@2x.png)


위 그림은 app delegate 객체와 관련된 상태 전환을 나타낸다. 실행 이후에, 시스템은 UI가 화면에 표시되려고 하는지 여부에 따라 앱을 inactive 혹은 background 상태로 전환시킨다. foreground 상태로 실행하는 경우, 시스템은 자동으로 앱을 active 상태로 전환시켜준다. 그 이후에 앱 종료 이전까지, 상태(state)는 불규칙하게 active와 background 사이에서 변하게 된다.

app 전환을 사용하여 다음과 같은 작업을 수행할 수 있다.
- 실행 단계에서, 앱의 데이터 구조와 UI를 초기화시킬 수 있다. 
- 앱을 활성화(activate)할 때, UI 구성을 완료하고 사용자와 상호작용할 준비를 한다.
- 앱을 비활성화(deactivate)할 때, 데이터를 저장하고 앱의 동작을 정지(quiet)한다. 
- background 상태로 진입할 때, 중요한 작업을 오나료하고 메모리를 가능한 만큼 정리하고, 앱 snapshot을 위해 준비한다. 
- 앱이 종료(terminate)될 때, 모든 작업을 즉시 멈추고 공유 리소스를 해제한다. 

> 3 . 다른 중요한 이벤트들에 대한 반응

life-cycle 이벤트들 이외에도, 앱은 아래 테이블의 이벤트들을 처리하기 위해 준비해야 한다. 이러한 이벤트들을 처리하려면 `UIApplicationDelegate` 객체를 사용해야 한다. 경우에 따라서, 앱의 다른 부분에서 응답할 수 있도록 알림(notifications)을 사용하여 이러한 이벤트들을 처리할 수 있다. 

|||
|---|---|
|메모리 경고 <br> (Memory warning)|앱의 메모리 사용량이 너무 높으면 이 경고를 받게 된다. 앱의 메모리 사용량을 줄여줘야 한다|
|보호받던 데이터가 사용가능/불가능해 질 때 <br> (Protected data becomes available/unavailable)|기기를 잠그거나 풀 때 해당 이벤트를 받을 수 있다|
|핸즈오프 작업 <br> (Handoff tasks)|`NSUserActivity`객체가 처리되어야 할 때 이벤트를 받을 수 있다.|
|시간 변화 <br> (Time changes)|여러 가지 다른 시간 변경사항에 대해 이벤트를 받을 수 있다.|
|URL 열기 <br> (Open URLs)|앱이 리소스를 열어야 할 때 이벤트를 받을 수 있다. |


### WWDC Architecting Your App for Multiple Windows

iOS 12 그리고 그 이전에는 App Delegate를 통해 화면을 관리해줬었다. 

`UIApplicationDelegate`는 1개의 process와 1개의 user interface 인스턴스를 가지고 있다. 여기서 `AppDelegate`는 2가지 역할을 한다.

-   Process Level의 이벤트 발생을 알려준다
    -   application이 launch / terminate되는지 등
-   UI의 상태변화를 알려준다.
    -   `willEnterForeground`, `DidEnterBackground` 같은 메서드를 통해 알림

![](https://blog.kakaocdn.net/dn/eTpK5I/btraXV2881k/QaMVMAMotHYn27Tj4kexvK/img.png)

UI LifeCycle들은 아래에서 2,3,4,5번과 같다. 
1,2번은 Process의 생애주기를 관리하는 것에 해당되며 앱이 실행되고, 종료되는 것을 관리해주는 역할을 한다. 

```swift
// 1. 앱이 처음 시작될 때 실행
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool

// 2. Active -> Inactive로 바뀔 때 실행
func applicationWillResignActive(_ application: UIApplication)

// 3. 앱이 Background 상태가 될 때 실행 (공유자원 해제 | 유저 데이터 저장)
func applicationDidEnterBackground(_ application: UIApplication)

// 4. Background -> Foreground로 이동될 때 실행
func applicationWillEnterForeground(_ application: UIApplication)

// 5. Active 상태가 되어 실행 중일 때 
func applicationDidBecomeActive(_ application: UIApplication)

// 6. 앱이 종료될 때 실행 
// 항상 호출되는 것을 보장하지는 않는다. 
func applicationWillTerminate(_ application: UIApplication)
```

하지만 iOS 13 이후부터는 1개의 process와 multiple user interface(multiple window: 한 스크린에 2개 이상의 화면을 동시에)를 지원함에 따라 `AppDelegate`는 `SceneDelegate`에게 일부 역할(UI LifeCycle)을 넘겨주게 된다.

![](https://blog.kakaocdn.net/dn/IHIrT/btraVTrcMCE/qCDcLzDp6iGs1uv17MSYHK/img.jpg)

UI의 상태 변화를 알려주는 역할들을 이제 `UIApplicationDelegate`가 아니라, `UISceneDelegate`에서 관리하게 되면서 메서드들 또한 변경되었다. 

```swift
// 위 AppDelegate 메서드들과 동일함  
1. scene(_: willConnectTo: options:)  
2. sceneWillResignActive(_ :)  
3. sceneDidEnterBackground(_ :)  
4. sceneWillEnterForeground(_ :)  
5. sceneDidBecomeActive(_ :)  
6. sceneDidDisconnect(_ :)
```

![](https://blog.kakaocdn.net/dn/qZ5Kl/btra9XkKuqX/GQQW410MO6StaKTfCX7Bg1/img.png)

살펴보면 기존에 `AppDelegate`에서 담당하던 UI의 상태변화를 알려주는 역할을 `SceneDelegate`가 하고 있음을 알 수 있다.

이에 따라 `AppDelegate`는 UI의 상태변화가 아닌 Session Lifecycle을 application에게 알리는 역할을 대신 하게 되었다.

-   `AppDelegate`
    -   Process Lifecyle
    -   Session Lifecycle (신규)
        -   새로운 Scene Session이 생성/제거될 때 application에게 알리는 역할.
-   `SceneDelegate`
    -   UI Lifecycle (기존 AppDelegate의 역할)

> Non-UI Setup은 AppDelegate의 `didFinishLaunching`에서 해준다!

## 고민된 점 
- scene이란?
- 왜 iOS 13이후로 scene delegate를 사용하는지?

## 해결 방법 
- 공식 문서와 WWDC 영상을 보고 학습 내용에 정리!
---

**Ref**
[Managing Your App's Life Cycle](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)
[Architecting Your App for Multiple Windows](https://developer.apple.com/videos/play/wwdc2019/258/)


