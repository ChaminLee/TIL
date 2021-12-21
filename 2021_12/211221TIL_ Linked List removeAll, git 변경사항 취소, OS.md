

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

그래서 아래와 같은 코드로 개선해보았다. 
> (확실하진 않지만 한 번 생각해봤다...!)

```swift
func removeAll() {
	while let next = head?.next {
		head = nil
		head = next
	}
	
	head = nil
	tail = nil
}
```

추가된 부분은 while문으로 다음 노드 여부를 확인하며 head를 계속 이동을 하게 되는데, 이동하기 전에 head를 nil로 만들어서 모든 노드를 순회하며 연결을 끊어주는 역할을 한다. 

![](https://i.imgur.com/6FbHQYs.png)

3개의 노드로 구성되어있는 Linked list를 생각해보자. 

만약에 처음 방법처럼 head/tail에만 nil을 주게 되면 node2가 메모리에 남게 된다. 

![](https://i.imgur.com/gkcmtzb.png)

하지만 새로 구현한 방법을 사용하는 경우 head에서 출발해서 노드 하나하나의 메모리를 해제해주기 때문에 큰 문제가 없어보인다. 

순서를 하나하나 봐보자.

1. `head = nil`

![](https://i.imgur.com/cB5E8BI.png)

현재 head를 nil로 만들어준다. 

2. `head = next`

![](https://i.imgur.com/yM0b0X4.png)

head가 다음으로 이동하게 된다. 그리고 1번을 다시 반복한다. 

3. `head = nil`, `tail = nil`

![](https://i.imgur.com/7u58faj.png)

앞의 while문을 다 통과하고 나면 head와 tail이 같아질 것이다. 

그렇게 되면 이제 head와 tail에 모두 nil을 넣어 마지막 노드를 제거하게 되고 모든 노드를 삭제할 수 있게 되는 것 같다. 

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

