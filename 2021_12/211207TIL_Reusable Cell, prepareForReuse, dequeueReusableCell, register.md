# Reusable Cell, prepareForReuse, dequeueReusableCell, register

## 211207_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용


### TableView Reuse Cell

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcrRnNS%2FbtraZLFv3Wr%2F1XQwBkj3kATQVXvKlYmlIK%2Fimg.png)

1. 가장 상단의 셀이 화면을 벗어나게 되면서 reuse될 cell로 선택되게 된다.  
2. 재사용 queue에 cell을 넣는다. (queue는 FIFO)  
3. 재사용 queue에서 dequeue한다.  
4. cell은 `prepareForReuse`로 보내져서 내부 구성요소들을 초기화해준다.  
(재사용을 위한 초기화하지 않으면 이전 셀의 구성요소/정보들을 그대로 가져와서 원치 않은 값들을 보여주게 된다)  
5. 다시 `cellForRowAt`의 해당하는 indexPath로 보내진다.  
(여기서 다시 원하는 데이터를 넣어주고 보여주게 된다)  
6. 최하단의 cell로 다시 등장한다.

중요한 점은 재사용 cell을 사용할 때 원치 않는 데이터가 보여지는 것을 방지하기 위해 사전에 초기화해야 한다는 것!

### `prepareForReuse()`

`UITableViewCell`의 인스턴스 메서드로 재사용 cell을 table view의 delegate가 재사용하기 위한 준비를 해준다. 

만약 `UITableViewCell`이 reuse identifier를 가지고 있다면 `prepareForReuse()`는 tableview의`dequeueReusableCell(withIdentifier:)` 메서드가 객체를 반환하기 이전에 실행된다. 잠재적인 성능 이슈를 피하기 위해, 컨텐츠와 관련이 없는 속성들만 리셋해야한다. 예를 들어, alpha, editing, selection state등이 있다.  `tableView(_:cellForRowAt:)`내 table view의 delegate는 cell을 재사용할 때 항상 모든 요소들을 리셋하여 사용해야한다. 

`UITableViewCell`이 reuse identifier를 가지고 있지 않다면 `prepareForReuse()` 메서드는 실행되지 않는다. 그럼에도 cell을 업데이트 하고 싶은 경우 `reconfigureRows(at:)` 메서드를 통해 기존 cell의 컨텐츠들을 업데이트 할 수 있다. 

### `dequeueReusableCell(withIdentifier:for:)`

특정 reuse identifier에 맞는 재사용 가능한 table view cell 객체를 반환하고 table에 추가해준다. 

해당 메서드는 `tableView(_:cellForRowAt:)` 메서드 내에서만 호출할 수 있다. 대기하고 있는 재사용 cell이 있는 경우 특정 타입의 기존 cell을 반환하고, 그렇지 않은 경우 calss나 기존에 제공된 스토리보드를 참고하여 새로운 cell을 생성하고 반환한다. 

- 스토리보드나 nib 파일로 cell이 생성되는 경우
	- `dequeueReusableCell(withIdentifier:for:)`는 cell 객체를 불러오고 `init(coder:)` 메서드를 사용하여 초기화한다. 
- 등록된 class를 통해 cell이 생성되는 경우
	- `dequeueReusableCell(withIdentifier:for:)`는 cell을 생성하고 `init(style:reuseIdentifier:)` 메서드를 통해 초기화 한다. 
- nib 기반의 cell을 생성하는 경우, `dequeueReusableCell(withIdentifier:for:)` 메서드는 제공된 nib 파일로부터 cell 객체를 불러온다. 
- 만약 reuse 가능한 cell이 있는 경우 `dequeueReusableCell(withIdentifier:for:)`  대신에 `prepareForReuse()` 메서드를 호출한다. 

### `register(_:forCellReuseIdentifier:)`

```swift
func register(_  cellClass: AnyClass?, forCellReuseIdentifier  identifier: String)
```

새로운 table cell을 생성하기 위해 `cellClass`, `UINib` 등을 등록하는 역할을 한다. 

어떠한 cell이던 dequeue하기 전에, 새로운 cell들을 어떻게 만드는지 table view에게 알려주기 위해`register` 메서드를 호출해야 한다. 만약 특정 타입의 cell이 재사용 큐에 없을 경우, table view는 제공받은 정보에 기반하여 새로운 cell 객체를 자동으로 만들어준다. 

이전에 동일한 reuse identifier로 등록한 class나 nib 파일이 있는 경우, `cellClass` 파라미터에 지정한 클래스가 이전 객체를 대체한다. `cellClass`에 `nil`을 지정하여 지정된 reuse identifier에서 클래스 등록을 취소할 수도 있다!

## 고민된 점 
- table view가 cell을 reuse하는 방식과 원리

## 해결 방법 
- 공식 문서 메서드 및 설명 참고하여 학습 내용에 정리!
---

**Ref**
[prepareForReuse](https://developer.apple.com/documentation/uikit/uitableviewcell/1623223-prepareforreuse)

[dequeueReusableCell(withIdentifier:for:)](https://developer.apple.com/documentation/uikit/uitableview/1614878-dequeuereusablecell)

[register](https://developer.apple.com/documentation/uikit/uitableview/1614888-register)
