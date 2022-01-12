# Modern CollectionView - List

## 220111_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용


# Modern Collection View 구현

iOS 14 이후부터 지원하는 기능으로 이전의 방법과 아예 다른 방식으로 collection view를 만들어주고 있다. 개괄적인 순서를 먼저 보고 하나 하나 예제 코드를 통해 살펴보자! 

이번에는 List 형태의 collection view를 만들어 볼 것이다.

![](https://i.imgur.com/hg3XEw7.png)

위와 같은 뷰를 보면 "테이블뷰로 만들었네!"라고도 할 수 있지만 collection view로도 동일한 뷰를 만들 수 있다. 

이제 modern하게 collection view를 list 형식을 차근차근 살펴보자.

# 순서

> 1. `CollectionView` 만들기 (List)
> 2. `CollectionViewListCell` 생성
    2-1. `UIConfigurationStateCustomKey` 생성    
    2-2. `UIConfigurationState`를 extension하여 state의 데이터 프로퍼티 생성
    2-3. `ConfigurationState`를 재정의하여 현재 상태의 cell이 가지고 있는 새로운 값을 위 프로퍼티에 넣어준다. 그 이후 리턴!
    2-4. cell에서 사용할 `UIListContentView`를 생성한다.
    2-5. cell내의 Layout을 잡아준다.
    2-6. `updateConfiguration(using state: UICellConfigurationState)` 재정의
> 3. `collectionView`의 layout 생성 (`UICollectionViewCompositionalLayout`)
> 4. `collectionView`에 레이아웃 적용하여 뷰에 추가
> 5. dataSource에 `CellRegistration`, `UICollectionViewDiffableDataSource` 할당
> 6. `snapshot`을 dataSource에 apply
>   6-1. 모델에 Hashable 프로토콜 채택 및 identifier 프로퍼티 생성


# 구현

## 1. `CollectionView` 만들기 (List)

우선은 ViewController에 collectionView를 만들어 놓고 대기시켜둔다. 

```swift=
var collectionView: UICollectionView!
```

## 2. `CollectionViewListCell` 생성

이제는 custom한 `UICollectionViewListCell`을 만들어 줄  것이다. 

순서를 따라 천천히 살펴보자. 

### 2-1. `UIConfigurationStateCustomKey` 생성

우선 `contifurationState`에 추가할 뷰의 커스텀 state를 정의한다.
이는 나중에 `UIConfigurationState`에서 커스텀한 state를 만들기 위해 사용된다!

```swift=
private extension UIConfigurationStateCustomKey {
    static let animal = UIConfigurationStateCustomKey("animal")
}
```

### 2-2. `UIConfigurationState`를 extension하여 state의 데이터 프로퍼티 생성

`UIConfigurationState`를 extension하여 위에서 만든 커스텀 state를 타입 프로퍼티로 추가해준다.

`UIConfigurationState` 프로토콜은 외관에 영향을 미치는 모든 일반적인 state와 함께 trait collection을 포함하는 configuration state 객체의 청사진을 제공한다. 

- `UIConfigurationState` : 뷰의 상태(state)를 캡슐화하여 가지고 있는 객체
- `get` : 위에서 만든 custom key를 가지고 state를 가져온다.
- `set` : 위의 키에 해당하는 value에 새로운 newValue를 할당한다.

```swift=
extension UIConfigurationState {
    var animalData: Animal? {
        get { return self[.animal] as? Animal }
        set { self[.animal] = newValue }
    }
}

```


### 2-3. Cell내에서 `ConfigurationState`를 재정의

현재 상태의 cell이 가지고 있는 새로운 값을 위에서 만든 `animalData` 프로퍼티에 넣어준다. 그 이후 반환해준다!

- `UICellConfigurationState` : cell의 상태(state)를 캡슐화한 객체
    - cell의 상태라고 하면 selected, focused, disabled의 상태인지 등을 나타낸다.
    
일반적으로는 `UICellConfigurationState` 객체를 직접 생성해주지는 않는다. configuration state를 갖기 위해서는 cell의 하위 클래스에서 `updateConfiguration(using:)` 메서드를 재정의하여 state 파라미터를 써주면 된다. 

이 메서드 밖에서는 configurationState 프로퍼티를 사용하여 cell의 configuration state를 얻을 수 있다. 앞서 말한 것 처럼 `UIConfigurationStateCustomKey`를 사용하여 커스텀 state key를 정의하면 커스텀 state를 cell configuration state에 추가해줄 수 있다.

```swift=
class AnimalListCell: UICollectionViewListCell {
    private var animalData: Animal?   
    
    // ...
    
    override var configurationState: UICellConfigurationState {
        var state = super.configurationState
        state.animalData = self.animalData
        return state
    }
}
```


### 2-4. cell에서 사용할 `UIListContentView`를 생성한다.

이 때 스타일은 `UIListContentConfiguration`를 통해 정해준다.

여기서는 `defaultAnimalConfiguration()` 메서드를 통해 정해주자!

```swift=
class AnimalListCell: UICollectionViewListCell {
    private var animalData: Animal?
    // list 기반의 contentView를 반환하는 메서드
    private func defaultAnimalConfiguration() -> UIListContentConfiguration {
        return .subtitleCell()
    }
    
    // list 기반의 contentView를 만든다
    private lazy var animalListContentView = UIListContentView(configuration: defaultAnimalConfiguration())    
}
```

### 2-5. cell내의 layout을 잡아준다. 

레이아웃은 당연히 자유니까 입맛에 맞게 해주면 된다.  

하지만 여기서 정의한 `setupViewsIfNeeded()`은 바로 다음에 볼 `updateConfiguration()` 에서 호출하게 되는데, 업데이트 되면서 레이아웃이 여러 번 중복적으로 잡히지 않게 하기 위해 특정 레이아웃 집합을 따로 빼두어 nil인지 확인해준다!

```swift=
extension AnimalListCell {
    func setupViewsIfNeeded() {
        // 커스텀 view에 대한 제약이 기존에 주어져있다면 다시 layout를 적용하지 않도록 함
        guard animalTypeConstraints == nil else {
            return
        }
        
        [animalListContentView, animalTypeLabel].forEach {
            contentView.addSubview($0)
            $0.translatesAutoresizingMaskIntoConstraints = false
        }
        
        let constraints = (leading:
                            animalTypeLabel.leadingAnchor.constraint(greaterThanOrEqualTo: animalListContentView.trailingAnchor),
                           trailing:
                            animalTypeLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor) )
        
        NSLayoutConstraint.activate([
            animalListContentView.topAnchor.constraint(equalTo: contentView.topAnchor),
            animalListContentView.bottomAnchor.constraint(equalTo: contentView.bottomAnchor),
            animalListContentView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor),
            animalTypeLabel.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            constraints.leading,
            constraints.trailing
        ])
        
        animalTypeConstraints = constraints
    }
}
```
### 2-6. `updateConfiguration(using state: UICellConfigurationState)` 재정의

현재 state를 사용하여 cell의 configuration을 업데이트해주는 역할을 한다.

하지만 이 메서드를 직접적으로 호출하는 것은 피해야하며 대신에 `setNeedsUpdateConfiguration()`를 호출하여 업데이트를 위한 요청을 보낼 수 있다.

- `setNeedsUpdateConfiguration()` : 현재 state로 configuration을 업데이트 할 것을 cell에게 알린다. 
     - cell의 configuration이 현재 state에 따라 업데이트 되어야 할 때 호출해줄 수 있다. 
     - 시스템은 `configurationState`가 바뀔 때 마다 이 메서드를 자동으로 호출해주며, 업데이트를 요청하는 다른 상황에서도 동일하다. 시스템은 여러 요청들을 통합하여 한 번의 업데이트를 한다. 
     - 만약 cell의 configuration state에 커스텀 state를 추가했다면 커스텀 state가 바뀔 때 마다 매번 이 메서드가 불릴 수 있도록 해줘야 한다.     
    
이 메서드는 아래와 같이 하위 클래스(셀)에서 재정의하여 제공되는 state를 사용하여 cell의 configuration을 업데이트할 수 있다.

```swift=
override func updateConfiguration(using state: UICellConfigurationState) {
    setupViewsIfNeeded()

    var content = defaultAnimalConfiguration().updated(for: state)

    content.image = urlToImage(state.animalData?.imageLink ?? "")
    content.imageProperties.maximumSize = CGSize(width: 50, height: 50)
    content.text = state.animalData?.name
    content.textProperties.font = .preferredFont(forTextStyle: .headline)
    content.secondaryText = "평균 수명: \(state.animalData?.lifespan ?? "")년"

    animalListContentView.configuration = content

    animalTypeLabel.text = state.animalData?.animalType ?? "1"
}

func urlToImage(_ urlString: String) -> UIImage? {
    guard let url = URL(string: urlString),
          let data = try? Data(contentsOf: url),
          let image = UIImage(data: data) else {
              return nil
          }

    return image
}
```

앞서 말한 것 처럼 cell의 configuration state에 커스텀 state를 추가했다면 커스텀 state가 바뀔 때 마다 매번 이 메서드가 불릴 수 있도록 해줘야 한다.

이에 cell 내에 해당 부분을 구현해준다!

```swift=
func update(with newAnimalData: Animal) {
    guard animalData != newAnimalData else {
        return
    }
    animalData = newAnimalData
    setNeedsUpdateConfiguration()
}
```

위 메서드는 이후 collectionView에서 `CellRegistration`의 handler에서 호출된다. 즉 이 때마다 cell을 업데이트 시켜주는 것이다. 

이제 cell 전체 코드를 보고 ViewController로 넘어가보자!
(접기)
```swift=
import UIKit

private extension UIConfigurationStateCustomKey {
    static let animal = UIConfigurationStateCustomKey("animal")
}

extension UIConfigurationState {
    var animalData: Animal? {
        get { return self[.animal] as? Animal }
        set { self[.animal] = newValue }
    }
}

class AnimalListCell: UICollectionViewListCell {
    private var animalData: Animal?
    private func defaultAnimalConfiguration() -> UIListContentConfiguration {
        return .subtitleCell()
    }
    
    private let animalTypeLabel = UILabel()
    private var animalTypeConstraints: (leading: NSLayoutConstraint, trailing: NSLayoutConstraint)?
    
    private lazy var animalListContentView = UIListContentView(configuration: defaultAnimalConfiguration())
    
    func update(with newAnimalData: Animal) {
        guard animalData != newAnimalData else {
            return
        }
        animalData = newAnimalData
        setNeedsUpdateConfiguration()
    }
    
    override var configurationState: UICellConfigurationState {
        var state = super.configurationState
        state.animalData = self.animalData
        return state
    }
}

extension AnimalListCell {
    func setupViewsIfNeeded() {
        guard animalTypeConstraints == nil else {
            return
        }
        
        [animalListContentView, animalTypeLabel].forEach {
            contentView.addSubview($0)
            $0.translatesAutoresizingMaskIntoConstraints = false
        }
        
        let constraints = (leading:
                            animalTypeLabel.leadingAnchor.constraint(greaterThanOrEqualTo: animalListContentView.trailingAnchor),
                           trailing:
                            animalTypeLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor) )
        
        NSLayoutConstraint.activate([
            animalListContentView.topAnchor.constraint(equalTo: contentView.topAnchor),
            animalListContentView.bottomAnchor.constraint(equalTo: contentView.bottomAnchor),
            animalListContentView.leadingAnchor.constraint(equalTo: contentView.leadingAnchor),
            animalTypeLabel.centerYAnchor.constraint(equalTo: contentView.centerYAnchor),
            constraints.leading,
            constraints.trailing
        ])
        
        animalTypeConstraints = constraints
    }
    
    override func updateConfiguration(using state: UICellConfigurationState) {
        setupViewsIfNeeded()
        
        var content = defaultAnimalConfiguration().updated(for: state)
        
        content.image = urlToImage(state.animalData?.imageLink ?? "")
        content.imageProperties.maximumSize = CGSize(width: 50, height: 50)
        content.text = state.animalData?.name
        content.textProperties.font = .preferredFont(forTextStyle: .headline)
        content.secondaryText = "평균 수명: \(state.animalData?.lifespan ?? "")년"
        
        animalListContentView.configuration = content
        
        animalTypeLabel.text = state.animalData?.animalType ?? "1"
        
    }
}

extension AnimalListCell {
    func urlToImage(_ urlString: String) -> UIImage? {
        guard let url = URL(string: urlString),
              let data = try? Data(contentsOf: url),
              let image = UIImage(data: data) else {
                  return nil
              }
        
        return image
    }
}

```

## 3. collectionView의 layout 생성 (UICollectionViewCompositionalLayout)

- `UICollectionViewCompositionalLayout`
    - 적응력이 높고 유연한 시각적 배열의 items를 결합할 수 있는 layout 객체다. 
    - 이는 collectionview layout의 한 종류이다. 이는 유연하고 빠르며 각 작은 구성 요소를 전체 레이아웃으로 결합하여 컨텐츠에 대한 시각적인 배열을 구축할 수 있다.

compositional layout은 뚜렷한 시각적 그룹으로 나뉘는 하나 이상의 섹션을 가지고 있다. 각 섹션은 표시하려는 가장 작은 데이터 단위인 개별 items가 담긴 group으로 구성된다. group은 수평/수직 또는 커스텀하게 정할 수 있다. 아이템에서 그룹, 그룹에서 섹션으로 구성하다보면 전체 레이아웃을 구성할 수 있다.
    
- `UICollectionLayoutListConfiguration`
    - list layout을 만들기 위한 configuration이다. 
    - 이 configuration을 사용하여 compositionalLayout에 대한 list section을 생성할 수 있다. 
    - 또한 layout은 오직 list secion만을 갖고 있다
    
아래의 방법으로 list section을 가지는 compositional layout을 만들 수 있다.
    
만약 긱 섹션에 대해 서로 다른 list configuration을 구현하고 싶다면 compositional layout의 section provider를 사용하여 각 섹션 별로의 list configuration을 만들면 된다!

```swift=
func createListLayout() -> UICollectionViewCompositionalLayout {    
    let config = UICollectionLayoutListConfiguration(appearance: .plain)
    return UICollectionViewCompositionalLayout.list(using: config)
}
```

## 4. collectionView에 레이아웃 적용하여 뷰에 추가

앞서 만든 collectionViewLayout을 추가하고, collectionView 인스턴스를 생성해주는 역할을 한다. 

```swift=
func configureCollectionView() {
    collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: createListLayout())
    view.addSubview(collectionView)
}
```

## 5. dataSource에 CellRegistration, UICollectionViewDiffableDataSource 할당

우선 dataSource가 있어야 한다. 

```swift=
var dataSource: UICollectionViewDiffableDataSource<Section, Animal>!
```

위 처럼 diffableDataSource를 만들어 줄 수 있다. 

`UICollectionViewDiffableDataSource`는 데이터를 관리하고 cell들을 collectionView에 제공해주는 객체이다.

diffableDataSource 객체는 collectionView 객체와 함께 작업하는 dataSource의 특정한 타입일 뿐이다. 주로 collection View의 데이터나 UI를 간편하고 효율적으로 업데이트 해주는 것을 관리하는 행위를 한다. 이 또한 마찬가지로 `UICollectionViewDataSource` 프로토콜을 채택하고 있기에 프로토콜이 가지는 메서드는 모두 제공할 수 있다.

collectionView에 데이터를 채우려면 다음의 순서를 따르면 된다.

> 1. diffableDataSource를 collectionView에 연결한다.
> 2. collection View의 cell을 구성하기 위해 cell provider를 구현한다.
> 3. 데이터의 현재 state를 생성한다.
> 4. 데이터와 UI를 나타낸다.


diffable data source를 collectionView에 연결하기 위해서는 `init(collectionView:cellProvider:)`을 통해 diffableDataSource를 만들고 collectionView에 넘겨주면 된다. 이 때 cell provider도 넘겨주게 되는데, 데이터와 UI를 어떻게 cell에 보여줄 지를 구성하는 역할을 해준다.

그 다음 데이터의 현재 state를 생성하고, data를 UI에 나타내고, snapshot에 applying 해준다.
    
> !!중요!! dataSource는 중간에 바꿔 끼우면 안된다!!
> diffableDataSource를 구성한 이후로는 collectionView의 dataSource를 바꾸면 안된다. 만약 collection View가 첫 구성 이후에 새로운 data source를 필요로 한다면, 새로운 collectionView와 diffableDataSource를 구성하고 만들어야 한다.
    
    
자 이제 만들어둔 dataSource를 사용하여 `CellRegistration`을 해보자

- `CellRegistration` : 
    - collection view에 cell을 등록해준다. 
    - collection View에 cell을 등록하고 각 cell을 보여주기 위해 구성한다. 
    - cell registration은 cell 타입과 data 타입을 generics 파라미터에 넣어준다. 
    - cell과 data를 registration handler로 넘겨서 cell을 구성해주는 것이다. handler에서는 content를 어떻게 구성할 지, cell의 외부모습은 어떻게 만들지 정해준다.
        
```swift=
let cellRegistration = UICollectionView.CellRegistration<AnimalListCell, Animal> { (cell, indexPath, animal) in
    cell.update(with: animal)
    cell.accessories = [.disclosureIndicator()]
}
```
cell registration 이후에는 `dequeueConfiguredReusableCell(using:for:item:)`에 넘겨서 data source의 cell provider를 호출해준다. 이에 `register()`를 해줄 필요가 없는 것이다. collectionView는 cell registration을 `dequeueConfiguredReusableCell(using:for:item:)`에 넘겨주면 자동으로 cell을 등록해주게 된다.

```swift=
dataSource = UICollectionViewDiffableDataSource<Section, Animal>(collectionView: collectionView) { (collectionView, indexPath, itemIdentifier) -> UICollectionViewCell? in
    return collectionView.dequeueConfiguredReusableCell(using: cellRegistration, for: indexPath, item: itemIdentifier)
}
```
- dequeueConfiguredReusableCell : 재사용 가능한 cell 객체를 dequeue한다.


> !! 중요 !!
> cell registration을 `UICollectionViewDiffableDataSource.CellProvider`의 클로저 내에서 만들면 안된다! 내부에서 만들 경우 cell이 reuse가 되지 않고, iOS 15 이상에는 예외가 발생할 수 있다고 한다.
        
`UICollectionViewDiffableDataSource`도 section과 사용할 데이터 타입을 받는다. itemIdentifier는 cell에 대한 item의 identifier를 의미한다. 해당 클로저는 nil이 아닌 구성된 cell 객체를 반환하며, cell provider는 collectionView에게 유효한 cell 객체를 반환해야만 한다!

##  6. snapshot을 dataSource에 apply

```swift=
var snapshot = NSDiffableDataSourceSnapshot<Section, Animal>()
snapshot.appendSections([.main])
snapshot.appendItems(animalData)
dataSource.apply(snapshot)
```

- `NSDiffableDataSourceSnapshot` : 특정 시점에서 view 내의 데이터의 state를 나타낸다.

diffableDataSources는 snapshot을 사용해서 collectionview나 tableview에 데이터를 제공한다. snapshot을 통해, 뷰에 보여질 데이터의 초기 상태를 정하고 추후에 데이터를 업데이트 해줄 수 있다.

snapshot의 데이터는 보여주고 싶은 순서로 section과 items로 구성된다. add/delete 혹은 움직여서 section/items를 보여줄 수 있다.

> !! 중요 !!
> 각 section들과 item들은 Hashable 프로토콜을 채택하는 unique한 identifiers를 가지고 있어야 한다.


snapshot을 사용하여 데이터를 보여주기 위한 순서는 다음과 같다.

> 1. snapshot을 만들고 표시하려는 데이터 state로 채운다.
> 2. UI의 변경사항을 반영하도록 snapshot을 적용한다.

snapshot은 아래 방법으로 만들고 구성할 수 있다.

- 빈 snapshot을 만들고 section들과 item들을 append한다
- diffableDataSource의 `snapshot()` 메서드를 호출하여 현재 snapshot을 가져온다. 그런 다음 표시할 데이터의 새 state를 반영하도록 snapshot을 수정한다.

###   6-1. 모델에 Hashable 프로토콜 채택 및 identifier 프로퍼티 생성

```swift=
struct Animal: Codable, Hashable {
    let name, latinName, animalType, activeTime: String
    let lengthMin, lengthMax, weightMin, weightMax: String
    let lifespan, habitat, diet, geoRange: String
    let imageLink: String
    let id: Int
 
    let identifier = UUID()
    
    enum CodingKeys: String, CodingKey {
        case name
        case latinName = "latin_name"
        case animalType = "animal_type"
        case activeTime = "active_time"
        case lengthMin = "length_min"
        case lengthMax = "length_max"
        case weightMin = "weight_min"
        case weightMax = "weight_max"
        case lifespan, habitat, diet
        case geoRange = "geo_range"
        case imageLink = "image_link"
        case id
    }
}

```

데이터는 Hashable 프로토콜을 채택하는 유일한 identifier를 가져야 한다.

---


![](https://i.imgur.com/hg3XEw7.png)

앞서 봤던 것 처럼 리스트 형태의 뷰를 볼 수 있다!

이렇게 collection view를 사용하여 list 형태로 나타내봤다. 만약 뷰가 grid 형태로도 보여져야 한다면 tableview 1개, collection view 1개가 아니라 collection view만을 사용하여 조금 더 편하게 구현해 볼 수 있을 것 같다. 

끄읕

---

## 고민된 점 
- collectionView로 list 형태를 어떻게 구성하는가?

## 해결 방법 
- apple이 제공하는 modern collectionview 코드 분석 및 예제 구현

---

**Ref**

https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/implementing_modern_collection_views

