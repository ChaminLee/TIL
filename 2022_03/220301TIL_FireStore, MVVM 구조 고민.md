# FireStore, MVVM 구조 고민

## 220301_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### FireStore 초기 세팅 

FireStore에 들어가면 collection/document 계층을 볼 수 있다. 

아직 데이터를 추가하지 않았으니 데이터를 추가하는 방법을 먼저 보자. 다음 코드는 공식 문서의 snippet이다 

```swift
var db = Firestore.firestore()

db.collection("cities").document("LA").setData([
    "name": "Los Angeles",
    "state": "CA",
    "country": "USA"
]) { err in
    if let err = err {
        print("Error writing document: \(err)")
    } else {
        print("Document successfully written!")
    }
}
```

먼저 cities라는 document 즉 폴더를 만들어준다. 그리고 그 하위에 LA라는 문서를 만들어주는데, 내용은 [name, state, country]를 갖고 생성된다. 

아무 세팅하지 않고 위 코드를 실행했다면 데이터가 추가되지 않을 것이다. 

왜일까?

FireStore의 규칙은 초기에 읽기/쓰기가 false이기 때문이다. 이에 true로 변경해줘야 한다. 

![](https://i.imgur.com/xLQwVsX.png)

그리고 다시 데이터를 추가하는 코드를 작성하면 정상적으로 colleciton/document가 추가되었음을 알 수 있다. 

### MVVM 구조 고민 

MVVM을 구현하기 위한 고민을 많이 했던 것 같다.

![](https://i.imgur.com/srbYGnc.png)

처음에는 추상적으로 이런 그림을 그려보기도 했다가 조금 더 구체화해봤다. 

![](https://i.imgur.com/KU6CL88.png)

비즈니스 로직들은 최대한 ToDoManager가 가지도록 해주고 ViewModel은 징검다리 느낌을 살려주도록 했다. 

또한 추후에 local/remote DB가 오더라도 쉽게 갈아끼울 수 있게 Repository 프로토콜을 만들어 DB가 해야하는 일을 추상화하였다. 

일단은 이 정도로 고민해봤고, 바인딩을 어떻게 해야할지 고민을 더 해봐야겠다. 

## 고민된 점 
- FireStore 초기 설정
- MVVM 패턴을 구현하는 방법

## 해결 방법 
- FireStore 공식문서 보기
- MVVM 패턴 그림으로 그려보고, 각 기능들을 추상화하기
---

**Ref**

https://firebase.google.cn/docs/firestore/manage-data/add-data?hl=ko
