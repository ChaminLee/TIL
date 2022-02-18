# RxSwift 특강 

## 220218_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)

## Todo

- CS 강의 2개 수강 
- 알고리즘 풀이 
- 묵찌빠 리뷰 
- Rx 복습 

## 학습 내용

### RxSwift 특강 

!!들어가기전에 복습!!
- @escaping : 메서드가 끝나고 전달해야하는 데이터, 뒤를 부탁하오 

다른 쓰레드에서 수행하는데 escaping closure가 아니라 결과는 return으로 받으면 안되니??라는 질문에서 Rx가 출발함!

그래서 받는 변수는 나중에 생기는 값이라는 타입(Observable)을 가지게 됨 

> async-await를 붙여주면, async: 너 비동기 작업이야, await: 비동기 작업으로 받을 값이야! 

RxSwift를 사용하면 코드가 파편화 되는 문제나 클로저 지옥을 벗어날 수 있다!

- Stream: 데이터가 흘러가는 통로 (추상적인 개념)
	- 반응형 프로그래밍: 이벤트가 생기면 반응한다. 
		- 여러 데이터가 전달될 수 있다.
		- 그렇기에 stream은 계속 연결된 상태로 두어야 한다. 
		- stream이 끊어지면 데이터 전달은 불가해진다. 
		- steam을 닫으려면 `.onCompleted()` 호출
- subscribe하는 쪽이 소비자 
- Rx에서는 event가 stream을 통해 흐른다. event의 종류는 세 가지다. 
	- next: 데이터를 전달
	- error: 에러 발생, stream도 끊긴다
	- complete: stream을 끊는다
	- onNext를 사용해서 간단하게 한 가지만 받을 수도 있다. 
- 생성한 Observable을 dispose시켜서 원하는 시점에 취소시킬 수 있다. 
- DisposeBag을 생성해서, 넣어두고 다시 생성하는 방식으로 한 번에 비울 수 있다.
	- class가 메모리 해제되면서 disposebag도 날라간다!
- flatMap을 쓰면 Observable 내의 요소를 꺼낼 수 있다. 
- take를 통해 스트림을 나눠서 여러가지 소비를 할 수 있다.
- 다수의 스트림을 하나로 묶어서 소비해볼 수 도 있다. 

---
#### Subject
- 아까 소비자는 소비만 할 뿐 생산은 하지 못했다
- 생산/소비를 같이 할 수 있는 존재는 없을까? -> Subject
	- `.onNext`로 생산! 
	- `.subscribe`로 소비 
	- 데이터가 계속 들어오면 소비도 계속하게 됨! 

#### Subject의 종류 

##### 1. AsyncSubject

