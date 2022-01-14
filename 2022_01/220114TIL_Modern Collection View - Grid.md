# Modern Collection View - Grid

## 220114_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Modern Collection View - Grid 구현

앞선 글([[iOS] Modern Collection View - List 구현](https://leechamin.tistory.com/555#Modern%--Collection%--View%--%EA%B-%AC%ED%--%--))에서는 collection View를 활용하여 list 형태의 뷰를 구현하는 방식을 봤었다. 이제는 원래 collection View의 목적이라고도 할 수 있는(?) grid(격자)형태의 뷰를 구성해보자.
(List 형태를 구현할 때와 코드가 거의 유사하니 이전 글을 참고해도 좋을 것 같다.)

이 또한 iOS 14이후의 버전에 대해 지원해주는 modern한 구현 방식이 있는데, 이를 알아보기 전에 기존 방식으로는 어떻게 구현했는지 살펴보자. 

## 기존 collectionView 구현 방식

우선 그릴 화면을 먼저 보자. 

![](https://i.imgur.com/zcGfv5V.gif)

가장 흔히 볼 수 있는 grid 형태의 뷰이다. 이를 구현하기 위해서는 어떠한 순서대로 구현하면 좋을지 먼저 살펴보자. 구현 방식에는 개개인마다 차이가 있다는 점을 유의하면서 보면 좋을 것 같다.


1. `UICollectionView`를 생성해둔다.
    1-1. 이 때 클로저 방식으로 생성하면서 속성들을 모두 부여해준다.
    1-2. 기본적으로 `UICollectionViewFlowLayout`으로 기본적인 레이아웃을 잡아준다. 
    1-3. 직접 만든 cell을 등록시켜준다. 
2. `UICollectionViewCell`을 생성한다. 
3. `UICollectionViewDataSource`를 `ViewController`에 채택한다.
    3-1. `collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int` 메서드를 통해 한 섹션에 몇 개의 아이템이 올지 정해준다. 
    3-2. `collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell` 메서드를 통해 cell을 dequeue해주고, cell에 데이터를 할당하고, 꾸며준다. 
    3-3. 커스텀한 레이아웃을 위해 `UICollectionViewDelegateFlowLayout`를 채택해주고, `collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize`메서드를 통해 item별 사이즈를 정해준다. 
    
세부적으로 스텝을 구분해보긴 했지만 크게 보면 3가지 부분으로 볼 수 있을 것 같다. 

1. `UICollectionView`의 데이터
2. `UICollectionView`의 레이아웃
3. `UICollectionViewCell` 생성


1. `UICollectionView`를 생성해둔다.
    1-1. 이 때 클로저 방식으로 생성하면서 속성들을 모두 부여해준다.
    1-2. 기본적으로 `UICollectionViewFlowLayout`으로 기본적인 레이아웃을 잡아준다. 
    1-3. 직접 만든 cell을 등록시켜준다. 
2. `UICollectionViewCell`을 생성한다. 
3. `UICollectionViewDataSource`를 `ViewController`에 채택한다.
    3-1. `collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int` 메서드를 통해 한 섹션에 몇 개의 아이템이 올지 정해준다. 
    3-2. `collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell` 메서드를 통해 cell을 dequeue해주고, cell에 데이터를 할당하고, 꾸며준다. 
    3-3. 커스텀한 레이아웃을 위해 `UICollectionViewDelegateFlowLayout`를 채택해주고, `collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize`메서드를 통해 item별 사이즈를 정해준다. 
    
세부적으로 스텝을 구분해보긴 했지만 크게 보면 3가지 부분으로 볼 수 있을 것 같다. 

1. `UICollectionView`의 데이터
2. `UICollectionView`의 레이아웃
3. `UICollectionViewCell` 생성


전체 코드를 보자. 

```swift
import UIKit

class AnimalViewController: UIViewController {
    let api = APIService()
    var animalData: [Animal] = []
    let collectionView: UICollectionView = {
        let flowLayout = UICollectionViewFlowLayout()
        flowLayout.scrollDirection = .vertical
        
        let cv = UICollectionView(frame: .zero, collectionViewLayout: flowLayout)
        cv.register(AnimalCollectionViewCell.self, forCellWithReuseIdentifier: AnimalCollectionViewCell.identifier)
        cv.contentInset = UIEdgeInsets(top: 10, left: 10, bottom: 10, right: 10)
        return cv
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.title = "ZOO"
        view.backgroundColor = .white
        getData()
    }
    
    func getData() {
        api.getAnimalData { result in
            switch result {
            case .success(let animals):
                self.animalData = animals
                DispatchQueue.main.async {
                    self.setupCollectionView()
                }
            case .failure(let error):
                print(error)
            }
        }
    }
    
    func setupCollectionView() {
        collectionView.dataSource = self
        collectionView.delegate = self
        
        view.addSubview(collectionView)
        
        collectionView.translatesAutoresizingMaskIntoConstraints = false
        
        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            collectionView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            collectionView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            collectionView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
        ])
        
    }
}


extension AnimalViewController: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return animalData.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        guard let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath) as? AnimalCollectionViewCell else {
            return AnimalCollectionViewCell()
        }
        
        let data = animalData[indexPath.item]
        cell.configCell(with: data)
        return cell
    }
}

extension AnimalViewController: UICollectionViewDelegateFlowLayout {
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        let margin: CGFloat = 10
        return CGSize(width: (self.view.frame.width - (margin * 3)) / 2, height: self.view.frame.height / 4)
    }
}
```

```swift 
import UIKit

class AnimalCollectionViewCell: UICollectionViewCell {
    static let identifier = "cell"
    
    let nameLabel: UILabel = {
        let lb = UILabel()
        lb.textColor = .white
        lb.textAlignment = .right
        lb.numberOfLines = 0
        return lb
    }()
    
    let animalImage: UIImageView = {
        let iv = UIImageView()
        iv.contentMode = .scaleAspectFill
        return iv
    }()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        layout()
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
    }
    
    func layout() {
        [animalImage, nameLabel].forEach {
            contentView.addSubview($0)
            $0.translatesAutoresizingMaskIntoConstraints = false
        }
        
        layer.masksToBounds = true
        layer.cornerRadius = 10
        
        NSLayoutConstraint.activate([
            animalImage.topAnchor.constraint(equalTo: contentView.topAnchor),
            animalImage.leadingAnchor.constraint(equalTo: contentView.leadingAnchor),
            animalImage.trailingAnchor.constraint(equalTo: contentView.trailingAnchor),
            animalImage.bottomAnchor.constraint(equalTo: contentView.bottomAnchor),
            
            nameLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 10),
            nameLabel.bottomAnchor.constraint(equalTo: contentView.bottomAnchor),
            nameLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -10),
        ])
    }
    
    func configCell(with animal: Animal) {
        ImageLoader.loadImage(from: animal.imageLink) { image in
            self.animalImage.image = image
        }
        
        let attrString = NSAttributedString(
            string: animal.name,
            attributes: [
                NSAttributedString.Key.strokeColor: UIColor.black,
                NSAttributedString.Key.foregroundColor: UIColor.white,
                NSAttributedString.Key.strokeWidth: -2.0,
                NSAttributedString.Key.font: UIFont(name: "Helvetica-Bold", size: 30)
            ]
        )
        
        nameLabel.attributedText = attrString
    }
}

```

기존의 방식은 잘 알 것이라고 생각한다...!
(사실 비교를 위해 기존 방식의 코드를 넣은거긴 하니까... 세세한 설명들은 스킵...!)

List를 구현할 때는 전용 cell을 만들어서 사용했지만, 다행인 건 grid 형태를 구현할 때는 modern 방식에서도 기존 cell을 그대로 사용할 수 있다! 

## Modern CollectionView - Grid 구현 

자 이제 본 목적인 modern하게 grid형태의 collection view를 그리는 방법을 살펴보자. 

이 또한 먼저 순서를 살펴보자!

1. `UICollectionView` 타입의 프로퍼티 정의
2. `UICollectionViewDiffableDataSource` 타입의 프로퍼티 정의
3. `UICollectionViewCompositionalLayout` 레이아웃 생성
4. `UICollectionView` 인스턴스 생성 및 레이아웃 적용
5. `UICollectionViewDiffableDataSource` 구현
    5-1. `CellRegistration` 구현
    5-2. `UICollectionViewDiffableDataSource` 인스턴스 생성 및 cellProvider의 `dequeueConfiguredReusableCell` 구현
    5-3. `NSDiffableDataSourceSnapShot` 생성
    
    
1. UICollectionView 타입의 프로퍼티 정의
2. UICollectionViewDiffableDataSource 타입의 프로퍼티 정의
3. UICollectionViewCompositionalLayout 레이아웃 생성
4. UICollectionView 인스턴스 생성 및 레이아웃 적용
5. UICollectionViewDiffableDataSource 구현
    5-1. CellRegistration 구현
    5-2. UICollectionViewDiffableDataSource 인스턴스 생성 및 cellProvider의 dequeueConfiguredReusableCell 구현
    5-3. NSDiffableDataSourceSnapShot 생성    
    
하나씩 살펴보자.

### 1. `UICollectionView` 타입의 프로퍼티 정의

```swift
var gridCollectionView: UICollectionView!
```

이건 뭐..! 
(사실 암시적 추출 옵셔널보다는 옵셔널(`?`)을 사용하는 것이 더 안전하지만 편의를 위해 암시적 추출 옵셔널을 사용하여 진행합니다!)

### 2. `UICollectionViewDiffableDataSource` 타입의 프로퍼티 정의

```swift=
var dataSource: UICollectionViewDiffableDataSource<Section, Animal>!
```

사용할 section과 데이터 타입을 명시해줍니다. 

### 3. `UICollectionViewCompositionalLayout` 레이아웃 생성


이제 `item -> group -> section` 순서대로 생성해주고 section을 사용하여 `UICollectionViewCompositionalLayout` 인스턴스를 하나 만들어 반환해줍니다. 

앞서서 먼저 item/group/section 그리고 이들의 사이즈를 정해주기 위한 `NSCollectionLayoutSize`에 대해서 알아보고 코드를 보자.


- `NSCollectionLayoutSize`
    - collection view내 요소의 너비와 높이를 나타낸다. 
    - width/height diemension을 통해 크기를 줄 수 있다. 
    - `.fractionalWidth`: 포함하는 그룹 너비에 상대적인 크기이다. `1.0` 이라는 것은 동일한 크기를 주겠다는 것. 1보다 큰 수를 주면 그 배수만큼 값이 적용된다. `0.2`를 적으면 20%만큼의 크기를 가지겠다는 의미가 된다!
    - `.fractionalHeight`: 위의 높이 버전!
    - `.estimated()` : 데이터가 로딩되거나 응답이 바뀌는 경우와 같이 런타임에 컨텐츠의 크기가 변경될 수 있기에 추정값을 넣어준다. 초기 추정 크기를 제공하면 시스템은 실제 값을 나중에 연산한다. 
    - `.absolute()`: 절대적인 크기를 입력하여 사용하는 방식이다.



### `NSCollectionLayoutItem`

![](https://i.imgur.com/fw8P4pY.png)

- collection view 레이아웃에서 가장 기본이 되는 구성 요소이다. 
- item은 collection view내 컨텐츠의 개별적인 공간의 배열이나 공간, 크기를 나타내는 청사진이다. item은 스크린에 보여질 단일 뷰이며, 일반적으로 cell이라고 봐도 된다. 하지만 item은 헤더, 푸터, 데코뷰가 될 수도 있다. 



### `NSCollectionLayoutGroup`

![](https://i.imgur.com/YdOrKTf.png)

- 방향에 따라 item을 배치하는 item들의 집합의 컨테이너이다. 
- 그룹은 collection view의 item이 서로간에 어떻게 배치되는지를 결정한다. 그룹은 item을 horizontal/vertical 혹은 커스텀하게 배치할 지 결정할 수 있다. 그룹은 item들이 렌더링되는 방법에 대한 규칙을 정하지만 스스로(그룹)는 내용을 렌더링하지 않는다.(즉 아이템이 무조건 있어야 한다는 것 같다!)
- 그룹은 `NSCollectionLayoutItem`의 하위 클래스이기 때문에 아이템처럼 사용할 수 있다. 그룹을 다른 아이템들과 결합하거나 아래와 같이 그룹을 더 복잡한 레이아웃으로도 사용해볼 수 있다. 

![](https://i.imgur.com/zkSTHbX.png)



### `NSCollectionLayoutSection`

![](https://i.imgur.com/PyV45TN.png)

- 그룹들을 고유한 시각적 묶음으로 결합하는 컨테이너이다. 
- 즉 그룹들의 모음집이라고도 볼 수 있다. 
- collection view의 레이아웃은 하나 이상의 섹션을 가진다. 섹션은 레이아웃을 서로 다른 조각으로 구분할 수 있는 방법을 제공한다. 
- 각 섹션은 collection view의 다른 섹션들과 동일한 레이아웃을 가질 수 있고, 다를 수도 있다. 섹션의 레이아웃은 그룹의 프로퍼티에 의해 결정된다. 
- 각 섹션은 다른 섹션과 구분하기 위해 배경, 헤더, 푸터를 가질 수 있다. 

자 이제 코드로 하나하나 쌓아보면 이해가 더 잘 될 것이다. 

```swift
func createLayout() -> UICollectionViewCompositionalLayout{
    let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0), heightDimension: .fractionalHeight(1.0))
    let item = NSCollectionLayoutItem(layoutSize: itemSize)

    let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0), heightDimension: .absolute(self.view.frame.height * 0.25))
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitem: item, count: 2)
    group.interItemSpacing = .fixed(10)

    let section = NSCollectionLayoutSection(group: group)
    section.interGroupSpacing = CGFloat(10)
    section.contentInsets = NSDirectionalEdgeInsets(top: 0, leading: 10, bottom: 0, trailing: 10)

    let layout = UICollectionViewCompositionalLayout(section: section)

    return layout
}
```

앞서 설명한 것 처럼 item -> group -> section 순으로 쌓고 layout을 만드는 것을 볼 수 있다. 

이제 `UICollectionViewCompositionalLayout` 인스턴스를 만들었으니 이를 활용하여 `UICollectionView` 인스턴스를 생성해보자. 

### 4. `UICollectionView` 인스턴스 생성 및 레이아웃 적용

앞서 만든 레이아웃을 활용하여 collection view에 적용하고 collection view의 레이아웃을 잡아주면 된다! 

```swift
func createGridCollectionView() {
    gridCollectionView = UICollectionView(frame: .zero, collectionViewLayout: createLayout())

    view.addSubview(gridCollectionView)
    gridCollectionView.translatesAutoresizingMaskIntoConstraints = false

    NSLayoutConstraint.activate([
        gridCollectionView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
        gridCollectionView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
        gridCollectionView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
        gridCollectionView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
    ])
}
```

### 5. `UICollectionViewDiffableDataSource` 구현

이제 data source를 다룰 시간이다. 
이 부분은 list를 구현할 때와 거의 사실상 동일하다. 

```swift
func configDataSource() {
    // 5-1. `CellRegistration` 구현
    let cellRegistration = UICollectionView.CellRegistration<AnimalCollectionViewCell, Animal> { cell, indexPath, animal in
        cell.configCell(with: animal)
    }

    // 5-2. `UICollectionViewDiffableDataSource` 인스턴스 생성 및 cellProvider의 `dequeueConfiguredReusableCell` 구현
    dataSource = UICollectionViewDiffableDataSource<Section, Animal>(collectionView: gridCollectionView, cellProvider: { (collectionView, indexPath, itemIdentifier) -> UICollectionViewCell? in
        return collectionView.dequeueConfiguredReusableCell(using: cellRegistration, for: indexPath, item: itemIdentifier)
    })

    // 5-3. `NSDiffableDataSourceSnapShot` 생성 
    var snapShot = NSDiffableDataSourceSnapshot<Section, Animal>()
    snapShot.appendSections([.main])
    snapShot.appendItems(animalData)
    dataSource.apply(snapShot)
}
```
    
    
이 부분은 크게 어렵지 않을 것이다. 

cell을 등록해주고, cell을 반환해주고, cell에 들어갈 데이터를 snapshot을 통해 넣어주는 그러한 일련의 과정이다. 
각 메서드에 대한 자세한 설명은 [[iOS] Modern Collection View - List 구현](https://leechamin.tistory.com/555#Modern%--Collection%--View%--%EA%B-%AC%ED%--%--)를 참고하자. 

---

이제 구현한 화면을 볼 시간!

![](https://i.imgur.com/sRtvnPV.gif)

이전에 만든 코드로 영상을 딴게 아닐까 싶을 정도로 유사하게, 아니 똑같이 만들 수 있다. 

modern하게 grid collection view 구현하는 방법의 장점을 생각해보면 2가지 정도 있는 것 같다.

- 직관적으로 grid내 계층을 그릴 수 있다. 
- `DiffableDataSource`로 데이터와 UI를 쉽게 관리할 수 있다. 

코드 자체의 길이는 비슷비슷한 것 같지만 modern의 방식이 조금 더 코드의 흐름을 읽기에도 좋은 것 같다는 생각이 든다. 

물론 iOS 14 이상에서만 가능한 코드이지만... 계속 버전이 올라감에 따라 이 방법이 표준이 될 날도 얼마 남지 않을 것 같다. 

약간 SwiftUI에서 뷰를 하나하나 쌓은 것 처럼, UIKit을 가지고 유사하게 구현한(?) 느낌을 개인적으로 받았다. 

---

아무튼! 지금까지 grid 형태의 collection view를 만드는 두 가지 방법을 봤다. 각자 생각하는 장단점에 의거해서 구현 방식을 선택하면 될 것 같다! 

끄읕

### Todo 
- 캐러셀 구현 

## 고민된 점 
- modern CollectionView 구현 방법
- 기존 방법과의 차이

## 해결 방법 
- 직접 구현해보고 장점 정리
---

**Ref**

https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/implementing_modern_collection_views
