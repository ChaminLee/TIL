# no such module error, Localization, SwiftyDropbox

## 220221_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 퀴즈 복습!

- intrinsic size를 가지는 요소
	- UIbutton: width/height 존재
	-  UISlider: height 존재
	- UITextfield: width/height 존재
	- UITextview: `isSrollEnabled`가 true이면 없다. false일 때는 생김!
- Data Container의 Library는 사용자에게 노출되지 않는다. 
	- 앱 기능 관리에 필요한 파일 저장 
	- 유일 접근 가능한 곳은 documents이다
- `~> 0.1.2`의 의미 : 0.1.2 이상,  0.2 미만 포함 가능
	-  다음 마이너 전까지 가능하다는 것

### `no such Module ~` 에러 

Xcode에서 라이브러리가 import되지 않는 문제이다. 

cocoapods에서 라이브러리를 받았다고 했을 때, 연결에 문제가 있는 것이기 때문에 다음과 같이 해결해 볼 수 있다. 

진행할 때. xcode는 꺼져있는게 좋다.

1. `pod deintegrate` 
2. `podfile.lock` 제거
3. `pod repo update`
4. `pod install`
5. xcworkspace 파일 열고 빌드

이렇게 하면 무난하게 문제가 해결될 것이다! 

만약 해결이 안되었다..? 본인이 M1 맥북이다...?

Xcode를 우클릭해서 Rosetta로 열어서 다시 빌드해보자,
그럼 말끔하게 해결될 것이다!

### Localization 

스토리보드/코드로 구현하는 방법을 간단하게 과정 위주로 알아보자! 

#### 스토리보드
1. `Project -> Info`에서 하단의 언어 추가 
2. 스토리보드 타겟 추가 
3. `Main.strings (언어)` 에 접근하여 ObjectID에 매칭될 문자열 입력 
	
	```swift
	"kJK-lo-ifp.text" = "언어에 맞는 문자를 입력합니다";	
	```
4. `text`말고도, `accessibilityLabel` 등 원래 UI요소가 가지는 프로퍼티에 문자열 할당이 가능하다 

#### 코드 

1. `Localizable.strings` 파일 생성 
2. 우측 inspector의 `Localize...` 	버튼 클릭
3. 기본 언어 선택 후 `Localize` 버튼 클릭
4. 우측 inspector의 Localization에서 원하는 언어의 체크박스에 체크 표시 
5. `"key" = "value";` 형식으로 저장 및 사용된다. 
	```swift
	"Touch the button below!" = "아래 버튼 눌러라";
	```
	이런 식으로 원하는 텍스트 키에 해당하는 지역화된 문자열을 입력해준다. 
6. NSLocalizedString을 사용하는데 extension하면 편하다
	```swift
	// comment는 주석같은 역할
	extension String {
	    func localized(comment: String = "") -> String {
	        return NSLocalizedString(self, comment: comment)
	    }
	}
	// 사용
	"test Text".localized()
	```


> ## Tip.
> 1. App target > Edit scheme > Option > App language/region 바꿔주고 빌드하면 매번 시뮬레이터에서 언어/지역 설정을 바꿀 필요가 없다!
>
> 2. 스토리보드로 지역화 구현 방식에서 프로젝트에 언어를 추가할 때, 언어 별로 스토리보드를 다르게 생성해서 개발할 수 있다. 
>
> 3. 스키마 단축키: `cmd + shift + ,`
> 
> 4. Region을 바꾸다고 해서 시간이 바뀌는 건 아님, 그 지역에 맞는 포맷을 바꾸는 것임! 

### SwiftyDropbox

Dropbox에서 App console에 접근하여 app을 생성하게 된다. 

간단하게 따라해보자

#### 1. App console내 app 생성

