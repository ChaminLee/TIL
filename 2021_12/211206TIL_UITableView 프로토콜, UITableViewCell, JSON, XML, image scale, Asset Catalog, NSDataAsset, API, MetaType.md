# UITableView 프로토콜, UITableViewCell, JSON, XML, image scale, Asset Catalog, NSDataAsset, API, MetaType

## 211206_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### !복습내용! 

- background 상태를 사용하는 앱은 실행 중 시스템이 언제 앱을 종료할지 알 수 있다. 
- background 상태를 사용하지 않는 앱은 시스템이 언제 앱이 종료되는지 알 수 없다

- RAM은 CPU의 구성요소가 아님 
	- L1 cache: 1차 캐시 메모리 (작동순서가 가장 먼저이며, 찾고자 하는 데이터가 L1에 없다면 L2, L3로 이동한다)
	- ALU(Arithmetic & Logic Unit): 연산장치
	- Control Unit: 제어 장치
	

- LLDB po를 통해 출력하는 문자열을 임의로 지정하려면 
	- [debugDescription](https://developer.apple.com/documentation/swift/customdebugstringconvertible/1540125-debugdescription) 사용
	- 바로 프로퍼티를 부르는 것은 지양하고, 인스턴스를`String(reflecting: )`을 통해 초기화하여 `String`으로 바꿔주는 방식을 사용한다. 

### `UITableViewDataSource`, `UITableViewDelegate`

> 1 . `UITableViewDataSource`

- table의 section과 row의 개수를 알려준다. 
- table의 각 row에 대해 cell을 제공한다. 
- section header와 footer에 title을 제공한다. 
- table의 index를 구성한다. 
- 사용자 혹은 table로부터 시작된 업데이트(데이터에 기반하여 변화가 필요한 경우)에 반응한다.

기본적으로 2개의 메서드를 필수적으로 요구한다. 

```swift 
// Return the number of rows for the table.     
override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
   return 0
}

// Provide a cell object for each row.
override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
   // Fetch a cell of the appropriate type.
   let cell = tableView.dequeueReusableCell(withIdentifier: "cellTypeIdentifier", for: indexPath)
   
   // Configure the cell’s contents.
   cell.textLabel!.text = "Cell text"
       
   return cell
}
```
> 1 . `UITableViewDelegate`

section을 관리하는, section의 header, footer를 구성하는, cell들을 재구성, 삭제하는 그리고 table view에서 다른 액션을 수행하는 메서드들을 관리한다. 

- 사용자 정의 header, footer 뷰를 생성하고 관리한다. 
- row와 header, footer의 높이를 지정한다. 
- 더 나은 스크롤링을 지원하기 위해 높이 추정치를 제공한다. 
- row 내용을 들여쓴다. 
- row 선택에 대해 응답한다. 
- table view에 대한 swipe나 다른 액션에 반응한다. 
- table의 콘텐츠를 편집하는데 지원해준다. 

### UITableViewCell 
UITableViewCell를 만들 때 무조건 custom하게 만드는 것이 아니라 apple이 기본적으로 제공하는 4가지 기본 스타일이 있다. 

> 1 . Basic 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fz196w%2Fbtrm3QXcuqM%2FQYYK3hjkg5d3tsvfSLC1J0%2Fimg.png)

**이미지(좌측) + 타이틀(좌측 정렬)**을 기본으로 제공해주는 타입이다. 

```swift
let cell: UITableViewCell = ...

var content = cell.defaultContentConfiguration()

content.image = UIImage()
content.text = ""

cell.contentConfiguration = content
```

위와 같은 방식처럼 기본 템플릿의 프로퍼티에 접근하여 값을 설정해줄 수 있다. 

> 기존에 기본 템플릿에서 `.textLabel`, `.detailTextLabel`을 사용하던 것이 iOS 14부터는 deprecated되고 위 방법을 사용한다고 한다. 

> 2 . Subtitle

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbw0tS0%2Fbtrm7NezUZx%2FdSPKXsOw7UHki5HOCqUE4k%2Fimg.png)

앞선 Basic 타입에 subtitle이 추가된 형식이다. 


> 3 . Value 1 (Right Detail)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FmrNti%2FbtrndGyuxrb%2FCWGjAe5c6huWE9uARbEw1k%2Fimg.png)

좌측 정렬된 타이틀과 우측 정렬된 subtitle이 같은 선상에 위치한다. 

