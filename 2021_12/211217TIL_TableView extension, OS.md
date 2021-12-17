# TableView extension, OS

## 211217_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. TableView extension

테이블 뷰를 만들다보면 cell identifier를 정하고... 기억하고... 적고... 이런 반복되는 행위들을 경험해 본 적이 있을 것이다. 

보통은 namespace를 만들거나, factory pattern을 구현하여 생성하는데 있어 조금 더 편리하게 구현할 것이다. 

위 방법 뿐만 아니라 이를 조금 더 간결하게 만들기 위해서 `UITableView`를 extension하여 구현해 볼 수 있다.

원리는 간단하다 

1. `UITableViewCell` 클래스 이름을 cell identifier로 사용한다. 
2. extension하여 generics의 className을 cell identifier가 들어가야 할 위치에 넣어준다. 
3. 해당 메서드를 사용하면 이제 cell identifier를 기억하고 있지 않아도 된다.


우선 1번의 내용을 코드로 구현해보자. 

```swift
extension UITableViewCell {
    static var className: String {
        return String(describing: self)
    }
}
```

이렇게 `UITableViewCell`을 extension하여 본인의 클래스 명을 cell identifier로 쓸 수 있도록 하는 준비를 마쳤다. 

이제 2번을 구현해보자. 

```swift
extension UITableView {
    public func register<T: UITableViewCell>(cell: T.Type) {
        register(T.self, forCellReuseIdentifier: T.className)
    }

    public func dequeueReusableCell<T: UITableViewCell>(for type: T.Type, for indexPath: IndexPath) -> T {
        guard let cell = dequeueReusableCell(withIdentifier: T.className, for: indexPath) as? T else {
            fatalError("Failed to dequeue cell.")
        }
        return cell
    }
}
```

우리가 알고 있는 기존의 `register()`, `dequeue()`와 비슷하게 생긴 것을 볼 수 있다. 좀 다른 점은 generics를 사용했다는 점이 가장 클 것이다. 이제 이 메서드들을 사용하게 되면 `T.className` 덕분에 cell identifier에 대한 신경을 조금 덜 수 있다.

사용은 원래 하던 것 처럼 하면 된다!

```swift
// register
override func viewDidLoad() {
	tableView.register(cell: CustomTableViewCell.self)
}

// dequeue
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(for: CustomTableViewCell.self, for: indexPath)
        
    // configure cell
        
    return cell
}

```

### OS 강의 수강!

[4. System Structure & Program Execution 2](https://github.com/ChaminLee/CS_Study/blob/main/OS/4.%20System%20Structure%20%26%20Program%20Execution%202.md)

1주에 2강씩 듣고 있는데, 아직까지는 초반부라 그런지 부담없이 듣고 있는 것 같다.

## 고민된 점 
- TableView의 cell identifier를 어떻게 하면 최대한 신경쓰지 않고 코딩할 수 있을까?

## 해결 방법 
- Name Space
- Factory pattern
- TableView extesion

---

**Ref**

https://github.com/yagom-academy/ios-exposition-universelle/pull/130#discussion_r770466578

https://github.com/SwifterSwift/SwifterSwift/blob/master/Sources/SwifterSwift/UIKit/UITableViewExtensions.swift


