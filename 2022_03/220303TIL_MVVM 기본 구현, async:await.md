

# MVVM 기본 구현, async/await

## 220303_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. MVVM 기본 구현

우선 MVVM 구조를 다시 잡아봤다.

![](https://user-images.githubusercontent.com/45652743/156524776-a0172ad3-47e6-4ed5-a97e-e69d7d31a181.png)

찾다보니 clean architecture라는 것도 있는 것 같은데, 맛만 본 것 같다...! 같이 더 공부해보고 적용해보면 좋을 것 같다. 

MVVM 구조를 구현함에 있어 View  ↔  ViewModel 사이에서 데이터 변화를 관측하고, 스스로 UI를 변경할 수 있도록 하기 위한 방법들에 대해 고민해봤다.

-   Observable 구현
    -   Rx와 가장 닮아있는 방식이라고 생각했으며 동시에 Rx를 구현하는...? 방법이지 않을까 생각했다
    -   이에 커스텀하게 Observable을 만들 경우, 추후에 operator를 직접 만들어야 하는 경우가 발생할 것 같아서 현재로서는 선택하지 않게 되었다.
-   Protocol
    -   Protocol을 통해 구현하는 방식은 delegate pattern 처럼 동작하고 있다고 생각했다.
    -   delegate를 할당해주어야한다.
-   Closure
    -   ViewModel내에 closure를 정의하고 호출이 필요한 곳에 위치시켜둔 후, ViewController에서 해당 클로저를 구현하여 작동하는 방식이다.
    -   여러 자료들을 봤을 때, Rx없이 구현하기에 가장 보편적인 방법이라고 생각했다.

클로저 방식과 프로토콜 방식은 은근 간단하다! 

예시 코드는 모두 여기서 가져왔다
https://demirciy.medium.com/mvvm-in-ios-development-with-protocol-closure-reactive-programming-rxswift-d0933b235235

#### MVVM + Protocol

```swift
import Foundation

protocol ListDelegate {
    func coinsDidRefresh()
    func coinsCouldNotRefresh()
}

class ListViewModel {

    // MARK: Properties
    var coins: [Coin] = []

    var delegate: ListDelegate?

    func refreshCoins() {
        coins = getDummyCoins()
        
        delegate?.coinsDidRefresh()
    }

    func search(_ text: String?) {
        if let text = text, !text.isEmpty {
            coins = getDummyCoins().filter { $0.symbol.lowercased().contains(text.lowercased()) }
        } else {
            coins = getDummyCoins()
        }

        delegate?.coinsDidRefresh()
    }
}

private extension ListViewModel {

    func getDummyCoins() -> [Coin] {
        [
            .init(symbol: "BTCUSDT", price: 42500),
            .init(symbol: "ETHUSDT", price: 1500),
            .init(symbol: "XRPUSDT", price: 0.5),
            .init(symbol: "LTCUSDT", price: 175)
        ]
    }
}
```

View가 ViewModel이 변경되었을 때 스스로 UI를 업데이트하기 위해서 중요한 것은 프로토콜에 구현되어 있는 메서드이다.

```swift
protocol ListDelegate {
    func coinsDidRefresh()
    func coinsCouldNotRefresh()
}
```

ViewModel의 `refreshCoins()`, `search()` 메서드 내부를 보면 delegate의 메서드들이 호출되고 있음을 볼 수 있다. 

예를 들어서, `refreshCoins()`처럼 데이터를 업데이트하게 되는 경우 뷰 또한 업데이트가 되어야 한다. 이에 delegate 메서드를 호출해준 것이다. 지금은 정의만 해두고 ViewController에서 필요로하는 UI관련 작업을 해당 메서드를 통해 구현하면 되는 것이다. 

그렇게 되면 viewModel의 메서드가 호출되면 자연스레 delegate를 통해 UI작업도 수행되는 것이다. 

```swift
import UIKit

class ListController: UITableViewController {

    // MARK: Properties
    private let searchController: UISearchController = {
        let controller: UISearchController = UISearchController()
        controller.searchBar.placeholder = "Search"
        controller.obscuresBackgroundDuringPresentation = false
        return controller
    }()

    private let viewModel: ListViewModel

    init(viewModel: ListViewModel = ListViewModel()) {
        self.viewModel = viewModel

        super.init(style: .grouped)

        self.viewModel.delegate = self
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        navigationItem.searchController = searchController

        searchController.searchResultsUpdater = self
    }

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)

        viewModel.refreshCoins()
    }
}

// MARK: - TableView
extension ListController {

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return viewModel.coins.count
    }

    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let coin = viewModel.coins[indexPath.row]

        let cell = UITableViewCell(style: .value1, reuseIdentifier: "\(indexPath.section)_\(indexPath.row)")

        cell.textLabel?.text = coin.symbol
        cell.detailTextLabel?.text = "\(coin.price)"

        return cell
    }

    override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)

        let coin = viewModel.coins[indexPath.row]

        let coinViewModel = CoinViewModel(coin: coin)
        let coinController = CoinController(viewModel: coinViewModel)

        navigationController?.show(coinController, sender: self)
    }
}

// MARK: - UISearchResultsUpdating
extension ListController: UISearchResultsUpdating {

    func updateSearchResults(for searchController: UISearchController) {
        viewModel.search(searchController.searchBar.text)
    }
}

// MARK: - ListDelegate
extension ListController: ListDelegate {

    func coinsDidRefresh() {
        tableView.reloadData()
    }

    func coinsCouldNotRefresh() {
        // An error occured
    }
}
```

그리고 ViewController를 보면 coinsDidRefresh에 테이블 뷰를 업데이트하도록 구현한 것을 볼 수 있다. 

즉 데이터가 업데이트 되면, 테이블뷰 또한 리로드 되도록 구현한 것이다. 

클로저 방식도 크게 다르지 않다. 

####  MVVM + Closure

위 프로토콜 방식을 기반으로 다른 부분의 코드만 보자. 
앞의 프로토콜, delegate가 하던 일을 클로저로만 바꿔주면 된다. 

먼저 viewModel을 보자 

```swift
class  ListViewModel {
	var coinsDidRefresh: (() ->  Void)?
	...
	func refreshCoins() {
		coins = getDummyCoins()
		coinsDidRefresh?()
	}
}
```

위와 같이 escaping 클로저로 선언해주고 필요한 시점에 viewModel의 메서드에서 호출시켜준다. 

마찬가지로 해당 클로저는 ViewController에서 원하는 UI 작업을 담아 구현해준다. 프로토콜과 달리 이번에는 viewDidLoad에서 클로저를 초기화시켜주는 느낌으로 세팅해주어야 한다. 

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    viewModel.coinsDidRefresh = { [weak self] in
        self?.tableView.reloadData()
    }
}
```

이렇게 구현하면 마찬가지로 viewModel의 데이터 변경시 테이블뷰 또한 자동으로 업데이트 된다. 

초기에 설정해두어야 하는 부분이기에 viewDidLoad에 구현하는 것 같다. 이 덕분에 이후 액션에  문제 없이 view가 변화될 수 있는 것 같다. 

Rx를 쓰는 방법은 아직 학습중이다... 
우선 MVVM의 기본적인 구조나 동작 원리 등에 익숙해지는 단계인 것 같다. 

이렇게 기본 라이브러리로 구현해보고 Rx를 적용해보면  더 빠르게 적응할 수 있지 않을까 싶다.

### 2. Concurrency

async await 알아보기 

#### 1. 비동기 프로그래밍?

- 동기: 프로그램의 흐름과 이벤트의 발생 및 처리를 종속적으로 수행하는 방법 
- 비동기: 프로그램의 흐름과 이벤트의 발생 및 처리를 독립
적으로 수행하는 방법  
동시성 프로그래밍: 여러 작업이 논리적인 관점에서 동시에 수행되는 것 
병렬성 프로그래밍: 여러 작업이 물리적인 관점에서 동시에 수행되는 것

기존 에 비동기적으로 작동하는 메서드들에는 여러 문제점 이 있었다. 
- 과도한 중첩 
- 오류 처리의 어려움 
	- 에러를 모든 블록에서 처리해줘야 함 
- 실수하기 쉽다
	- return, completion 안하는 등 

이 문제를 해결하기 위해 애플에서 내놓은 것이 async/await이다.

async/await을 통해 비동기 코드를 동기 코드처럼 읽히도록 작성할 수 있다. 

```swift
// 일반 비동기 메서드
listPhotos(inGallery: "Summer Vacation") { photoNames in
    let sortedNames = photoNames.sorted()
    let name = sortedNames[0]
    downloadPhoto(named: name) { photo in
        show(photo)
    }
}
```

보통은 이렇게 후행 클로저 방식으로 completion을 구현해주게 된다. 

이제 async/await을 보자

```swift
let photoNames = await listPhotos(inGallery: "Summer Vacation")
let sortedNames = photoNames.sorted()
let name = sortedNames[0]
let photo = await downloadPhoto(named: name)
show(photo)
```

뭔가 진짜 동기적으로 작동하는 것 처럼 읽힌다.

대략 이해해보자면 async가 붙은 메서드를 사용할 때는, await을 써줘야 하는 것이다. 

async는 함수를 비동기적으로 작동시킬 수 있는데, await을 만나면 쓰레드 block을 해제하고 suspend된다. 

즉 `listPhotos()`메서드가 비동기적으로 작동하여 반환값을 보낼 때 까지 다음 라인이 실행되지 않고 멈춰있는 것이다. 

await은 글로벌 쓰레드에서 실행되며, 이외의 함수 내 기본적인 코드들은 속해있는 함수가 호출된 쓰레드와 동일한 쓰레드를 갖는다. 

iOS 15.0 이상부터 적용 가능한 기술이라 간략하게만 알아봤다..! 나중에 사용하게 되면 더 자세하게 알아보고 쓰면 좋을 것 같다. 

> async 프로퍼티는 get only 만 가능! 

> 코루틴이란? 
> - 두 개 이상의 루틴이 서로룰를 호출하는 관계 
> - A를 프로그래밍할 때 B를 A의 서브루틴으로 생각(반대도)
> - 실행되는 코루틴은 이전에 자신의 실행이 마지막으로 중단되었던 지점 다음의 장소에서 실행을 재개 
> 

## 고민된 점 
- MVVM을 어떻게 구현해야하나...
- 비동기 프로그래밍을 편리하게 구현하는 방법

## 해결 방법 
- MVVM을 구현하는 여러 방법 서칭
- async/await 찍먹해보기

---

**Ref**

https://demirciy.medium.com/mvvm-in-ios-development-with-protocol-closure-reactive-programming-rxswift-d0933b235235

https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html
