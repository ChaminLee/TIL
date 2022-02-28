# Notification 

## 220228_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### !퀴즈 복습 

- TCP: 연결 지향형
- UDP: 비연결 지향형
	- 속도가 빠르지만 신뢰성 보장 X
- UIAccessibility 프로토콜 -> accessibility 지원 프로토콜
- XCTestExpectation -> 비동기적으로 도출되는 테스트의 결과를 검사하기 위함 (비동기 작업이 끝나는 시점에 fullfill을 사용하는 등)
- View의 constraint/layout은 viewDidAppear 이후에 확정된다. 실제 수행은 viewWillAppear에서 되고, viewDidAppear에서 확정이 되는 것

### 1. Notification 

push 알림을 보내기 위한 notification이다. 

- 로컬 푸시
	- 서버를 거치지 않고 로컬 정보에 기반해서 기기 자체에서 만들어서 푸시를 보내는 형태
	- 특정 조건에 따라 여러번 울리도록 만들 수 있다
	- 트리거가 존재한다 (시간, 지역, 날짜)
- remote noti를 쓰려면 무조건 APNs를 거쳐야 한다 
- remote는 언제올지 모르고 네트워크를 타고 오는 푸시를 의미한다.

로컬 푸시를 보내는 방법에 대해서 간단하게 알아보자. 

1. push notification 권한 받기
2. noti 내용 채우기
3. noti를 받았을 때 수행할 액션 정의

먼저 사용자에게 권한을 요청한다. 

```swift
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { didAllow, error in
    print(didAllow)
}
```
alert, sound, badge에 대한 요청을 한 번에 받도록 해준다. 

이제 보내고자 할 노티를 만들어준다. 

```swift
let content = UNMutableNotificationContent()
content.title = "Hi there"
content.body = "This is the letter from England..."
content.userInfo = ["target_view" : "yellowView"]
```

우선 content를 만들어서 내용을 채워준다. 제목, 내용 뿐만 아니라 보내고자 할 데이터를 딕셔너리 형태로 담아서 보내줄 수 있다. 

그 다음 트리거를 구현해보자.

```swift
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 3, repeats: false)
let request = UNNotificationRequest(identifier: UUID().uuidString, content: content, trigger: trigger)

UNUserNotificationCenter.current().add(request) { error in
    if error != nil {
        print(error)
    }
}
```

트리거는 아까 본 것 처럼 시간, 지역, 날짜등으로 설정 가능하다. 

트리거까지 만들었다면, 앞서 만든 컨텐츠를 함께 사용해서 request를 만들 수 있다. 

그 다음 request를 `UNUserNotificationCenter.current()`에 add하여 푸시 알림을 등록한다. 

추가적으로 `UNUserNotificationCenter` 가 여러 역할을 더 할 수 있는데, 이를 위해 delegate를 채택해줘야 한다. 

```swift
UNUserNotificationCenter.current().delegate = self
```

예를 들어, 앱이 foreground 에서 실행 중일 때 도착한 노티를 어떻게 처리할 것인지를 delegate에게 물을 수 있다. 

이러한 delegate 관련 메서드들은 app delegate에서 구현해주는 것이 좋을 것 같다. 특정 화면에 종속되는 것이 아니라 계층의 최상위에 해당하는 app delegate에 있어야 원하는 액션을 구현해주기도 편하고, 앱 실행시 가장 첫 부분이기에 정상적인 실행을 보장해줄 수 있을 것 같기도 하다. 

```swift
userNotificationCenter(_:willPresent:withCompletionHandler:)
```

위 메서드를 통해 알림이 도착할 때 앱이 foreground에 있는 경우 공유 사용자 알림 센터는 이 방법을 호출하여 알림을 앱으로 직접 전달한다. 이 방법을 구현하면 알림을 처리하고 앱을 업데이트하는 데 필요한 모든 작업을 수행할 수 있다. 완료되면 completionHandler 블록을 호출하고 시스템에서 사용자에게 알림을 보내는 방법을 지정합니다(있는 경우).

```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    completionHandler([.alert, .badge, .sound])
}
```
이렇게 completionHandler에 원하는 알림 종류를 보내주면 된다. 

```swift
userNotificationCenter(_:didReceive:withCompletionHandler:)
```

위 메서드는 전송받은 알림에 대한 사용자의 응답 처리 방법을 delegate에게 묻는다. 즉 알림을 받고 취할 액션을 여기서 정해주는 것이다. 

앞서 userinfo에 특정 데이터를 담아 보내는 알림을 생성했었다. 이를 받기 위해서는 해당 메서드의 response를 이용하면 된다.

```swift
let userInfo = response.notification.request.content.userInfo
```

지금처럼 foreground일 때는 문제 없지만, 앱이 종료되어 있는 상태에서도 알림을 받을 수 있다. 이 때는 어떤 경로로 앱을 오픈했는지가 중요할 수 있다. 

이 때 `UIApplication.LaunchOptionsKey`를 통해 didFinishLaunchingWithOptions 메서드에서 유입 경로를 확인할 수 있다. (app이 실행될 때 어떤 옵션으로 실행되었는지를 파라미터로 가지고 온다!)

현재 localNotification은 deprecated되었으며 아까 위에서 언급한 didReceive 메서드를 대체재로 사용하면 된다.


## 고민된 점 
- push notification을 보내는 방법

## 해결 방법 
- 공식문서... 또 공식문서...
---

**Ref**

https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate/1649518-usernotificationcenter

https://developer.apple.com/documentation/usernotifications/unusernotificationcenterdelegate/1649501-usernotificationcenter

https://developer.apple.com/documentation/uikit/uiapplication/launchoptionskey
