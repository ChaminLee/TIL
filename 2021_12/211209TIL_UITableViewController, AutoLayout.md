# UITableViewController, AutoLayout

## 211209_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용


### 1. UITableViewController

Table View 사용을 조금 더 편리하게 해주기 위한 타입이다. 

```swift
open class UITableViewController : UIViewController, UITableViewDelegate, UITableViewDataSource {
```

보다싶이 `UITableViewDelegate`와 `UITableViewDataSource` 프로토콜을 기본적으로 채택하고 있는 것을 알 수 있다. 그리고 자동적으로 table view의 data source와 delegate에 `self`를 넣어준다.

또한 기본적으로 스토리보드나 nib 파일에 보관된 table view를 자동으로 로드한다. 그 덕분에 `tableView` 프로퍼티를 사용하여 table에 접근할 수 있다. 

`viewWillAppear(_:)` 메소드를 구현하고 처음 나타날 때 table view에 대한 데이터를 자동으로 다시 로드한다. 이 때 selection 상태도 초기화되기 때문에, 이를 원하지 않는다면 `clearsSelectionOnViewWillAppear` 프로퍼티의 값을 바꿔서 이를 막을 수 있다. 

`setEditing(_:animated:)` 메서드를 구현하여 사용자가 edit 버튼을 누르면 table의 편집 모드가 자동으로 전환된다. 

또한 키보드의 등장, 사라짐에 따라 table view의 크기를 자동으로 조정한다. 

### 2. Auto Layout 복습 

- Hugging Priority
	- 우선순위가 높으면 내 크기를 유지 
	- 우선순위가 낮으면 크기가 늘어남
	- 요소들 중 하나가 커져야 하는 경우
- Resistance Priority
	- 우선순위가 높으면 내 크기를 유지 
	- 우선순위가 낮으면 크기가 작아짐
	- 요소들 중 하나가 작아져야 하는 경우

- StackView를 사용하는 것이 조금 더 편리하다.

- 최대한 View가 계산을 덜 하는 방향으로 제약을 잡아주는 것이 좋다.

## 고민된 점 
- `UITableViewController`를 왜 써야할까?

## 해결 방법 
- 공식 문서를 읽고 내용을 정리
---

**Ref**

[uitableviewcontroller](https://developer.apple.com/documentation/uikit/uitableviewcontroller)

