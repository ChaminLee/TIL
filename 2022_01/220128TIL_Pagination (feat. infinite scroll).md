# Pagination (feat. infinite scroll)

## 220128_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Pagination 

페이지네이션을 어떻게 구현해야하나 고민을 많이 해본 것 같다. 

우선 현재 프로젝트가 modern collectionview로 구현되어 있어서 최대한 기존의 UICollectionViewDataSource나 UICollectionViewDelegateFlowLayout을 채택하지 않고 구현해보고자 했다! 

우선 생각한 방식은 두 가지가 있었다. 

1. 스크롤뷰의 contentOffset의 y를 확인하여 일정 기준치를 넘어서면 업데이트
2. CollectionView의 footer를 보여줄 때 업데이트
3. CollectionView의 prefetch 메서드에서 다음 페이지가 있고, 현재 indexPath가 마지막 데이터인지 확인 후 업데이트 

우선 1번의 방식으로 할 수도 있었겠지만 offset으로 하기 보다는 현재 보이는 데이터가 마지막인지를 코드로도 명확하게 확인할 수 있는 2,3번으로 시도를 해봤다. 

#### 2번. CollectionView의 footer를 보여줄 때 업데이트

우선 collectionView에 footer를 달아주기 위해 UICollectionViewLayout을 생성할 때 다음과 같이 추가적인 코드를 작성해줬다. 

```swift
func createListLayout() -> UICollectionViewLayout {
    let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                          heightDimension: .absolute(self.view.frame.height/10))
    let item = NSCollectionLayoutItem(layoutSize: itemSize)

    let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                          heightDimension: .absolute(self.view.frame.height/10))
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])

    let section = NSCollectionLayoutSection(group: group)

    let footerSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                                 heightDimension: .estimated(44))

    let sectionFooter = NSCollectionLayoutBoundarySupplementaryItem(
        layoutSize: footerSize,
        elementKind: UICollectionView.elementKindSectionFooter, alignment: .bottom)
    section.boundarySupplementaryItems = [sectionFooter]

    let layout = UICollectionViewCompositionalLayout(section: section)
    return layout
}
```

엥?? 리스트를 만드는데 왜 굳이 UICompositionalLayout을 사용했을까...?

중간에 우여곡절이 있었는데... 원래 list형태의 UICollectionView를 만들기 위해서는 간략하게 아래의 코드로도 가능했다. 

```swift
private func createLayout() -> UICollectionViewLayout {
    var config = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
    config.footerMode = .supplementary
    return UICollectionViewCompositionalLayout.list(using: config)
}
```

위와 같은 기존 방식에서 `.footerMode`에 `.supplementary`만 넣어주면 간단하게 footer도 추가할 수 있었다. 

하지만....

구현하고 보니 footer가 고정(pin)되어 있는 모습을 볼 수 있었다. 다른 프로퍼티나 메서드를 사용해서 pin을 해제해보려고 했는데 왜 안되나 하던 순간... 