> 4 . Value 2 (Left Detail)

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fr3hTD%2FbtrneuR66KF%2FnOt9NJwDgXS4MWYLj38zRK%2Fimg.png)

우측 정렬된 타이틀, 좌측 정렬된 subtitle이 같은 선상에 위치한다. 

### JSON

객체를 데이터로 저장하여 전달/보관하기 위해 사용한다!
( 인스턴스 > JSON >인스턴스 )

- `{ }` : 객체(딕셔너리)
- `[ ]` : 배열
- `" "` : 문자열
- `100` : 숫자 
- `true/false` : Bool
- `null` : 빈 값 


> ### XML 
구시대적인 방식으로, 요즘엔 잘 안쓰고 사람친화적인 JSON 쓴다.
XML은 컴퓨터에겐 좋으나, 사람이 읽기는 어렵다는 특징이 있다. 

### @1x @2x @3x ?

![](https://kangraemin.github.io/assets/images/posts/2021-01-21-iOS-image-set/pic1.png)

기기가 점점 더 발전함에 따라 디스플레이 또한 점점 개선되고 좋아지면서 1 point에 많은 픽셀이 들어가게 되었다고 한다. 즉 해상도가 더 높아지게 된 것이다.

- 1x (표준) : 1 point = 1 pixel
- 2x (retina) : 1 point = 4 pixel
- 3x (retina HD) : 1 point = 9 pixel

수치로 보면 3x의 이미지가 가장 해상도가 좋아보이긴 한다. 하지만 1x 즉, 표준 디스플레이를 가지는 기기에 3x 이미지를 넣으면 기존 3x 이미지 처럼 보이지 않는다. 

하드웨어 기기 자체가 가지는 특성이 1x이기 때문에 3x를 넣어도 1x처럼 보이게 되는 것이다. 이 때문에 사용자들이 어떠한 디스플레이를 갖는 기기를 사용할지 모르기에 이 3가지 버전의 이미지를 갖고 대응해야하는 것이다

### Asset Catalog

Xcode 좌측 네비게이터의 파일들을 보면 `Assets.xcassets` 폴더를 볼 수 있다. 

해당 폴더에는 image set 뿐만 아니라, color set. data set, symbol image set 또한 추가될 수 있다. 즉 앱 실행에 필요한 자원들을 보관하고 있는 곳이라고 볼 수 있다. 이를 `Asset Catalog`라고 부른다고 한다. 

그렇다면 어떻게 데이터들을 추가할까?

예시로 [Image Set](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_ref-Asset_Catalog_Format/ImageSetType.html#//apple_ref/doc/uid/TP40015170-CH25-SW1)을 추가하는 방법을 보자. 

Xcode > Assets > list의 UI에서 (+)를 눌러서 원하는 데이터셋을 추가할 수도 있고, 그게 아니라면 로컬에 있는 파일을 드래그 앤 드롭해서 추가할 수 도 있다. 

> `Contents.json`파일로 옵션들을 미리 지정하여 각 스케일(1x, 2x, 3x)에 맞게 이미지 셋을 만들어줄 수도 있다. 
> 
> image-folder
> ㄴㅡ image_1x
> ㄴㅡ image_2x
> ㄴㅡ image_3x
> ㄴㅡ Contents.json (메타 데이터 파일)
>  
>  `image-folder`를 Asset 폴더에 넣어준다! 
>  Contents.json에는 문서상 옵션들을 참고해서 원하는 데이터 타입을 키로 주고 내부 값을 채워넣는다. 


### NSDataAsset

`NSDataAsset`란 asset catalog내 저장된 data set 유형의 객체이다. 

```swift
class NSDataAsset : NSObject
```

`init?(name: NSDataAssetName)`을 통해 현재 asset catalog에 속해있는 data set을 찾아올 수 있다. 

```swift
var products: [Items] = []

guard let asset = NSDataAsset(name: "items") else {
    return
}
// .data 프로퍼티로 데이터에 접근 가능 
let data = asset.data

let decoder = JSONDecoder()
products = try decoder.decode([Items].self, from: data)
```

### API란?

API는 프로그램들이 서로 상호작용하는 것을 도와주는 매개체라고 한다. 

공공 데이터 API, OPEN API등등 많이 들어봤을텐데 위 설명만으로는 이해가 되지 않을 것이다. 

위키를 참고해보면, 어떠한 응용 프로그램에서 데이터를 주고 받기 위한 방법을 의미한다고 한다. 예시로 공공 데이터 API가 있다고 하면, 어떤 방식으로 데이터를 요청해야하는지, 어떤 데이터를 받을 수 있는지에 대한 규격들을 API라고 말한다고 한다. 

많이 보이는 예시로 요리사 <-> 점원 <-> 손님의 관계 속 점원의 역할처럼 "손님의 주문을 받아 요리사에게 전달하고, 요리사의 요리를 전달받아 손님에게 다시 전달해주는" 과정과 유사하다고 볼 수 있다. 이러한 매개자 역할을 API라고 볼 수 있을 것 같다. 

사람이 TV를 켜기 위해 리모컨을 사용하는 것도 유사한 사례이지 않을까 싶다..! 리모컨 속 많은 버튼들이 나열되어 있는 것도 데이터를 요청하는 하나의 규격이며, 어떤 반응을 얻을 수 있는지에 대한 정보라고도 볼 수 있을 것 같다!

그래서 일종의 소프트웨어 인터페이스라고 설명하는 것 같다. 

### MetaType

`Person`이라는 Class가 있다고 했을 때, `Person.self`를 통해 인스턴스가 아니라, 해당 타입을 반환하는 경우를 봤을 것이다. 이 때 `Person.Type`을 마주할 수 있는데, 이를 `Person`의 메타 타입이라고 한다.

메타 타입은 class, struct, enum, protocol 타입을 포함한 모든 타입의 타입을 나타낸다. calss, struct, enum은 `.Type`으로 표현되고, protocol은 `.Protocol`로 표현된다. 

타입의 타입이라는 말이 무엇일까...

즉 `Person`이라는 타입의 타입을 말한다고도 볼 수 있다. 그래서 `Person` 타입의 타입을 `Person.Type`이라고 말할 수 있는 것 같다. 

`type(of: )`를 통해 인스턴스의 타입을 확인할 수 있는데, 이 또한 반환값이 메타타입으로서 이를 통해 타입 프로퍼티/메서드에 접근할 수 있다. 

```swift
struct Person {
    let name: String
    static let power = 100
}

let chacha = Person(name: "chacha")
chacha.power // Error

let cha = type(of: chacha) // Person.Type
cha.power // 100
```

정리하자면 `Person`은 Person이라는 자체적인 타입인 것이고, `Person.self`가 메타타입의 값(`Person.Type`)이 되는 것이다. 

```swift
struct Person {
    let name: String
    static let power = 100
    static func printType() {
        print(Self.self)
    }
}

Person.printType() // Person.Type
```

위와 같은 방식으로도 메타 타입을 불러올 수 있다. 특정 타입 내부에서 `Self`를 쓰면 속해있는 타입 자체를 말하기 때문에, 여기에 `.self`를 붙인 것과 같게 되는 것이다!


## 고민된 점 
- JSON 형식의 데이터는 어떻게 parsing하는가?
- Asset Catalog를 추가하고 다루는 방법?

## 해결 방법 

- 공식문서 참고 및 예제 구성을 해보고... 학습 내용에 정리!

---

**Ref**
[UITableViewDataSource](https://developer.apple.com/documentation/uikit/uitableviewdatasource)
[UITableViewDelegate](https://developer.apple.com/documentation/uikit/uitableviewdelegate)
[Tables](https://developer.apple.com/design/human-interface-guidelines/ios/views/tables/)
[UITableViewCell.CellStyle](https://developer.apple.com/documentation/uikit/uitableviewcell/cellstyle)
[iOS - image set에서 1x, 2x, 3x 이미지를 사용하는 이유](https://kangraemin.github.io/ios/2021/01/21/iOS-image-set/
)
[Create asset catalogs and sets](https://help.apple.com/xcode/mac/current/#/dev10510b1f7)
[Asset Catalog Format Reference](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_ref-Asset_Catalog_Format/ImageSetType.html#//apple_ref/doc/uid/TP40015170-CH25-SW1)
[NSDataAsset document](https://developer.apple.com/documentation/uikit/nsdataasset)
[NSData​Asset 활용](https://nshipster.co.kr/nsdataasset/)
[WIKI - API](https://ko.wikipedia.org/wiki/API)
[API란?](http://blog.wishket.com/api%EB%9E%80-%EC%89%BD%EA%B2%8C-%EC%84%A4%EB%AA%85-%EA%B7%B8%EB%A6%B0%ED%81%B4%EB%9D%BC%EC%9D%B4%EC%96%B8%ED%8A%B8/)