![](https://reactivex.io/documentation/operators/images/S.AsyncSubject.png)

완료되면 그 때 가장 마지막의 요소가 subscribe한 stream에 전달됨(completed이 오는 경우)

![](https://reactivex.io/documentation/operators/images/S.AsyncSubject.e.png)
위 경우는 error와 함께 observable이 종료되었을 때인데, 이 때는 아이템을 emit하지 않는다. 

##### 2. BehaviorSubject

![](https://reactivex.io/documentation/operators/images/S.BehaviorSubject.png)

- 초기 값을 가지고 시작한다. 
- subscribe하면 초기값을 주고 이후 생성되는 데이터 또한 전달한다
- 이후에 subscribe하면 초기 값이 아닌 가장 최근 데이터를 주고, 이후에 발생하는 데이터를 준다. 

![](https://reactivex.io/documentation/operators/images/S.BehaviorSubject.e.png)

error와 함께 종료되는 경우, 이후에 subscribe할 경우 에러만 전달해준다. 

##### 3. PublishSubject
![](https://reactivex.io/documentation/operators/images/S.PublishSubject.png)

- subscribe 이후의 데이터만 전달된다
- 이에 subscribe 시점에 따라 이전의 데이터를 받지 못할 수 있다. 
- 그래서 위의 문제를 해결하려면 `ReplaySubject`를 써야한다

![](https://reactivex.io/documentation/operators/images/S.PublishSubject.e.png)

에러가 발생하는 경우에도 그대로 이후의 이벤트인 에러만 받을 수 있다. 

##### 4. ReplaySubject

![](https://reactivex.io/documentation/operators/images/S.ReplaySubject.png)

- subscribe 하면 여태까지의 데이터를 모두 보여주고 이후의 데이터도 보여준다. 
- 즉 모든 데이터를 전달해준다. 
- 여러 쓰레드에서 `onNext` 메서드를 사용하지 않도록 주의해야한다
	- 왜냐하면 어떤 아이템이 먼저 replayed 되어야 할지 모호해지기 때문이라고 한다. 

> - Cold Observable
	- subscribe(소비)가 되어야만 데이터가 흐름
>- Hot Observable
	- subscribe(소비자)가 없는데 데이터가 계속 흐름 
	- 시간초 같은 경우 


---
#### RxCocoa
- `.bind(to: imageView.rx.image)`처럼 UI 요소와 연결 가능 
- `button.rx.tap.subscribe()` 처럼 버튼 눌렀을 때 처리도 해줄 수 있다.
- `self.rx.viewWillAppear` 같은 코드도 가능 + `take()`를 붙이면 `viewWillAppear`도 한 번만 호출 가능함

다양한 RxSwift의 extension들이 존재한다. 

데이터가 소진되었을 때 stream은 completed 되거나, error가 날 때, 소비자가 dispose할 때 끊겼었다. dispose는 괜찮은데, complete이나 error일 때 끊기면 앞으로 데이터를 받지 못하게 되는 상황이 발생한다. 

이러한 상황은 UI를 처리할 때 문제가 된다. 

UI의 stream에는 특징이 있다!
- stream이 끊어지면 안된다!
	- 화면은 항상 보여져야한다! 
- 항상 메인(UI) 쓰레드에서 수행되어야 한다. 

그래서 RxCocoa의 요소들은 completed, error에 대해서 처리하지 않는다!

- subscribe할 때는 항상 UI Thead에서 실행되어야 한다. 
	- completed, error가 오면 안된다!
- `bind(to: )`를 사용하면 completed, error가 들어와도 stream이 끊기지 않음 
- subscribe대신 drive를 사용할 수도 있다. 
	- `asDriver(onErrorJustReturn: nil)`: 에러가 오면 nil 반환
	- `drive(onNext: )`로 subscribe를 대체하고, 에러가 나는 경우 에러 전달이 아니라 nil이 오고, completed가 오면 무시하고, 데이터를 받아주고, 처리도 메인 쓰레드에서 해준다! 
	- UI에서는 drive라고 쓰면 화면을 건드리는구나! 라고 알아볼 수 있다. 
- Subject가 생산/소비도 한다.
- RxRelay: subject와 동일한데, stream이 안끊긴다! 
	- `PublishRelay()` 처럼 생성하면 stream이 안끊긴다
	- 기존 subject 4가지 종류에 Relay를 붙이면 된다!

#### 설계의 순서
- 모델 설계 
- 비즈니스 로직 (모델 처리 로직) 
	- 비즈니스 로직을 프로토콜로 감싸기 
	- 프로토콜을 통해 VC ~ 로직 간에 양방향 호출 
- 동작 검증 
- UI 붙이기 

#### 아키텍쳐
- 어떤 코드로 분류할 것인가
- 코드를 어디에 배치할 것인가

MVC, MVVM은 화면을 처리하는 패턴일 뿐이다! 

---


> ### Tip
> https://github.com/iamchiwon/whatisarchitecture 보고 개발 순서 파악
> github1s 붙이면 바로 코드 볼 수 있게 뷰가 바뀜


## 고민된 점 
- RxSwift 의 기원 및 사용방법

## 해결 방법 
- 곰튀김님 강의 듣고, 정리하고 ReactiveX 홈페이지 읽어보기
---

**Ref**

곰튀김님 RxSwift 특강

https://reactivex.io/documentation/subject.html
