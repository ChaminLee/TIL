
# Code Snippets, Device Orientation, UIStackView alignment/distribution simulator, #/@ available

## 211214_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. Code Snippets 만들기

Xcode를 통해 코딩을하다보면 자연스레 기본적으로 제공해주는 자동완성을 많이 사용하게 된다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F3kUlX%2FbtrnPKtVQzr%2FIk8IyBcEodhERsXRGkIPj0%2Fimg.png)

이렇게 뜨는 박스에서 원하는 코드를 선택해서 사용한다. 

앞에 붙어있는 박스 안의 알파벳은 `Code Sense`라고 하는데 그 종류에 대한 설명은 [Xcode Code Sense 종류](https://github.com/ccmoret/XcodeCS)에 잘 설명되어 있다. 

우리는 그 중에서도 [![](https://github.com/ccmoret/XcodeCS/raw/master/images/snippets.png "snippets")](https://github.com/ccmoret/XcodeCS/blob/master/images/snippets.png) 로 나타나는 `Code Snippets`에 대해 알아볼 것이다. 

단어의 뜻 그대로 코드 조각이라는 의미를 가지고 있다. 예를 들어 앞서 봤던 if문 박스의 첫 번째 자동완성을 클릭하면 아래와 같이 코드가 완성된다. 

```swift
if condition {
	code
}
```
조금 더 편리하게 사용할 수 있도록 제공하는 것 같다. 이러한 code snippets은 사용자가 만들 수도 있다. 

MARK 주석을 달기 위해 매번 `// MARK: - `를 입력해주는데 이를 한 번에 하고 싶으니 code snippets으로 등록해보자. 

우선 원하는 코드를 Xcode에 입력한다. 

```swift 
// MARK: - <#Element#>
```

> #### Tip
> `<#Element#>`로 적으면 회색 음영의 placeholder가 된다! 적어줘야할 요소의 위치를 나타내기 적합하다!

그리고 드래그 한 뒤에, 우클릭하면 `Create Code Snippet`을 누르면 `Library` 창이 뜬다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbausEn%2FbtrnOPoC617%2FlzSAYTeOkhcVmyDzT6tsI1%2Fimg.png)

이제 snippet의 원하는 이름을 입력하고 코드를 추가로 더 입력해 줄 수도 있다. 

그리고 제일 중요한 `Completion` 쪽에 원하는 단어를 적으면 실제로 해당 단어를 입력했을 때 자동 완성이 뜨게 된다. 만약에 `mark`라고 했다면, `mark`라고 입력했을 때 자동완성이 뜨게 되는 것이다! 

이제 입맛에 맞게 자주 사용하는 코드들은 Code Snippets으로 만들어서 사용하면 생산성이 조금 올라갈 것 같은 기분이다!

### 2. Device Orientation

iPhone을 요리조리 뒤집어보고 돌려보면 앱의 화면이 기기의 방향에 맞게 조정되는 경험을 해본 적이 있을 것이다. 

과연 이게 그냥 다 되는걸까... 아무것도 몰랐을 때는 아무 생각없이 뒤집고 이리저리 봤지만, 이제보니 다 노력의 산물이라는게 느껴진다... 아무튼 이렇게 기기의 방향에 따라 화면의 방향도 동일하게 지원해주려면 알아야 하는 것들이 있다. 

- `supportedInterfaceOrientations`
- `shouldAutoRotate`

먼저 `supportedInterfaceOrientations`를 보자. 

#### 2-1. `supportedInterfaceOrientations`

View Controller가 지원하는 인터페이스의 방향을 말한다. 

```swift
var supportedInterfaceOrientations: UIInterfaceOrientationMask { get }
```

`UIViewController`의 인스턴스 프로퍼티로 view controller가 지원하는 방향을 특정하는 비트 마스크를 반환한다. `UIInterfaceOrientationMask`는 그렇다면 뭘까... 좀 더 깊이 들어가보자. 

`UIInterfaceOrientationMask`는 view controller의 지원되는 인터페이스 방향을 나타내는 상수이다. 그렇다면 지원되는 방향들은 어떤게 있을까? 

총 7개로 다음과 같다. 


![](https://t1.daumcdn.net/cfile/tistory/232C824B5524F4CC04)
- `portrait` : 기본적인 세로 모드이다. 
- `landscapeLeft` : 오른쪽으로 눕혔을 때 가로 모드
- `landscapeRight` : 왼쪽으로 눕혔을 때의 가로 모드
- `portraitUpsideDown` : 세로 모드이지만 뒤집힌 상태
- `landscape` : 위의 가로모드 두 가지를 지원한다. 
- `all` : 위 4가지 모드를 모두 지원한다. 
- `allButUpsideDown` : `portraitUpsideDown` 빼고 모두 지원한다. 

자 이제 어떤 방향들이 있는지 확인했으니 다시 돌아와보자. 

기기의 방향이 바귀게 되면 시스템은 root view controller나 window를 채우는 최상위 modal view controller에서 이 메서드를 부르게 된다. 만약 view controller가 새로운 방향을 지원한다면, 시스템은 윈도우와 view controller를 회전시킨다. View controller의 `shhouldAutorotate` 메서드가 true를 반환한다면, 시스템은 오직 이 메서드만 호출한다.

> #### `shouldAutorotate`
> `var  shouldAutorotate: Bool { get }`와 같이 구현되어있다. Boolean 타입으로 view controller의 컨텐츠가 자동회전되어야 하는지 여부를 나타낸다. 

View controller가 지원하는 방향을 선언하려면`supportedInterfaceOrientations`를 오버라이드하면 된다. iPadOS에서 기본값은 `all`이고 iOS에서는 기본값이 `allButUpsideDown`이다. 그리고 반환하는 값이 0이면 절대 안된다!

이제 회전 여부를 결정하기 위해 시스템은 view controller의 지원되는 방향(supported orientation)을 앱의 지원되는 방향과 디바이스의 지원되는 방향을 비교한다. 보통 `Info.plist` 파일이나 app delegate의 `application(_:supportedInterfaceOrientationsFor:)` 메서드를 통해 결정된다. 이 메서드를 구현하지 않는다면 앱의 `Info.plist`의 default interface orientations를 참고한다고 한다. 

> #### 왜 `portraitUpsideDown`이 있을까?
> iPadOS는 `portraitUpsideDown`를 지원해준다. 하지만 iPhone 12와 같이 홈 버튼이 없는 iOS 기기는 이 방향을 지원하지 않는다. 즉 X이전의 기기들은 지원이 된다는 말이다. 하지만 문서에서는 iPhone에 대해서 이 방향을 비활성화시켜야 한다고 한다!

자 이제 코드를 적어 실제로 기기의 방향을 돌려보자. 앞서 말했던 것 처럼 `supportedInterfaceOrientations`를 재정의하여 기기의 방향을 정해줄 수 있다. 

```swift
// portrait 모드만 지원 
override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
	return .portrait
}
```

원하는 모드가 여러 개라면 위의 옵션을 참고하거나, 배열로 구성하여 반환해줘도 된다.

하지만 단일 view controller에 대해서는 기기의 방향을 잘 제어할 수 있지만 상위의 `UIViewController`가 있는 경우에는 상위의 컨트롤러에서 재정의해줘야 한다. 왜냐하면 하위의 view controller에서 재정의해줘도 상위 view controller의 속성을 따르기 때문이다.

예를 들자면, `UIViewController`를 상속하는 `UINavigationController`내의 `ViewController`의 관계와 같은 상황이다. 

아무리 `ViewController`에서 재정의 해줘도 `UINavigationController`의 속성을 따르기 때문에, 상위에서도 재정의를 해줘야 한다.

```swift
class CustomNavigationController: UINavigationController {
	override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
		return self.topViewController?.supportedInterfaceOrientations ?? .all
	} 
}
```

이렇게 상위 controller에서 정의해주고 하위 controller에서는 입맛에 맞게 사용할 수 있다. 

```swift
// 첫 번째 VC
class FirstViewController: UIViewController {
	override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
		return .portrait
	}
}

// 두 번째 VC
class SecondViewController: UIViewController {
	override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
		return .landscape
	}
}
```

### 3. UIStackView alignment/distribution

이번에 auto layout을 잡으면서 `UIStackView`의 distribution과 width에 대한 제약에 충돌이 생긴 적이 있었다. 이러한 상황을 마주하니... 아직도 `UIStackView`의 alignment와 distribution에 대해 잘 알고 있지 못한 것 같다는 생각이 들어서 다시 정리해보려고 한다!

![](https://docs-assets.developer.apple.com/published/82128953f6/uistack_hero_2x_04e50947-5aa0-4403-825b-26ba4c1662bd.png)

우선 Horizontal stack view를 기준으로 alignment는 세로축 정렬, distribution은 가로축 정렬을 담당하고 있다고 봐도 될 것 같다. 

#### 3-1. Alignment

> 1. `fill`

Stack View의 축에 따라 Stack View의 가로, 세로를 꽉 채워준다. 

> 2. `leading`

Stack View를 기준으로 좌측 정렬한다. 다만 축이 horizontal일 때는 `top`과 같아진다

> 3. `trailing`

Stack View를 기준으로 우측 정렬한다. 다만 축이 horizontal일 때는 `bottom`과 같아진다

> 4. `top`

Stack View를 기준으로 위로 정렬한다. 다만 축이 vertical일 때는 `leading`과 같아진다

> 5. `bottom`

Stack View를 기준으로 아래로 정렬한다. 다만 축이 vertical일 때는 `trailing`과 같아진다

> 6. `center`

Stack View의 축을 기준으로 중앙 정렬해준다. 

> 7. `firstBaseline`

Stack View 내의 뷰들을 first base line에 맞춰준다. 

> 8. `lastBaseline`

Stack View 내의 뷰들을 last base line에 맞춰준다. 

#### 3-2. Distribution


> 1.  `fill`

StackView 축(axis)의 가능한 공간을 채우기 위해 내부의 뷰(arranged view)들의 크기를 조정해준다. 만약 stack view 내부 뷰의 크기가 stack view에 맞지 않다면   compression resistance 우선순위에 따라 뷰가 줄어들게 된다. 만약 stack view의 공간이 채워지지 않았다면, hugging 우선순위에 따라 뷰를 늘려서 채워주게 된다. 만약 조금 애매한 부분이 있다면, 내부 뷰들이 속해있는 `arrangedSubviews` 배열의 Index에 기반하여 뷰의 크기를 조정해준다. 

한 마디로 정리하자면 stackView를 채우기 위해 내부 컨텐츠를 resistance/hugging priority에 따라 줄여주고, 늘여주는 옵션이다!

> 2.  `fillEqually`

StackView 내부를 동일한 크기로 리사이즈하여 채워넣어주는 옵션이다. 

> 3. `fillProprotionally`

뷰는 intrinsic content size에 기반하여 리사이징되어 stackview를 채우게 된다. 

> 3. `equalSpacing`

만약  arranged view들이 stack view를 채우지 못하는 경우 뷰들 사이에 공간을 넣어주게 된다. 

> 4. `equalCentering`

Stack view내 뷰들의 center 사이 공간을 일정하게 만들어준다. 

자 이제 이 모든 요소들을 조합해서 결과를 보기 위해 StackView Simulator를 만들어보자. 모든 alignment와 distribution 그리고 axis까지 선택하도록 하여 보여지는 모습들을 관찰해보자. 


![](https://im6.ezgif.com/tmp/ezgif-6-c32a20c9c318.gif)

하나하나 조합을 보기에는 너무 종류가 많다보니.. 단일 요소들이 어떤 역할을 하는지 잘 파악하고 있으면 문제없이 생각하는대로 잘 사용할 수 있을 것 같다. 

### 4. `#available` vs `@available`

간단하게 정리해볼 수 있다. 

- `#available` : OS의 버전, 플랫폼을 체크하기 위해 주로 사용한다. 
- `@available` : class나 method 앞에 주로 사용하여 특정 타겟(OS, 버전)에서만 동작가능하다는 것을 알려준다.

```swift
// iOS 버전 체크 코드
if #available(iOS 14.0, *) {
	// 14.0 이상 지원 코드 작성
}

// OS 플랫폼 체크
#if os(macOS) 
	return "MAC"
#elseif os(iOS)
	retrurn "iOS"

// 특정 플랫폼의 버전에서만 작동가능하다는 것을 표시
@available(iOS 14.0, *)
func doSomething() {
	// iOS 14.0 이상 지원 기능
}
```

## 고민된 점 
- 기기의 방향에 따라 어떻게 회전시키는지?
- `UIStackView`의 속성과 layout
- Factory pattern 생성

## 해결 방법 
- 공식문서를 찾아보고 프로젝트에 적용
- 실제로 `UIStackView`의 속성을 바꿔보는 시뮬레이터 생성
- Factory pattern 학습 및 프로젝트 내 View controller factory 생성 진행
---

**Ref**

[Xcode Code Sense](https://github.com/ccmoret/XcodeCS/blob/master/images/macro.png)

[Custom Code Snippets](https://sarunw.com/posts/how-to-create-code-snippets-in-xcode/)

[supportedInterfaceOrientations](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621435-supportedinterfaceorientations)

[iOS, 특정 화면만 방향 고정시키기](https://odong-tree.github.io/ios/2020/12/27/supportedInterfaceOrientations/)

[UIStackView](https://developer.apple.com/documentation/uikit/uistackview)

