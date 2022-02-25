
# UISearchController, 단축키 꿀팁

## 220225_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. UISearchController

검색을 위해 사용된다. 기본적으로 `UISearchBar` 프로퍼티를 가지고 있으며, 이를 통해 검색어를 입력할 수 있다. 

일반적인 사용법은 다음과 같다. 

#### 1. `UISearchController` 인스턴스를 생성한다
- 이 때 검색 결과를 보여줄 `ViewController`가 필요하다. 
- 이를 `searchResultsController`라고 하는데, 인스턴스 생성시 init을 통해 지정해줄 수 있다. 

```swift
init(searchResultsController: UIViewController?)
```

이를 가지고 일반적으로 아래와 같이 생성해줄 수 있다. 

```swift
let SearchController = UISearchController(searchResultsController: ResultTableViewController())
```

#### 2.  navigationItem에 searchController에 주입

`navigationItem`은 기본적으로 `searchController` 프로퍼티를 가지고 있다. 이에 원하는 `UISearchController` 인스턴스를 생성해서 넣어주면 된다. 

```swift
self.navigationItem.searchController = memoSearchController
```

매우 간단하다. 

관련해서 스크롤 할 때 숨길 것인지 말지를 묻는 프로퍼티도 있다. 이외에도 여러 속성을 지정할 수 있는 프로퍼티들이 있기에 들여다봐도 좋을 것 같다. 

```swift
self.navigationItem.hidesSearchBarWhenScrolling = **false**
```

#### 3. 검색에 따라 보여줄 결과 지정

유저가 search bar를 통해 입력한 정보를 기반으로 검색 결과를 업데이트 해줘야 한다. 이 때 `UISearchResultsUpdating` 프로토콜을 채택하여 업데이트 해줄 수 있다. 

```swift
func updateSearchResults(for searchController: UISearchController) {
	// 검색바에 입력시 매번 호출된다
}
```
이 메서드에서 검색어를 받아 전처리해주고 검색 결과를 반영해줄 메서드를 호출해주면 된다. 

공식문서의 예제를 보면 검색어를 받을 때, 양쪽의 공백을 지우고, 띄어쓰기를 기준으로 검색어를 나누어 필터링하고 있음을 볼 수 있다. 

```swift
guard let searchKeyword = searchController.searchBar.text else {
    return
}

let strippedKeyword = searchKeyword.trimmingCharacters(in: CharacterSet.whitespaces)
let searchKeywords = strippedKeyword.components(separatedBy: .whitespaces)
```

이외에도 요구되는 처리 방법에 따라 문자열을 처리해주면 될 것 같다. 

아무튼 이렇게 구현까지 해주고 delegate도 채택해주면 된다. `UISearchController`의 `searchResultsUpdater`에 지정해주면 된다. 

```swift
UISearchController().searchResultsUpdater = self
```

이외에도 `UISearchControllerDelegate` 프로토콜이 제공해주는 메서드들을 사용해 볼 수 있다. searchbar의 life cycle에 따라 호출될 메서드를 사용해 볼 수도 있다.

추가적으로 커스텀한 searchBar를 구현하고 싶은 경우, 이 또한 가능하다. 직접 프로퍼티에 접근해서 높이를 바꾸거나, corner radius를 바꾸거나 배경 색을 바꾸거나, 아이콘을 바꾸는 등 입맛에 맞게 쉽게 사용할 수 있다. 

### 2. 단축키 꿀팁

예거가 여러 단축키들을 알려주셨다.
감사합니다 예거!👍🏻

#### 시뮬레이터 폰트 크기 조절

다이나믹 타입을 테스트할 때 사용하기 좋을 것 같다.
- `option` + `command` + `- or +`

#### 시뮬레이터 스크린샷

가끔 캡쳐된 것을 보면 시뮬레이터 상단의 바(?)도 포함되게 찍힌 사진을 볼 수 있다. 기본 캡처의 응용방식이라 기억해두면 좋을 듯 하다

- `command` + `shift` + `4` 이후 `space` 
	- 그 다음 원하는 화면 스크린샷 찍기
	- 사방에 음영이 있는 스크린샷이 찍힌다.
- `command` + `shift` + `4` 이후 `space`  + `option` 하면 시뮬레이터 음영없이 찍힌다

잘 써먹을 수 있을 것 같다!

### 3. Readme 손보기

전체적으로 지금까지 진행했던 프로젝트들의 readme를 조금씩 손봤다. [야곰 아카데미 학습 기록](https://github.com/ChaminLee/iOS_Yagom_Academy)

나중에 언젠가 해야할 거라면... 미리 해두는게 정신건강에도 좋을 것 같아서 미리 해뒀다! 

포폴로 쓰기에는 애매한 것 같다는 생각이 들지만... 
개인 프로젝트에 지금까지 배운 내용들을 녹여내는 것이 중요할 것 같다. 



## 고민된 점 
- UISearchController 구현 시, 검색 결과를 처리하는 방법


## 해결 방법 
- 공식문서 예제 참조 

---

**Ref**

https://developer.apple.com/documentation/uikit/uisearchcontroller
