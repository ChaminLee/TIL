# RxSwift 맛보기

## 220217_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. RxSwift 

비동기 작업을 하는데 추가적인 비동기 작업해야하는 경우 콜백 지옥에 빠지기 쉽다..

- 그렇다면 비동기로 생성된 데이터를 어떻게 리턴해줄까??

비동기로 작업하기 때문에 반환값 또한 나중에 생기게 된다. 이러한 나중에 생기는 데이터들을 묶어서 `@escaping` closure로 전달하는 것이 아니라 반환 하도록 해주는 유틸리티들이 생기게 되었다! 

(왜냐 completion을 계속하면 콜백 지옥에 빠지니, 간단하게 반환값을 내보내주는 방식이 더 좋아보이기 때문!)

- PromiseKit (then)
- Bolt (then)
- RxSwift
	- subscribe: 나중에 오면! event를 받아서 할 일
		- next: 받아서 작업 수행 
		- completed
			- subscribe의 결과는 disposable임! 
	- Observable: 나중에 생기는 데이터! (감싸서 전달하면 된다)
	- onNext: 데이터 전달 

### !!!RxSwift는 비동기로 생기는 데이터를 completion같은 클로저가 아니라 리턴할 수 있도록 해주는 것!!!

취소를 위해 Disposables가 있다.

- disposable.dispose : 
	- 버린다! 
	- 작업을 시켜둔 게 끝나지 않아도 취소시킬 수 있다. 

complete 나 error에서 클로저가 종료된다. 이 때 클로저가 사라지면서 ref count가 줄어들게 하려면 
onCompleted()를 넣어주면 된다! 

#### 1. 비동기로 생기는 데이터를 Observable로 감싸서 리턴하는 방법
- Observable.create

```swift 
func createObservable() -> Observable<String?> {
    return Observable.create() { emitter in
        // data를 전달하는 역할
        emitter.onNext("전달할 데이터")
        // 전달 끝
        emitter.onCompleted()

        return Disposables.create()
    }
}
```

네트워킹을 접목하면 아래와 같이 작성해볼 수 있다. 

```swift
func downloadData(with url: String) -> Observable<String?> {
    return Observable.create() { emitter in
        let url = URL(string: url)!

        let task = URLSession.shared.dataTask(with: url) { data, _, error in
            guard error == nil {
	            // 에러 리턴
                emitter.onError(error)
            }

            if let data = data, let json = String(data: data, encoding: .utf8) {
	            // 데이터가 준비되면 전달
                emitter.onNext(json)
            }
			// 데이터가 준비되지 못했다면 그냥 종료
			emitter.onCompleted()
        }

        task.resume()
        // 작업 cancel시에 task도 cancel 시켜야 함
        return Disposables.create() {
            task.cancel()
        }
    }
}
```
- Observable의 생명 주기
	1. create
	2. subscribe -- 이 때 동작 
	3. onNext 
	4. onCompleted / onError
	5. disposed

이미 3번 이후에는 끝난 것과 같다. 

동작이 끝난 Observable은 재사용 불가하다. 다른 곳에서 또 subscribe 처리를 해줘야 한다. 

- `.debug()` : observable의 생명주기를 보여줌

#### 2. Observable로 오는 데이터를 받아서 처리하는 방법 
- Observable.subscribe

subscribe로 데이터를 받는다.

- event를 받는데 종류가 세 가지있다
	- next, error, completed

```swift
let disp = downloadData(with: "www.naver.com")
                .subscribe { event in
                    switch event {
                    case .next(let data):
                        // 데이터 처리
                    case .error(let error):
                        // error 처리
                    case .completed:
                        // 완료
                    }
                }
disp.dispose()
```

subscribe의 결과로 disposable이 나온다. 이를 가지고 작업을 취소할 수 도 있다. 

지금까지의 기본 사용법에서 귀찮은 것들을 제거해 볼 수 있다. 바로 `operator`를 활용해보면 된다.

https://reactivex.io/documentation/operators.html 
여기서 operator들을 확인해 볼 수 있다.  

위에서 create 했던 것을 간결하게 해보려면 `just`를 사용하면 된다. 
```swift
return Observable.just("Hello")
// Hello
```

just에서 배열을 내보내면 배열 1개 자체가 한 번에 내보내진다.

```swift
return Observable.just(["Hello","Hi"])
// Hello, Hi
```

반대로 `from`을 사용하면 배열 내의 데이터가 하나하나 보내진다.  
```swift
return Observable.from(["Hello","Hi"])
// Hello
// Hi
```

subscribe의 경우도 next만 받아서 간결하게 처리해 줄 수 있다. 나머지 error, complete, dispose의 경우도 해당하는 클로저에 원하는 코드를 적어주면 된다.
```swift
let disp = downloadData(with: "www.naver.com")
            .subscribe(onNext: { data in
                // data 처리
            }, onError: { error in
                // error 처리
            }, onCompleted: {
                // 완료
            }, onDisposed: {
                // dispose
            }
```
next, error, complete, dispose 4가지 중 필요한 걸 선택하여 구현하면 된다. 

또 메인 쓰레드에서 작동시키기 위해 `observeOn()` 을 사용해볼 수 있다. `observeOn()` 은 다음 라인부터 영향을 준다

```swift
.observeOn(MainScheduler.instance)
```

