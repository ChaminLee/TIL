# Sync/Async, Concurrency/Parallelism, DispatchQueue, Dynamic Type AutoLayout

## 211216_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용


### 1. 동기 vs 비동기 

> 1 . 동기 (Sync)

Synchronous라는 단어는 "동시에 일어나는"이라는 의미를 가지고 있다. 즉 요청과 요청에 대한 결과가 동시에 발생한다는 의미이다. 

어떠한 작업을 시키고 바로 연이어 다음 작업을 시키는 것이 아니라, 이전 작업이 끝나야 다음 작업을 시킬 수 있다고도 이해해 볼 수 있다.

즉 하나의 일만 처리할 수 있다고 봐도 될 것 같다.  

동기의 단순하지만, 앞의 작업을 기다려야 하기 때문에 대기 시간에 대한 비용이 발생하게 된다. 

> 2. 비동기 (Async)

Asynchronous라는 단어는 "동시에 일어나지 않는"이라는 의미를 가진다. 즉 요청과 결과가 무조건 같이 일어나지 않을 거라는 것이다.

이에 동기와 다르게 어떠한 일을 시키고 해당 작업이 끝나기 전에도 새로운 작업을 시킬 수 있다. 

비동기의 경우 복잡하지만, 요청 이후 결과를 기다리는 동안에도 다른 작업을 수행할 수 있어 조금 더 효율적이라고 볼 수 있다.

### 2. 동시성 vs 병렬 프로그래밍

> 1 . 동시성(Concurrency) 프로그래밍

말처럼 동시에 무언가가 처리될 수 있는 것 처럼 보인다. 하지만 실제로는 작업을 동시에 처리하는 것이 아니라, 짧은 시간동안 왔다갔다 하면서 여러 일을 처리하고 있다고 한다. 즉, 동시에 처리되는 것 처럼 보이도록 하는 것이다. 

그래서 동시성 프로그래밍은 싱글 코어에서 멀티 스레드를 사용하는 경우를 말한다. 동시에 여러가지 일을 수행할 수 없기 때문에 A일을 했다가 B일을 했다가 C일을 했다가 계속 번갈아가며 작업을 수행한다. 이러한 과정에서 context switching(문맥 교환)이 발생하게 되는데, 이 점을 단점으로 꼽을 수 있을 것 같다. 

앞서 말한 것 처럼 실제로 동시에 작업하는게 아닌데, 그렇게 보이기 때문에 논리적인 개념에 가깝다고 한다. 


> 현재 컴퓨터는 동시성 프로그래밍 환경이라고 봐도 무방하다. 

또한 멀티코어 환경이라고 해서 동시성 프로그래밍이 불가한 것은 아니다. 멀티 코어 내부에서도 동시성 프로그래밍의 방법이 사용될 수 있다. 
정리하자면 다음과 같다. 

- 동시에 처리되는 것 처럼 보이게 하는 것이다. 
- 싱글 코어에서 멀티 스레드를 사용한다
- 멀티 코어에서도 동시성 가능 
- 논리적인 개념


> 2 . 병렬성(Parallelism) 프로그래밍 

동시성과 달리 진짜로 동시에 작업을 처리해준다. 멀티 코어 환경에서 멀티 스레드를 사용하여 하나의 일을 여러 코어가 수행하도록 해준다. 이에 앞서 봤던 동시성 프로그래밍과 달리 물리적인 개념으로서 작동한다. 

병렬성 환경에서 어떠한 무거운 작업을 시키게 되면 여러 코어들이 이 일을 나눠서 하게 된다.

정리해보면 다음과 같다. 

- 실제로 작업을 동시에 처리한다.
- 멀티 코어에서 멀티 스레드를 사용한다.
- 물리적인 개념 

### 3. DispatchQueue

앱의 메인 쓰레드 혹은 백그라운드 쓰레드에서 작업을 직렬(serial)적으로 혹은 동시(concurrent)적으로 실행할지 관리해주는 객체이다. 

