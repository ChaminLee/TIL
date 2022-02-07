
# View Drawing Cycle, UISplitViewController

## 220207_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. View Drawing Cycle

![](https://user-images.githubusercontent.com/45457678/84453568-1f171d80-ac93-11ea-8e61-9ab28225f2ed.png)

- requiresConstraintBasedLayout
    - constraint-based layout을 사용할지 말지를 나타내는 Bool 타입의 타입 프로퍼티이다.
    - True면 constraint-based layout을 사용한다는 것이고, False이면 다른 것을 사용한다는 의미이다.

### Constraints
- updateConstraints()
    - 뷰의 제약 사항을 업데이트하는 역할을 한다.
- setNeedsUpdateConstraints()
    - 뷰의 제약 사항을 업데이트 하기 위한 updateConstraints를 예약한다. 바로 반영되지 않고, 다음 드로잉 사이클에 반영된다. 
- updateConstraintsIfNeeded()
    - 뷰의 제약 사항을 즉시 업데이트 한다. 
- intrinsicContentSize
    - 뷰가 가지는 Content의 크기. layout system과 동적으로 소통하기 때문에 Frame과는 독립적인 값이다. 
    
### Layout
- viewWillLayoutSubviews()
    - viewController에게 view의 하위뷰가 layout 될 것(layoutSubviews)을 알려준다.
- layoutSubviews()
    - 실제로 view의 layout이 적용되는 메서드
- viewDidLayoutSubviews()
    - viewController에게 view의 하위뷰가 layout 되었음(layoutSubviews)을 알려준다.
- setNeedsLayout()
   - layout이 다음 drawing cycle에 업데이트 되도록 예약하는 메서드이다.
- layoutIfNeeded()
   - layout이 즉시 업데이트 되도록하는 메서드이다.

### Draw 

- draw()
    - 직접적으로 호출하면 안된다!
    - 주어진 rectangle에 실제로 그려주는 역할을 한다.
    - draw를 override 한지 안한지 모르니까 draw는 무조건 계속 호출되고 있는 것이다.
- setNeedsDisplay()
    - 다음 drawing cycle에 draw하도록 예약하는 메서드이다.
- setNeedsDisplay(_ r: CGRect)
    - 업데이트가 필요한 일부분을 나타낸다. 

---

- displayIfNeeded() (view의 메서드가 아니라 Core Animation의 메서드임!)
    - CALayer의 메서드로 View의 drawing과는 직접적으로 관련이 있지는 않다!

Layout은 배치하는 것 자체, draw는 뷰 내의 요소를 그리는 것이다. 이에 layout이 바뀌게 되면 내부 요소들도 다시 그려줘야 하기에 draw도 호출된다.


짧게 정리해보자. 

|메서드의 목적|Constraints|Layout|Display
|:---:|:---:|:---:|:---:|
|실제로 업데이트 실행(재정의할 뿐 직접 호출하면 안된다!)|updateConstraints|layoutSubviews|draw|
|다음 업데이트 주기에 업데이트가 필요한 것으로 명시적으로 표시|setNeedsUpdateConstraints|setNeedsLayout|setNeedsDisplay|
|업데이트가 필요하면 즉시 업데이트|updateConstraintsIfNeeded|layoutIfNeeded|-|

즉 핵심은 Constraints/Layout/Display를 예약/실행하는 각각의 메서드임을 인지하고 필요에 따라 호출해서 업데이트를 관리하면 될 것 같다. 

### 2. UISplitViewController

![](https://i.imgur.com/JpBk4bf.png)

아이패드를 사용해본 사람이라면 위와 같은 화면을 본 적이 있을 것이다. 

![](https://i.imgur.com/bUlvxS0.png)

이처럼 Sidebar가 하나 이상 보이는 경우도 있는데, 왼쪽부터 Primary. Supplimentary, Secondary view라고 부른다. 

이러한 스타일은 `UISplitViewController(style: )`을 통해 지정할 수 있다. 

![](https://i.imgur.com/5quYLM7.png)

- `.doubleColumn`
- `.tripleColumn`
- `.unspecified`

```swift
let splitVC = UISplitViewController(style: .doubleColumn)
```

이를 간단하게 구현해보면서 필요한 메서드들에 대해 일부 알아보자! 
순서부터 보자. 

1. Primary VC를 생성한다.
    - TableView를 구성한다
2. Second VC를 생성한다.
3. Main VC에 Split VC를 생성한다
    - SplitVC에 primary/secondary VC를 세팅한다. 

#### 1. Primary VC를 생성한다.

SplitVC에서 primary에 해당하는 화면을 구성하기 위한 viewController이다. 

간략하게 tableView를 구성해두기만 하면 된다!

```swift
class PrimaryViewController: UITableViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        self.navigationItem.title = "첫번째"
        self.navigationItem.rightBarButtonItem = UIBarButtonItem(title: "추가", image: nil, primaryAction: nil, menu: nil)
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
    }
    
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 20
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        
        var content = cell.defaultContentConfiguration()
        content.text = "\(indexPath)"
        
        cell.contentConfiguration = content
        return cell
    }
    
    override func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        
    }
}
```

#### 2. Second VC를 생성한다.

이제 secondary에 해당하는 화면을 구성해보자. 우선 primary의 등장 유무에 따른 화면 변화를 눈으로 보기 위해 화면 정중앙에 UILabel을 하나 배치해보자. 

```swift
class SecondViewController: UIViewController {
    let label: UILabel = {
        let lb = UILabel()
        lb.text = "가운데 글씨가 있어요!"
        return lb
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .lightGray
        self.view.addSubview(label)
        label.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
}
```

#### 3. Main VC에 Split VC를 생성한다

우선 `UISplitViewController` 객체를 생성한다. 

```swift
class MainViewController: UIViewController {
    let splitVC = UISplitViewController(style: .doubleColumn)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        config()
    }

    func config() {
        let nav = UINavigationController(rootViewController: PrimaryViewController())
        splitVC.setViewController(PrimaryViewController(), for: .primary)
        splitVC.setViewController(SecondViewController(), for: .secondary)
        splitVC.preferredDisplayMode = .oneBesideSecondary
        
        self.view.addSubview(splitVC.view)
    }
}
```
splitVC의 style은 `.doubleColumn`으로 해준다. 

그리고 이제 좌측에 뜨게 될 primary view의 테이블에 navigation bar의 타이틀, 버튼을 세팅하기 위해 `UINavigationController`로 감싸준다. 

그리고 `setViewController(vc: ,for: )`을 통해 어떤 Column에 어떤 VC를 넣어줄지 정해준다. 

그리고 `.preferredDisplayMode`를 지정해준다. 

종류는 여러 개가 있는데, 공식문서의 그림을 보면 이해가 좀 더 수월할 것이다. 

![](https://i.imgur.com/AJzfJya.png)

Beside가 붙은 화면들을 보면 말 그대로 화면 옆에 붙어 있다는 것을 알 수 있다. 

Displace가 붙어있는 mode의 경우 이름 그대로 화면이 대체된다. 즉 화면이 밀리게 되는 것과 유사하다. 

그리고 Over가 붙어있는 mode도 말 그대로! 기존 Secondary 화면 위에 덮어지는 스타일이다. 

이러한 6가지 스타일을 잘 선택해서 사용하면 된다! 

이렇게 간략하게 구성한 화면을 살펴보자. 

![](https://i.imgur.com/3XcXquR.gif)

좌측 primary가 등장하고 사라짐에 따라 secondary view의 text가 움직이는 것을 볼 수 있다! 



## 고민된 점 
- UISplitViewController는 어떻게 구현하는가?
- View의 drawing cycle은 어떻게 구성되어있는가?

## 해결 방법 
- 공식문서 개념과 코드를 통한 기본 예제 구현
- 공식문서, WWDC 학습을 통한 사이클 이해 

---

**Ref**

https://developer.apple.com/documentation/uikit/uisplitviewcontroller

https://developer.apple.com/design/human-interface-guidelines/ios/user-interaction/drag-and-drop/