![](https://i.imgur.com/97N6jNc.png)

이런... 

configuration의 appearance가 `.plain` 혹은 `.sidebarPlain`이면 무조건 footer가 pin된다고 공식문서에 적혀있었다... 

다음에 pin된 footer를 만들어야 할 때는 잘 사용할 수 있겠지만... 지금은 pin이 필요없는데 해제하지도 못하고... 그래서 앞서 본 것 같이 UICompositionalLayout으로 구성을 하게 되었다. 

아무튼 다시 돌아와서 이제 footer를 사용하기 위해`UICollectionViewDiffableDataSource`의 `supplementayViewProvider`를 구현해주었다. 

우선은 다음 페이지에 데이터가 있는 지 `hasNextPage`로 확인을 해준다. 만약 존재한다면 `fetchNextPageProductData()` 메서드로 다음 페이지의 데이터를 기존의 데이터에 이어 붙여주었다. 

그리고 footer의 activity indicator를 시작하도록 하면, datasource의 snapshot을 활용하여 뷰가 업데이트 되면서 새로운 데이터가 추가된다. 

```swift
private var listDataSource: UICollectionViewDiffableDataSource<ProductSection, ProductDetail>?

listDataSource?.supplementaryViewProvider = { (_, _, indexPath) -> UICollectionReusableView? in
    if self.hasNextPage {
	    self.fetchNextPageProductData()
        return self.productListCollectionView.dequeueConfiguredReusableSupplementary(using: footerRegistration, for: indexPath)
    } else {
        let loading = self.productListCollectionView.dequeueConfiguredReusableSupplementary(using: footerRegistration, for: indexPath)
        loading.loadingIndicator.stopAnimating()
        return loading
    }
}
```

다음 페이지에 데이터가 없을 경우에는 footer의 activity indicator를 멈추어 사용자에게 더 이상의 데이터가 없음을 알려주도록 했다. 

이 방법으로 구현을 하니 페이지네이션은 잘 되기는 했는데... 뭔가 footer를 보여줄 때 업데이트를 한다는게 조금 어색하기도 하고, 타이밍만 끼워맞춘 느낌이 들어서 다음 3번의 방법으로 시도해보았다. 

또, 최적화가 되지 않아서인지 스크롤에 버벅임이 있어서 다른 방법을 찾게 되었다..!

#### 3. prefetch 사용하여 업데이트 하기 

이전에 footer를 넣어준 것 까지는 동일하다. footer에서는 여전히 activity indicator가 돌고있고, 데이터를 업데이트 할 위치만 달라지게 되는 것이다. 

우선은 prefetch를 하기 위해 `UICollectionViewDataSourcePrefetching` 프로토콜에 대해서 알아봤다. 

`UICollectionViewDataSourcePrefetching` 프로토콜은 collectionview가 필요로 하는 데이터에 대한 사전 통지를 제공하여 비동기 데이터 로드 작업을 실행(trigger)할 수 있는 역할을 한다. 

쉽게 말하면, `cellForItemAt` 처럼 실제로 데이터를 가져다가 사용하기 이전에 데이터를 불러오는 작업을 해줄 수 있는 공간이다. 

말 그대로 이전에 준비하기 때문에 딜레이가 줄어들 것이며, 스크롤이 버벅거리는 문제 또한 해결해 줄 수 있을 것이라 생각했다. 

문서에서는 아래의 순서대로 prefetch를 사용할 것을 말하고 있다. 

1. collectionview와 이에 사용할 dataSource를 생성해라.
2. `UICollectionViewDataSourcePrefetching` 프로토콜을 채택하는 객체를 생성하고, 이를 collectionview의 프로퍼티인 `prefetchDataSource`에 할당해라. 
3. `collectionView(_:prefetchItemsAt:)` 메서드에서 지정된 indexPath에 있는 cell에 필요한 데이터의 비동기 로드를 시작한다. 
4. `collectionView(_:cellForItemAt:)`에서는 앞서 prefetch된 데이터를 사용하여 cell을 표시하도록 준비한다. 
5. 만약 데이터를 필요로 하지 않는 경우, `collectionView(_:cancelPrefetchingForItemsAt:)`에서 이전에 prefetch한 요청을 취소해준다. 

1번부터 차근차근 보자. 
(문서처럼 완전히 똑같지는 않고 일부 로직을 다르게 한 부분이 있다!)

#### 1. collectionView & dataSource 생성

```swift
private var productListCollectionView: UICollectionView!
private var listDataSource: UICollectionViewDiffableDataSource<ProductSection, ProductDetail>?
```

이건 앞서서 해뒀던 거라... 우선 생성만 해주면 된다. 

#### 2. `UICollectionViewDataSourcePrefetching`  채택

```swift
override func viewDidLoad() {
    ...
    productListCollectionView.prefetchDataSource = self
}

extension MainViewController: UICollectionViewDataSourcePrefetching{ 
    ... 
}
```

Viewcontroller에 해당 프로토콜을 채택해주고, 우선은 viewDidLoad에서 `prefetchDataSource`에 스스로를 할당해주었다!

#### 3. `collectionView(_:prefetchItemsAt:)` 구현

이 부분이 아마 핵심로직이 되지 않을까 싶다!

```swift
func collectionView(_ collectionView: UICollectionView, prefetchItemsAt indexPaths: [IndexPath]) {
    let customQueue = DispatchQueue(label: "custom", attributes: .concurrent)
    if indexPaths.last?.row == productData.count - 1 && hasNextPage {
        customQueue.async {
            self.fetchNextPageProductData()
        }
    }
}
```

우선은 비동기적으로 작업해주기 위해 커스텀하게 dispatchQueue를 생성해주었다. 

커스텀한 큐를 만들어야 해당 작업만을 위해 작동할 것이라고 생각해서, 기본적인 global 큐를 사용하지 않았다! 확실하진 않지만 global 큐는 뭔가 다른 용도로도 시스템에서 사용할 것 같아서, 최대한 빠르게 우선적으로 처리해주고자 커스텀 큐를 만들었다. 

아무튼! 로직은 간단하다.

현재 보유하고 있는 데이터의 개수와 현재 `indexPath.row`를 비교하고 동시에 다음 페이지가 있는지 확인하는 것이 핵심이다. 

indexPath의 row가 (현재 데이터의 개수 - 1)개라면 현재 row가 마지막 데이터를 나타내고 있음을 알 수 있다. 이와 동시에 다음 페이지에 데이터가 있는지 확인해서 위 조건을 통과한다면 아까 만들어준 커스텀 큐에서 `fetchNextPageProductData()` 메서드를 실행하도록 해주었다. 

`fetchNextPageProductData()` 메서드는 서버에 요청하여 다음 페이지의 데이터를 가져오고 기존 dataSource의 snapshot을 사용하여 데이터를 업데이트 해준다. 

```swift
func configSnapShot(with dataSource: UICollectionViewDiffableDataSource<ProductSection, ProductDetail>?) {
    var snapshot = NSDiffableDataSourceSnapshot<ProductSection, ProductDetail>()
    snapshot.appendSections([.main])
    snapshot.appendItems(productData)
    dataSource?.apply(snapshot)
}
```

위 메서드가 `fetchNextPageProductData()` 메서드에서 정상적으로 데이터를 받아오게 되면 불리게 된다.

#### 4. `collectionView(_:cellForItemAt:)` 구현

문서에서는 `collectionView(_:cellForItemAt:)`를 구현하라고 되어있었지만... modern collectionview 방식으로 구현했으니... cellProvider를 구현해주면 된다. 

메서드 이름만 다르지 같은 역할을 해준다!

```swift
listDataSource = UICollectionViewDiffableDataSource<ProductSection, ProductDetail>(collectionView: productListCollectionView) { (collectionView, indexPath, product) -> UICollectionViewCell? in
    guard let cell = collectionView.dequeueConfiguredReusableCell(using: cellRegistration, for: indexPath, item: product) as? ProductListLayoutCell else {
        return nil
    }
    DispatchQueue.main.async {
        cell.updateWithProduct(from: product)
    }
    return cell
}
```

우선 cell을 dequeue해주고, cell의 메서드를 사용해서 내부 UI 요소들을 업데이트 해준다. 

그러면 cell을 반환할 때마다 새로운 데이터로 업데이트하게 되어 새로운 cell을 확인할 수 있다. 

아 그리고 아직까지 prefetch한 데이터가 필요없는 경우를 찾지 못해서... 5번은 스킵했다! (추후에 더 무거운 데이터를 prefetch 하게 되는 경우라면 고려해볼 수 있을 것 같다. 필요없는데 리소스를 많이 들여서 받아오는게 비효율적이기 때문이다!)

그래도 스크롤이 조금 버벅이길래 이미지를 로드해줄 때, 동기적인 방식이 아니라 `URLSession`을 활용하여 비동기적으로 받아오도록 수정하였다. 

그렇게 하니 좀 부드럽게 스크롤되면서 새 데이터를 로드해오는 모습을 확인할 수 있었다. 

![](https://i.imgur.com/OV760oy.gif)

뷰를 조금 더 자연스럽게 업데이트 할 수 있게 최적화를 해줘야 할 것 같다...!

## 고민된 점 
- 무한 스크롤 구현은 어떻게 할까?

## 해결 방법 
- prefetch를 통한 사전 데이터 업데이트!
---

**Ref**

https://developer.apple.com/documentation/uikit/uicollectionviewdatasourceprefetching