스케줄러를 선택하여 메인, 글로벌 쓰레드를 고를 수 있다. 

`subscribeOn()`은 첫 번째 라인에 영향을 미친다. 즉 어디에 적어도 항상 첫 라인에 적용이 되기에 위치가 상관이 없다.
```swift
.subscribeOn(ConcurrentDispatchQueueScheduler(qos: .default))
```

Observable을 합쳐주는 operator는 다음 세 가지가 대표적이다. 


- `Merge`
	- 여러개 Observable을 하나의 obsevable로 합쳐준다. 
	- 단순하게 여러 개의 데이터를 순서대로 합쳐주는 것! 
	- 대신 데이터 타입이 같아야 한다. 
- `Zip`
	- 각 observable에서 데이터를 하나씩 쌍으로 만들어서 보내준다.
	- 항상 2개의 데이터가 있어야 쌍으로 생성할 수 있다. 
- `CombineLastest`
	- 쌍을 만들게 없으면 가장 최근에 나온 값과 쌍으로 만들어준다. 


disposeBag을 만들어서 insert해줄 수 있다! 
`.disposed(by: )`를 통해 Observable에 이어서 disposeBag에 넣어줄 수 있다. 

그래서 나중에 disposeBag을 한 번에 비우거나 처리해준다. 

---

- Subject (PublishSubject)
	- observable처럼 subscribe로 값을 받을 수 있고 값을 외부에서 통제할 수도 있다.

한 번만 subscribe해두면 이후 값의 변경이 생기면 매번 onNext가 호출되어 ui업데이트가 자동적으로 된다는 것!

#### RxCocoa

- RxCocoa는 RxSwift의 요소를 UIKit 뷰들에 extension해서 접목 시킨 것!

이런 것들이 제공된다. 

- `UI요소.rx.text`
	- bind(to: )
	- 바인드를 하면 데이터와 UI를 연결해줄 수 있다. 
	- subscribe를 안쓰기에 weak self를 안써도 순환참조 문제 해결이 된다. 
	- 즉 bind를 통해 UI 요소에 데이터를 전달해주는 것이다. (데이터에 변경이 있을 경우)

뷰는 아무런 역할을 하지 않고 화면만 그리고!! 뷰 모델에서 데이터에 대한 처리를 다 해주는 것이다! 대부분의 로직이 뷰 모델에서 이루어진다!

그래서 뷰 모델을 가지고 test case를 만들기가 수월해진다. 그러면 view controller는 알아서 잘 돌아갈 것이다! 

- MVP
	- ViewController가 View쪽으로 가고 기존 로직을 presenter가 가지도록 함 
	- View ~ Presenter는 1:1 관계 
	- 모든 판단과 로직은 Presenter가 가지도록!
	- presenter, model은 testable하다!
	- view는 항상 presenter에게 물어봐야 하고, presenter는 항상 view에게 시켜야 하는 단점...
- MVVM
	- ViewModel은 화면에 뭘 그리라고 지시하지 않음
	- View -> ViewModel을 보며 바라본다. **ViewModel내 요소의 값이 바뀌면 스스로 뷰를 바꾸는 것**이 중요한 개념이다
	- 비슷한 뷰가 많고 데이터가 같다면 뷰는 여러 개, 뷰모델은 1개로 구현 가능하다. 같은 데이터를 기반으로 보여줄 수 있는 것 
	- ViewModel, Model은 testable하다!


정리해보자!

- Observable
	- 런타임에 동적으로 변경이 어렵다
	- 이미 데이터가 정해져있는 stream이다
	- create할 때 부터 뭘 반환할 지 정해져있다
	- 그래서 변경이 필요할 때는 Subject를 사용! 
- Subject
	- 데이터를 외부에서 넣고, 받을 수 있다. 
	- subscribe도 가능!
- RxCocoa
	- UI작업에서 사용됨, UIKit + RxSwift
	- UI는 메인쓰레드에서 작업되야하기에 observeOn 써줘야 함!
	- bind를 사용하면 UI와 연결할 수 있고, 순환 참조 문제를 해결할 수 있다. 
	- 어떤 에러가 나더라도 stream이 끊기면 안된다. 
		- `asDriver` 에 기본 값을 넣어두면 `.drive`에 `rx.text`를 연결해줄 수 있다. 
		- driver는 에러가 나는 경우에 대해 기본 값으로 처리하고, driver는 항상 메인 쓰레드에서 돌아간다! 말 그대로 UI 처리용!
- RxRelay
	- 끊어지지 않는 subject를 생성하기 위함 
	- BehaviorRelay 이런 식으로 생성 가능, 에러가 나도 끊어지지 않고 무시하게 된다. 
	- 오로지 next만 처리되고 error, complete는 발생하지 않는다. 그래서 next가 아닌 accept를 사용한다. 

Relay와 Driver는 모두 UI의 아래 특징을 고려해서 생긴 것이다.

- 에러가 나더라도 스트림이 끊어지면 안된다
- 메인 쓰레드에서 수행되어야 한다
	
## 고민된 점 
- RxSwift의 사용 목적과 사용 방법

## 해결 방법 
-갓곰튀김님 강의와 예제를 통한 이해

---

**Ref**

https://github.com/iamchiwon/RxSwift_In_4_Hours
