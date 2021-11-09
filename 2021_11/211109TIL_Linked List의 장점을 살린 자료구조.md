# Linked List의 장점을 살린 자료구조


## 211109_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Linked List에 대하여 

알고리즘 문제를 풀 때만 linked list를 써봤지 실제 iOS 프로젝트에서는 처음 사용해봤다. 기존에는 웬만하면 array 타입으로 많이 진행했었는데, 이번에는 장단점을 파악해보고 이유있는 선택을 해봤다. 

|자료 구조|Array|LinkedList|
|:---:|:---:|:---:|
|장점|1. 인덱싱(검색)이 빠르다(`O(1)`)|1. 데이터 추가 및 삭제가 빠르다 <br> 2. 데이터 추가/삭제시 메모리 재배치가 필요없기에 오버헤드가 낮다  |
|단점|1. 메모리의 위치가 연속적이고 고정적이기에 오버헤드가 높다 <br> 2. 데이터 추가/삭제가 느리다 |1. 인덱싱(검색)이 느리다(`O(n)`) |

우선 Queue를 위해 linked list를 만드는 것이니 필요한 몇몇 메서드만 우선적으로 구현해주었다. 

```swift
// Generics로 타입 나타냄 
class LinkedList<Element> {
	// Linked List내의 Node들을 나타낼 객체 
    class ListNode<Element> {
        var value: Element
        var next: ListNode?
        
        init(value: Element) {
            self.value = value
        }
    }
    
    // 자주 쓰일 예저이니 별칭 달아주기 
    typealias Node = ListNode<Element>
    
    // 외부에서는 접근하지 못하도록 하고
    // 값이 있을 수, 없을 수 있다. 
    private var head: Node?
    
    // 첫 번째 노드를 가리킨다
    var first: Node? {
        return head
    }
    
    // 마지막 노드를 가리킨다
    var last: Node? {
    	// 우선 head가 존재하는지 확인
        guard var node = head else {
            return nil
        }
        
        // nil이 나올 때 까지 계속 다음 노드로 이동
        while let next = node.next {
            node = next
        }
        
        // 최종 노드가 가장 마지막 노드가 된다
        return node
    }
    
    
    // 링크드 리스트가 비어있는지 확인
    var isEmpty: Bool {
        return head == nil 
    }
    
    // 링크드 리스트내 노드들의 개수
    var count: Int {
	    // 우선 head가 존재하는지 확인
        guard var node = head else {
        	// 없다면 0개
            return 0
        }
        
        // head는 있기에 1부터 시작
        var count = 1
        
        
        // 다음 노드로 이동하며 카운트 1씩 증가 
        while let next = node.next {
            node = next
            count += 1
        }
        
        return count
    }
    
    // 새로운 값 추가 
    func appendNewNode(value: Element) {
    	// Node 타입으로 생성
        let newNode = Node(value: value)
        
        // 마지막 값 확인
        guard let lastNode = last else {
        	// 없다면 빈 리스트에 추가하는 것과 같음
            head = newNode
            return
        }
        
        // 있다면 그 다음 요소로 추가 
        lastNode.next = newNode
    }
    
    // 첫 번째 노드 제거하기 
    @discardableResult
    func removeFirstNode() -> Node? {
    	// 노드가 없다면 nil
        if head == nil {
            return nil
        } else {
        	// 첫 번째 노트 임시 저장 
            let firstNode = head
            // 제거
            head = head?.next
            // 반환
            return firstNode
        }
    }
    
    // 링크드 리스트 내 모든 노드 삭제    
    func removeAllNodes() {
        head = nil
    }
}
```

이렇게 간략하게 Queue 구현을 위한 링크드 리스트를 구현해 볼 수 있다. 

## 고민된 점 

- class나 struct를 만들 때 타입을 명시해야하나?
- Array vs Linked List
- TDD의 정석은 무엇인가

## 해결 방법 
 
- `Generics` 사용 
- 각 자료구조별 장단점 비교 및 요구사항에 기반한 최적의 선택!
- 테스트 코드 작성(red) -> 테스트를 통과하는 코드 작성(green) -> 코드 리팩토링(blue)
	- 이게 정석인 것 같으나 테스트 코드를 처음에 완벽하게 짜지 못하니 테스트 코드 작성(red) -> 테스트를 통과하는 코드 작성(green)과정이 여러 번 반복되는 것 같다.
	- blue단계가 프로덕션 코드의 리팩토링을 말하는 것 같으나, 테스트 코드 자체의 리팩토링도 필요해지는 순간이 오는 것 같다. 이게 TDD가 맞는지는 더 살펴봐야겠다. 혹은 테스트 코드를 처음부터 완벽하게 설계해서 프로덕션 코드를 이에 맞추는 방식으로도 한 번 테스트 해봐야겠다. 

---

**Ref**
[Linked List](https://github.com/raywenderlich/swift-algorithm-club/tree/master/Linked%20List)
[Generics](https://leechamin.tistory.com/523?category=941561)

