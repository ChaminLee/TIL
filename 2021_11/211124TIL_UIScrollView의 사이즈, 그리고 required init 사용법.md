# UIScrollView의 사이즈, 그리고 required init 사용법

## 211124_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### UIScrollView의 사이즈 파헤치기

![](https://i.imgur.com/PmjrnZt.png)

#### `contentOffset`
```swift
var contentOffset: CGPoint { get  set }
```
`CGPoint`는 2차원 좌표계에서 x,y 좌표를 나타낸다. 
즉 `contentOffset`의 값을 변경시켜서 보여지는 화면을 이동시킬 수 있는 것이다. 마찬가지로 화면을 스크롤하면 `contentOffset` 값 또한 변경되게 되는 것이다.

#### `contentSize`

```swift
var contentSize: CGSize { get  set }
```

이는 콘텐츠의 크기를 결정하는 요소로 `CGSize`는 (width, height)로 이루어져 있는데, 이는 `UIScrollView` 내부 content의 가로,세로 크기를 말한다. 이 사이즈는 기기의 크기보다 더 커질 수 있게 된다. 

#### `bounds`
```swift
var bounds: CGRect { get  set }
```

`CGRect`는 `CGPoint`와 `CGSize`를 가지고 있다. 즉 현재 위치(x,y)와 크기(너비,높이)를 알고 있는 것이다. 

`bound`는 `frame`과 달리 자기 자신의 좌표를 가지고 있다.

-   `bound` : 자신만의 좌표 시스템안에서 view의 위치와 크기를 나타냄
-   `frame` : 상위 뷰의 좌표 시스템안에서 view의 위치와 크기를 나타냄

큰 액자속 작은 틀이 있다고 했을 때, bound를 변경하면 큰 액자가 움직이는 것(상위뷰가 움직여버림)과 같고, frame을 변경하면 작은 틀이 움직이는 것과 같다.

즉 `bounds`는 고정된 자신만의 좌표라고 볼 수도 있을 것 같다. `UIScrollView`의 `bounds`는 위와 같은 좌표 시스템을 따르고 너비와 높이를 가지고 있는 것이다. 

`bounds.origin`을 통해 `CGPoint`인 좌표를 얻거나 설정할 수 있고, `bounds.size`를 통해 `CGSize`인 크기를 얻거나 설정할 수 있다. 

#### `contentInset`
```swift
var contentInset: UIEdgeInsets { get  set }
```
`UIEdgeInsets`이란 뷰와의 inset 거리를 나타낸다. 즉 뷰 외부의 마진을 이야기하는 것이다. 

```swift
init(top: CGFloat, left: CGFloat, bottom: CGFloat, right: CGFloat)
```
이러한 `contentInset`의 경우 `UIScrollView`에 표시되는 `Content`의 외부 여백의 크기를 얻어올 수도 있고, 위, 왼쪽, 아래, 오른쪽 순서로 결정할 수도 있다. 
![](https://miro.medium.com/max/1026/1*3KfHKOkzwVQZtuXoIAZ7cg.png)

#### `UIScrollView` AutoScroll

처음에 말한 것 처럼 `contentOffset` 을 통해 보여지는 화면의 위치를 조정할 수 있었다. `contentOffset`을 변경하기 위해서는 `setContentOffset` 메서드를 사용할 수 있다. 

```swift
func setContentOffset(_  contentOffset: CGPoint, animated: Bool)
```

### `required init(coder: )`

`required init(coder: )`에서 기본적으로 제공해주는 `fatalError`는 최대한 지양하자. 앱이 가급적 꺼지지 않는 방향으로 개발해야하기 때문이다!

대신에 상위 타입의 이니셜라이저를 명시해줘야 한다.
```swift
required init(coder: NSCoder) {
   super.init(coder: coder)
}
```

## 고민된 점 
- `required init(coder: )`내 기본적으로 주는 `fatalError`를 써도 되는가?
- scrollView를 autoScroll할 때 사용되는 `contentSize`, `bounds`, `contentInset`, `contentOffset`의 차이는?

## 해결 방법 
- `fatalError` 사용을 지양하는 것이 좋다고 한다. 학습 내용 참고!
- `UIScrollView`의 사이즈 요소들 참조!

---

**Ref**
[ScrollView](https://jinshine.github.io/2018/12/05/iOS/ScrollView/)
[UIEdgeInsets](https://developer.apple.com/documentation/uikit/uiedgeinsets)
