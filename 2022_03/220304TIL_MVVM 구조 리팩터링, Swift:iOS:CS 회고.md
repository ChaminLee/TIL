# MVVM 구조 리팩터링, Swift/iOS/CS 회고

## 220304_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 피드백 반영한 점!

- 프로토콜 네이밍의 모호함
	- ViewModel, Domain, Repository를 추상화 한 프로토콜의 이름에도 서비스의 요소가 담겨 있으면 좋을 것 같다는 의견이 있었다.
	- 기존에는 레이어의 이름만 띡 있었다면, 개선 이후에는 `Task-` 가 붙거나 뷰 모델의 경우 어느 뷰에 데이터를 전달할 것인지가 중요하기에 `TaskList-`라는 이름을 붙여주었다. 
- ViewModel내 메서드의 네이밍
	- ViewModel은 view에 데이터를 전달하는 역할을 하며 view는 Viewmodel을 호출해서 사용한다.
	- 이에 결국에는 view에서 호출될 메서드들이기 때문에 view의 관점에서 네이밍해주는 것이 좋다고 한다!
	- cell이 눌렸을 때 해당 데이터를 반환하기 위해 `fetch`라는 이름보다 뷰의 관점인 `didSelectRow`같은 네이밍이 적합하다는 것이다!
- MVVM+CleanArchitecture
	- 데이터를 불러오거나, 필터링하는 경우에는 무조건 데이터의 원천인 Repository에서 관리할 수 있도록 해주는게 좋다고 한다.
	- 외부의 배열에 저장해두고 사용하는 경우에는 동기화되지 않은 시점에서 사용될 경우가 있기 때문에 데이터를 반환, 다루는 객체는 repository를 직접적으로 이용하는 객체여야 한다! 

### Swift/iOS/CS 회고

캠프의 끝이 보이기에, 여러 지식들을 다시 정리해봐야겠다는 생각이 들었다.

동시에 좋은 기회를 얻어 면접을 앞두고 있는 상황에서 iOS, Swift, CS에 대한 지식을 점검해야 할 필요가 있다고 생각했다. 

그래서 현재 정리중에 있다..!

## 고민된 점 
- MVVM의 전체적인 흐름과 구조

## 해결 방법 
- 계속 살펴보고... 어색한 점이 있나 보기. 
- 리뷰어 또치의 피드백을 참고하여 개선해보기!

---

**Ref**