![](https://i.imgur.com/kJmzV7F.png)

옵션은 목적에 맞게 선택하면 된다.

#### 2. 기본 설정

![](https://i.imgur.com/64o56hN.png)

이 화면을 보게 된다. 

파란색 박스에 적힌 것만 잘 따라하면 된다!

1. 초기 app setting을 해준다.
2. 접근 범위(access scope)를 `permissions` 탭에서 선택해준다.
3. `Branding`에서 앱 아이콘도 고르고, 이름을 정해준다.

3번은 왜 해야하나 싶긴 하겠지만... 실제로 방금 만든 앱을 배포하려면 `Apply for production`을 눌러 활성화 시켜야 한다. 그러기 위해서는 위 1~3을 필수적으로 해야한다. 

![](https://i.imgur.com/q1HITXn.png)

`permissions`에 와서 접근 권한을 정해주고 아래 광고바 같은 바에 있는 확인 버튼을 눌러준다. 

그리고 `Apply for production`을 누르면...

![](https://i.imgur.com/e1JHfEz.png)

앞서 말한 것 처럼 app icon을 등록하라고 나온다.. 시키는대로 잘 하자..!

![](https://i.imgur.com/flRdHNB.png)

고분고분하게 앱 아이콘을 추가해준다. 

이제 배포를.... 아직이다!

![](https://i.imgur.com/iX66hez.png)

아까 있었던 `enable additional users`를 눌러주자. 그러면 0/500으로 현재 권한을 얻어 사용하고 있는 사용자 수를 나타낸다. 

이제 `Apply for production`를 누르면 아래와 같이 리뷰를 기다린다고 뜨는데, 이제 아까 허용한 읽기/쓰기 등이 가능해진다. 

![](https://i.imgur.com/Ne4ABgn.png)

앱 프로덕션에 관한 내용은 [문서](https://www.dropbox.com/developers/reference/developer-guide#production-approval)에서 볼 수 있다. 

아무튼 이렇게 우여곡절을 겪고 app 세팅을 마쳤다면 이제 Xcode로 갈 자격을 얻게된 것이다.

읽고/쓰기 전에 우선 사용자의 계정을 dropbox에 연동하기 위해 연결을 먼저 해줘야 한다. 

```swift
private var client = DropboxClientsManager.authorizedClient

func connectDropbox(viewController: UIViewController) {
    let scopes = ["account_info.read", "files.content.write", "files.content.read"]
    let scopeRequest = ScopeRequest(scopeType: .user, scopes: scopes, includeGrantedScopes: false)
    DropboxClientsManager.authorizeFromControllerV2(
        UIApplication.shared,
        controller: viewController,
        loadingStatusDelegate: nil,
        openURL: { (url: URL) -> Void in UIApplication.shared.open(url, options: [:], completionHandler: nil) },
        scopeRequest: scopeRequest
    )
}
```

[SwiftyDropbox 문서](https://github.com/dropbox/SwiftyDropbox)에 따라 인증 권한을 얻도록 해준다. dropbox 앱이 있는 경우는 앱으로 리다이렉트 되며, 없는 경우는 사파리를 이용해 로그인 및 권한 확인 화면을 띄우게 된다. 

앱이 시작될 때 요청하기 보다는, 사용자의 선택에 따라 연결할 수 있게 버튼에 연결해줬다. 

그러면 사용자는 dropbox에 로그인하고 권한을 허용하면, 연결을 마치게 된다. 

그렇다면 연결을 마치면 다시 원래 화면으로 돌아와줘야 하는데, 해당 코드는 iOS 13 이상일 경우 `SceneDelegate`에 적어준다.

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
     let oauthCompletion: DropboxOAuthCompletion = {
      if let authResult = $0 {
          switch authResult {
          case .success:
              print("Success! User is logged into DropboxClientsManager.")
          case .cancel:
              print("Authorization flow was manually canceled by user!")
          case .error(_, let description):
              print("Error: \(String(describing: description))")
          }
      }
    }

    for context in URLContexts {
        if DropboxClientsManager.handleRedirectURL(context.url, completion: oauthCompletion) { break }
    }
}
```

인증 결과를 출력해주고, 다시 리다이렉트해주는 역할을 한다. 

그럼 이제 읽기/쓰기/업데이트/삭제를 위한 준비는 끝났다. 

이제 API를 호출하여 적절하게 사용해주면 된다! 

## 고민된 점 
- 지역화는 어떻게 구현해줄까?
- dropbox와 coredata를 어떻게 동기화해줄까?
- 적절한 데이터 동기화 시점은 언제인지, 구조는 어떻게 해야할지

## 해결 방법 
- 스토리보드/코드를 통한 지역화 방법 학습
- SwiftyDropbox 공식 문서 확인 및 코드 작성
- 최대한 효율적으로 작동하기 위해 coredata에서 관리하되, 앱이 종료되거나 편집을 마치거나 할 때 dropbox에 데이터를 쓰고, 앱 첫 실행시에만 dropbox에서 coredata로 데이터를 불러오도록 고민해봤다. 이를 위해 core data 관리 객체와 dropbox 관리 객체가 appdelegate 단에 위치해야할지에 대해서도 고민해봤다.
---

**Ref**

https://babbab2.tistory.com/59

https://www.dropbox.com/developers/reference/developer-guide#production-approval

https://github.com/dropbox/SwiftyDropbox
