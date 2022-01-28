# 화면 및 데이터 업데이트 방법 (feat. 인스타 새 게시물), Animation

## 220127_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용


### 1. 화면 및 데이터 업데이트 방법 (feat. 인스타 새 게시물)

인스타를 하다보면 새로운 게시물이 등록되었다는 것을 알려주기 위해 "새 게시물" 버튼을 띄워준다. 

![](https://i.imgur.com/v4ogUMM.jpg)

이 버튼을 누르면 메인 화면은 업데이트 되고, 스크롤이 맨 위로 향하게 된다. 이를 현재 진행하고 있는 프로젝트에서 구현해보기로 이야기했다. 

대략적인 로직은 다음과 같다. 서버에 요청해서 얻은 데이터와 현재 보유하고 있는 데이터를 비교하여, 얻은 데이터가 비교적 최신이라면 업데이트를 할 수 있게 "새 게시물" 버튼을 띄워준다. 

이후 버튼을 클릭하게 되면, 메인 화면을 업데이트 시켜주고 스크롤을 맨 위로 향하게 해준다. 혹시나 버튼을 누르지 않고 다른 화면으로 이동했을 경우에는 버튼을 보이지 않게 해준다. 

이를 순서대로 나타내보자. 

1. N초 이후에 서버에 데이터를 요청하도록 한다.
2. 1번의 기능을 `viewWillAppear`에 넣어, 메인 뷰가 보여지려고 할 때마다 서버에 요청하도록 한다. 
3. 서버의 데이터와 가지고 있던 데이터를 비교한다.
4. 서버의 데이터의 최근 데이터의 id가 더 높다면 최신 게시물이 등록된 것으로 판단하여 업데이트 버튼을 띄운다.
5. 버튼을 클릭하면 업데이트 후 화면을 상단으로 이동시켜준다. 

#### 1. N초 이후에 서버에 요청 보내기 

바로 코드를 보자
```swift
private var refreshTimer: Timer?
private var refreshTime = 0

private func startRefreshTimer(initialTime: Int) {
    refreshTime = initialTime
    refreshTimer = Timer.scheduledTimer(timeInterval: 1, target: self, selector: #selector(timeRefresh), userInfo: nil, repeats: true)
}

@objc func timeRefresh() {
    if refreshTime == 0 {
        updateProductDataPeriodically()
        refreshTimer?.invalidate()
        return
    }

    refreshTime -= 1
}
```

데이터를 확인해주는 `updateProductDataPeriodically()`의 내용은 3번에서 보자. 

#### 2. `viewWillAppear`에 요청하는 메서드 삽입

```swift
override func viewWillAppear(_ animated: Bool) {
    super.viewWillAppear(animated)
    startRefreshTimer(initialTime: 3)
}
```

`viewWillAppear`가 호출되고나서 n초 이후에 서버에 요청하게 된다. 


#### 3. 서버의 데이터와 현재 데이터를 비교한다. 

```swift
private var productData: [ProductDetail] = []
private let apiService = APIService()

private func updateProductDataPeriodically() {
    apiService.retrieveProductList(pageNo: RequestInformation.pageNumber, itemsPerPage: RequestInformation.itemsPerPage) { result in
        switch result {
        case .success(let data):
            guard let previousProductData = self.productData.first,
                  let currentProductData = data.pages.first else {
                return
            }

            if previousProductData.id < currentProductData.id {
                DispatchQueue.main.async {
                    self.productData = data.pages
                    self.showRefreshButton()
                }
                return
            }
        case .failure(let error):
            print(error)
        }
    }
}
```

코드가 길긴 하지만, 역할은 단순하다. 이전의 데이터와 서버에 요청한 따끈따끈한 데이터와 비교해서 최근 등록된 게시글이 있는 지 확인하는 역할을 한다. 

#### 4. 최신 데이터가 있다면 업데이트 버튼을 띄워준다.

위 코드에서 이 부분에 해당한다. 

```swift
if previousProductData.id < currentProductData.id {
    DispatchQueue.main.async {
        self.productData = data.pages
        self.showRefreshButton()
    }
    return
}
```
이 때 만약 요청을 통해 받은 데이터의 id가 더 높다면 최근 게시물이 있다는 것이기 때문에 버튼을 띄워줄 수 있도록 한다. 

#### 5. 버튼 액션을 통해 업데이트 하고, 상단으로 이동한다. 

먼저 버튼을 만들어준다. 

```swift
private lazy var refreshButton: UIButton = {
    let button = UIButton(frame: CGRect(x: UIScreen.main.bounds.midX - 40, y: 0, width: 80, height: 30))
    button.setTitle("새 게시글", for: .normal)
    button.setTitleColor(.black, for: .normal)
    button.titleLabel?.font = .preferredFont(forTextStyle: .subheadline)
    button.backgroundColor = .white
    button.layer.shadowColor = UIColor.gray.cgColor
    button.layer.shadowOffset = .zero
    button.layer.shadowOpacity = 0.5
    button.layer.masksToBounds = false
    button.layer.cornerRadius = 10
    button.contentEdgeInsets = UIEdgeInsets(top: 5, left: 5, bottom: 5, right: 5)
    button.addTarget(self, action: #selector(didTapRefreshButton), for: .touchUpInside)
    return button
}()
```

인스타에서 본 버튼과 유사하게 만들어주기 위해, 그림자를 설정하고, 모서리도 둥글게 해봤다.


```swift
private func showRefreshButton() {
    self.view.addSubview(refreshButton)
    self.refreshButton.alpha = 0

    UIView.animate(withDuration: 0.2, delay: 0, options: [.curveEaseInOut]) {
        self.refreshButton.alpha = 1
        self.refreshButton.frame.origin.y += self.navigationController?.navigationBar.frame.maxY ?? 0
    }
}

@objc func didTapRefreshButton() {
    self.configProductCollectionViewDataSource()

    UIView.animate(withDuration: 0.2, delay: 0, options: [.curveEaseInOut]) {
        self.resetRefreshButton()
    }

    self.productListCollectionView.setContentOffset(.zero, animated: true)
    self.productGridCollectionView.setContentOffset(.zero, animated: true)
}

private func resetRefreshButton() {
    self.refreshButton.alpha = 0
    self.refreshButton.frame.origin.y = 0
}
```

이제 버튼을 띄우는 메서드를 볼 수 있다. 

우선은 투명한 상태로 들어가고, 애니메이션을 통해 점점 불투명해지고 y축을 기준으로 점점 아래로 이동하다가 navigationBar 아래에 딱 붙는 순간 멈추도록 해줬다. 

그리고 버튼을 클릭하면 요청하여 받은 데이터를 통해 `UICollectionViewDiffableDataSource`를 업데이트해주고, 버튼의 속성, 위치를 초기화 해준다. 

그리고 collectionView의 스크롤을 상단으로 이동시켜서 새로 업데이트 된 게시글을 확인할 수 있게 해준다!

![](https://i.imgur.com/vTMA1b8.png)

이런 식으로 버튼을 띄워주도록 했다.

아래는 실제 구동되는 화면이다! 
열심히 상품들을 구경하다가 메인에 나오면 3초 이후에 서버와 비교를 하고, 새 게시글이 있다면 버튼을 노출시키고, 터치하면 업데이트까지 이어지도록 하는 것이다!

![](https://i.imgur.com/nTDW4dq.gif)


### 2. Animation 

- layoutConstraint 값은 animation 블록 이전에 바꿔주고, 애니메이션 블록에서는 layoutIfNeeded를 써줘야한다.
- 기본 animate 메서드를 쓰면 콜백지옥...	
	- keyframe메서드를 쓰면 된다! 
	- 내부에서 나열 가능
	- relativeStartTime을 지정해줘서 애니메이션이 실행될 시간을 표기해준다.
- CGAffineTransform은 concatenate해서 이어 붙일 수 있다. 붙이는 순서에 따라 애니메이션 실행 순서가 된다.
- transform에 `.identity`의 값을 주면 다시 원래대로 돌아온다.

## 고민된 점 
- 데이터가 업데이트 되었는지 알려줄 수 있는 방법은??

## 해결 방법 
- 인스타그램의 "새 게시물" 버튼의 로직을 상상해보고 구현해봤다.
---

**Ref**

야곰의 animation 강의
