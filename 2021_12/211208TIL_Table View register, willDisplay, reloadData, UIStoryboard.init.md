# Table View register, willDisplay, reloadData, UIStoryboard.init

## 211208_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### `register(_:forCellReuseIdentifier:)`

`register` 메서드는 `cellClass`나 `nib`과 `Identifier`를 묶어주기만 하는 단순 등록을 위한 메서드이다. 이 때는 `Cell`을 생성하지 않고 등록만 한다.

즉 그렇기 때문에 보통 처음에 `viewDidLoad` 단계에서 `register`를 한다고 해서 cell이 생기는 것은 아니다!

###  `tableView(_:willDisplay:forRowAt:)`

table view가 delegate에게 특정 행에 대한 cell을 그려야 한다고 알려주는 역할을 한다. 
```swift
optional func tableView(_ tableView: UITableView, 
		willDisplay cell: UITableViewCell, 
		forRowAt indexPath: IndexPath)
```

cell을 행에 그리기 직전에 수행되기에, 직접 cell을 구성하는 `cellForRowAt`보다 나중에 호출된다. 그리기 직전에 delegate는 알림을 받기 때문에 cell 객체를 수정할 수 있다. 그래서 `willDisplay` 메서드가 호출될 때 cell 선택 및 배경색과 같은 table view가 이전에 정한 상태 기반의 특성을 다시 지정할 수 있다.
(prefetch 도 함께 알아보기.)

### `reloadData()`

table view의 row와 section을 모두 다시 채운다. 

table, cell, section의 header, footer, index 배열등 에서 쓰이는 데이터를 다시 채우기 위해 이 메서드를 사용한다. 효율성을 위해, table view는 보여지는 row만 다시 채워서 보여주게 된다. 다시 채우는 과정에서 table이 축소되는 경우 offset을 조정하게 된다. table view의 delegate나 dataSource는 table view의 데이터가 완전히 다시 채워지길 원할 때 이 메서드를 부른다.

특히 `beginUpdates()`나 `endUpdates()`에 대한 호출로 구현된 애니메이션 블록 내에서 row를 추가하거나 삭제하는 메서드에서는 `reloadData()` 메서드가 호출되서는 안된다. 

### `UIStoryboard.init(name:bundle:)`

지정한 리소스 파일에 대한 스토리보드 객체를 만들고 반환한다. 

```swift
init(name: String, bundle storyboardBundleOrNil: Bundle?)
```

- name: storyboard의 파일 이름 (확장자 제외)
- bundle: 스토리보드 파일과 이에 연관된 자원을 포함하는 번들을 의미한다. `nil`로 지정할 경우 이 메서드는 현재 응용 프로그램의 main bundle에서 찾게 된다. 





## 고민된 점 
- table view의 life cycle

## 해결 방법 
- table view에 연관된 메서드들의 공식문서를 살펴보았다. (+ delegate 메서드 다 찍어보고 파악해보면 좋을 것 같다.)
---


**Ref**
[register](https://developer.apple.com/documentation/uikit/uitableview/1614888-register)

[willDisplay](https://developer.apple.com/documentation/uikit/uitableviewdelegate/1614883-tableview)

[reloadData](https://developer.apple.com/documentation/uikit/uitableview/1614862-reloaddata)

[UIStoryboard.init](https://developer.apple.com/documentation/uikit/uistoryboard/1616216-init)