DispatchQueue의 큐는 우리가 아닌 FIFO의 특성을 갖는 큐이다. 해당 큐에는 블록 객체 형태로 작업을 넣어줄 수 있다. DispatchQueue는 작업을 순차적(serially) 혹은 동시에(concurrently) 실행한다. DispatchQueue에 등록된 작업들은 시스템에 의해 관리되는 thread pool에서 실행된다. 이 때 메인 쓰레드를 나타내는 `DispatchQueue.main`를 제외하고는 시스템이 어떤 쓰레드를 사용하여 작업을 실행할지 보장하지 않는다. 

이 때 serially 하게 처리하는 것은 `DispatchQueue.main`, concurrently하게 처리하는 것은 `DispatchQueue.global()`을 의미한다. 앞서 말한 것 처럼 `DispatchQueue.global()`에서는 어떤 쓰레드를 몇 개 사용할지 보장하지 않는다. 

여기서 동기/비동기 개념도 적용해 볼 수 있다. 만약 작업을 동기적으로 처리하도록 한다면 해당 작업이 끝날 때 까지 코드는 기다리게 될 것이다. 반대로 작업 항목을 비동기적으로 처리하도록 한다면 해당 작업이 실행중임에도 다른 코드도 계속 실행이 된다. 

> #### 중요
> 메인 큐에서 작업들을 동기적으로 처리한다면 교착상태(deadlock)가 발생한다. `DispatchQueue.main.sync`

한 번 작성해보면서 살펴보자. 

만약 `DispatchQueue.global().async` 내에 두 줄의 코드를 적었다면, 동시에 작업이 배분은 되나 무엇이 먼저 실행될지 모른다.

```swift
for i in 1...3 {
	DispatchQueue.global().async {	
		print("\(i) 시작")
		print("\(i) 끝")
	}
}
```

실행 결과를 한 번 봐보자.

```swift
3 시작
1 시작
2 시작
1 끝
3 끝
2 끝
```

분명 시작/끝을 출력하는 코드는 하나의 task로 인식되지만 출력된 결과를 보면 붙어있지 않다. 즉 하나의 task로 인식되어 시작은 되지만, 뒤의 작업들이 기다리지 않고 바로 실행되기 때문에 이런 결과를 볼 수 있는 것이다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FPBuNZ%2Fbtrn15ZYQ5A%2F0squVkdfi81kI6qMaxkjGK%2Fimg.png)

하지만 확실한 것은 무조건 `n 시작`이 먼저 출력되고 해당되는 `n 끝`이 그 뒤에 출력된다는 것이다. 

> main thread의 일이 종료되면 프로그램이 종료됨. 즉 다른 thread가 실행될 수 없게 된다. 
> (+) Playgrounds에서는 main 스레드가 계속 돌고 있어서 main 스레드가 계속 실행된다고 한다. 

반대로 예상한 것 처럼 `DispatchQueue.global().sync`은 작업을 순차적으로 순서를 지키면서 실행하게 된다. 

```swift
for i in 1...3 {
	DispatchQueue.global().sync {	
		print("\(i) 시작")
		print("\(i) 끝")
	}
}
```

예상된 결과...!를 살펴보자!

```swift
1 시작
1 끝
2 시작
2 끝
3 시작
3 끝
```

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcKUswY%2Fbtrn4qJjlpj%2F8j8Sa2os88bgc0wPZwT971%2Fimg.png)

그림으로 봐도 순차적으로 순서를 지키면서 작업을 수행하고 있음을 알 수 있다. 

`DispatchQueue.global(qos: )`를 통해 우선순위를 정해줄 수도 있지만, 그것보다 작업을 동기/비동기로 처리해야하는지 잘 정하는 것이 중요할 것 같다. 

뿐만 아니라 작업이 처리되어야하는 순서나 그 중요성에 기반하여 동기/비동기를 선택하는 것또한 중요할 것 같다.

### 4. 프로세스 vs 프로세서 vs 스레드

- 프로세스
	- 현재 실행중인 프로그램
    - 메모리에 올라와 실행되고 있는 프로그램의 인스턴스
	- 모든 메모리 영역을 독자적으로 가짐
- 프로세서
	- CPU
- 스레드
	- 프로세스 내의 여러 실행 흐름 단위
	- stack 영역을 독자적으로 가짐, 힙 공간 공유



- 멀티 프로세스가 아닌 멀티 스레드를 사용하는 이유?
	- 공유자원을 적극 활용하기 위해 사용한다
	- 프로세스 생성하는 시스템 콜이 줄어든다


