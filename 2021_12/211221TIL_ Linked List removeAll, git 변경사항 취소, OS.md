

# Linked List removeAll, git 변경사항 취소, OS

## 211221_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. Linked List 모든 노드 삭제

기존에 Linked List를 구현하고 모든 노드를 삭제하기 위해서 아래와 같이 코드를 구현했었다. 

```swift
func removeAll() {
	head = nil
	tail = nil
}
```

리뷰 내용 중에, head와 tail 사이에 연결된 노드들은 이제 메모리 해제하지 못하는 것이 아니냐는 코멘트가 있었다. 

생각해보니 head와 tail을 nil로 만들어줌으로써 linked list 자체는 노드와 관련이 없어지지만, 그 사이의 노드들은 연결된 상태로 메모리를 점유하고 있겠다는 생각이 들었다. 

즉 메모리 누수(memory leak)라고도 볼 수 있을 것 같다. 그렇기 때문에 생각해본 다른 방법은 N개의 모든 노드를 순회하면서 nil로 만들어주는 방법이었다. 

N개의 노드를 순회한다는 점에서 시간 복잡도가 `O(n)`이 되겠지만, 메모리 누수가 발생하는 것 보다는 시간이 조금 더 걸리는게 나을 것 같다고 판단했다. 

하지만 생각해보니... head가 사라지면 head.next의 참조가 사라지기 때문에 연쇄적으로 메모리 해제가 될 것으로 생각되어서 deinit을 찍어보았다. 

```swift
class Node {
    var value: Element
    ...
    
    deinit {
        print("\(value) 사라진다")    
    }
}
```

현재 상황을 아래와 같이 가정해보자. 

![](https://i.imgur.com/Cz9Ns2M.png)




여기서 `head = nil`을 하게 되면 아래와 같은 상황이 된다. 

![](https://i.imgur.com/b3o7lR7.png)


자연스레 참조가 없는 노드는 사라지게 되며 tail만 남게 되는 것이다. 

![](https://i.imgur.com/yhkFsRh.png)


그래서 tail은 따로 `tail = nil`을 주어 삭제하면 정상적으로 모든 노드들이 연쇄적으로 메모리 해제됨을 확인할 수 있다. 

![](https://i.imgur.com/wOXspVc.png)


### 2. git 변경사항 취소

원하지 않는 변경사항들을 commit & push하기 이전에 지워줄 수 있다. 

- git reset 
	- staging 파일이 모두 unstage 된다.
- git checkout .
	- 모든 변경사항을 취소한다
- git clean -fdx
	- 추적할 수 없는 모든 파일 제거

간혹가다가 xcSharedData가 변경되었다거나, 원하지 않는 변경사항인데 tracking하려면 add하라는 문구가 뜨게 된다. 이러한 변경사항을 취소하기 위해서는 `git clean -fdx`를 하여 추적하지 않는 변경사항들을 모두 취소해줄 수 있다. 


### 3. OS - 3주차

프로세스 강의 수강! 
[5. Process 1](https://github.com/ChaminLee/CS_Study/blob/main/OS/5.%20Process%201.md)


## 고민된 점 
- Linked List내 노드를 모두 삭제하고 싶은 경우 어떻게 구현해야 하는가?

## 해결 방법 
- 메모리 관점에서 접근하여 생각하고 구현해보기

---

**Ref**

[git 커밋되지 않았거나 저장되지 않은 모든 변경 사항 취소](https://extbrain.tistory.com/83)