- 32 bit vs 64 bit
	- 한 번에 처리할 수 있는 bit의 수를 의미한다
	- 32bit는 4GB RAM만 사용 가능
		- 표현 가능한 최대치가 2^32(4GB)이기 때문에
	- 레지스터의 크기 차이 

### 5. Dynamic Type AutoLayout 적용하기

[WWDC 2017 Building Apps with Dynamic Type](https://developer.apple.com/videos/play/wwdc2017/245/)를 보다가 마침 현재 프로젝트에도 있는 문제가 있어서 적용 가능해보이는 부분을 발견했다.

![](https://a11y-guidelines.orange.com/en/mobile/images/iOSdev/wwdc17-245-SideBySideText.png)

바로 `UILabel`이 나란히 2개 있는 상황에서 기기의 폰트 크기가 커짐에 따라 어떻게 레이아웃을 잡아야하는지에 대한 내용이었다. 

당연히 텍스트 일부가 `...`등으로 짤리면 안되고, 뭔가 텍스트의 줄바꿈이 이상하게 되어있어도 안된다고 이야기하고 있다. 딱 봐도 이상해보이긴 한다... 

그래서 권장하는 방식은 텍스트 크기가 커졌을 때, 우측 `UILabel`을 좌측 `UILabel` 아래로 내리는 방법이다! 

WWDC에서는 아래와 같은 방법으로 구현할 수 있다고 이야기하고 있다.

![](https://a11y-guidelines.orange.com/en/mobile/images/iOSdev/wwdc17-245-AutoLayoutsystemSpacingConstraints_2.png)
우측 `UILabel`의 firstBaseline을 좌측 `UILabel` lastBaseline에 맞추는데, 바로 아래 딱 붙게해줄 수 있도록 `constriantEqualToSystemSpacingBelow`를 주고 있는 것을 볼 수 있다. 

근데 [공식문서](https://developer.apple.com/documentation/uikit/nslayoutyaxisanchor/2866022-constraint)를 보면 아래와 같이 사용하고 있는 것 같다. 

```swift
constraint(equalToSystemSpacingBelow:multiplier:)
```

괄호의 위치가 조금 다른 느낌...?

아무튼 이렇게 레이아웃을 주면 vertical하게 위치시킬 수 있다고 한다. 이제 어떻게 시스템 폰트 크기를 식별하느냐인데, 이 또한 WWDC에서 알려주고 있다. 

![](https://a11y-guidelines.orange.com/en/mobile/images/iOSdev/wwdc17-245-PreferredContentSizeCategory_2.png)

두 가지 방법이 있는데, 하나는 `isAccessibilityCategory`를 사용하는 방법, 다른 하나는 특정 글자 크기를 지정해서 그걸 넘어서는지 확인하는 방법이다. 

`isAccessibilityCategory`의 문서를 보면 
[`accessibilityMedium`](https://developer.apple.com/documentation/uikit/uicontentsizecategory/1623110-accessibilitymedium), [`accessibilityLarge`](https://developer.apple.com/documentation/uikit/uicontentsizecategory/1623050-accessibilitylarge), [`accessibilityExtraLarge`](https://developer.apple.com/documentation/uikit/uicontentsizecategory/1622959-accessibilityextralarge), [`accessibilityExtraExtraLarge`](https://developer.apple.com/documentation/uikit/uicontentsizecategory/1623120-accessibilityextraextralarge)와 [`accessibilityExtraExtraExtraLarge`](https://developer.apple.com/documentation/uikit/uicontentsizecategory/1623086-accessibilityextraextraextralarg) 만 true의 값을 반환한다고 한다. 

그래서 우선은 이를 활용해서 레이아웃을 따로 잡아주었다. 

자 그러면 한 번 구현해보자!!

우선 레이아웃을 담아줄 변수를 생성하자.

```swift
var LargeFontTypeLayout: [NSLayoutConstraint]?
var standartFontTypeLayout: [NSLayoutConstraint]?
```

`isAccessibilityCategory` 여부에 따라 나뉘는 두 가지 케이스에 맞게 레이아웃을 잡아주기 위해 두 가지 레이아웃 배열을 만들어줬다. 

각 케이스에서만 특수하게 적용해줄 레이아웃들을 각 배열에 담아준다. 공통으로 적용되는 레이아웃들은 굳이 배열에 담지 않고도 바로 적용시켜줘도 된다. 

```swift
LargeFontTypeLayout = [
    second.firstBaselineAnchor.constraint(equalToSystemSpacingBelow: first.lastBaselineAnchor, multiplier: 1),
    second.leadingAnchor.constraint(equalTo: first.leadingAnchor)
]

standartFontTypeLayout = [
    second.centerYAnchor.constraint(equalTo: first.centerYAnchor), 
    second.leadingAnchor.constraint(equalTo: first.trailingAnchor, constant: 10)
]
```
자 이제 각 케이스에 따라 담아준 레이아웃 배열을 active시켜주면 된다. 

```swift
if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
	LargeFontTypeLayout?.forEach {
		$0.isActive = true
	}
} else {
	standartFontTypeLayout?.forEach {
		$0.isActive = true
	}
}
```

자 여기까지 잡아준 레이아웃 코드를 메서드로 잘 정리해서 `viewDidLoad()`에 넣어주면 앱이 처음 실행되었을 때 기기의 폰트 크기에 따라 잘 분기가 될 것이다. 

하지만 중간에 앱을 백그라운드로 보내고, 기기의 폰트 크기를 바꾸게 되면 현재 코드로서는 대응하지 못하게 된다. 

그렇기에 추가해줘야하는 코드가 있는데, 바로 기기의 폰트 설정이 바뀌었다는 정보를 받기 위해 NotificationCenter에 `addObserver`를 해줘야 한다. 

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    NotificationCenter.default.addObserver(self, selector: #selector(setLayoutByDynamicType), name: UIContentSizeCategory.didChangeNotification, object: nil)
}

// setLayoutByDynamicType
@objc func setLayoutByDynamicType() {
    if traitCollection.preferredContentSizeCategory.isAccessibilityCategory {
        standartFontTypeLayout?.forEach {
            $0.isActive = false
        }
        LargeFontTypeLayout?.forEach {
            $0.isActive = true
        }    
    } else {
        LargeFontTypeLayout?.forEach {
            $0.isActive = false
        }
        standartFontTypeLayout?.forEach {
            $0.isActive = true
        }
    }
}
```

시스템에서 폰트 카테고리가 바뀌었을 때 post를 주는데 그 채널은 `UIContentSizeCategory.didChangeNotification`이다. 이에 기본적으로 제공해주는 채널에 대해 `addObserver`를 해두고 알림을 받았을 때 실행할 메서드를 작성해준다. 

다른 것은 없고 두 케이스의 레이아웃 배열을 스위치 온/오프 하듯이 원하지 않는 레이아웃을 deactive해주고, 원하는 레이아웃을 active해주면 된다. 

자 이제 결과물을 확인해 볼 시간...!

![](https://im2.ezgif.com/tmp/ezgif-2-12800c662c.gif)

이제 텍스트 크기에 맞게 레이아웃이 자동으로 적용되는 것을 볼 수 있다. 

다 하고 알게된 사실인데... 만약 저 두 개의 `UILabel`을 `UIStackView`로 묶어도 되는 상황이라면 더 쉽게 해결해 볼 수 있다... 

글자가 커지는 경우 `.axis`만 `vertical`로 바꿔주고, 기본인 경우 다시 `horizontal`로 바꿔주면 된다. 

뿐만 아니라 필요하고, 필요하지 않는 레이아웃을 아까 봤던 방법처럼 active/deactive해서 적용해주면 더 간단하게 해결해 볼 수도 있을 것 같다. 

## 고민된 점 
- Dynamic Type에 따른 AutoLayout을 어떻게 조정할까?
- 동기/비동기, 동시성/병렬성의 차이

## 해결 방법 
- WWDC 세션, 공식문서 참고!
---

**Ref**


https://developer.apple.com/documentation/dispatch/dispatchqueue

https://developer.apple.com/documentation/dispatch/dispatchqueue/1781006-main

https://developer.apple.com/documentation/uikit/nslayoutyaxisanchor/2866022-constraint

https://developer.apple.com/documentation/uikit/uicontentsizecategory/2897444-isaccessibilitycategory
