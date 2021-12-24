# DispatchQueue, Race Condition

## 211224_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Race Condition과 해결 방법

Race Condition이란, 말 그대로 경쟁이라고 해석해도 좋을 것 같다. 어떤 종류의 경쟁이 될 수 있을까?

주로 `Thread-safe`하지 않은 타입들에 대해서 발생할 수 있을 것 같다. `Thread-safe`하다는 것은 쓰레드에서 사용해도 안전하다는 말로, 여러 thread가 동시에 접근해도 정상적으로 작동하다는 것을 의미한다. 

반대로 `Thread-safe`하지 않다는 것은? 어떠한 변수나 객체에 대해 여러 쓰레드에서 접근하게 되었을 때 문제가 발생할 수 있다는 것이다. 이런 경우에 발생할 수 있는 문제 중 하나로 race condition을 뽑는 것이다. 

정의 내용을 살펴보자.

> Race Condition : 두 개 이상의 Concurrent한 프로세스(쓰레드)들이 하나의 자원에 접근하기 위해 경쟁하는 상태 

예를 들어서 계좌(=자원)에 10만원이 들어있다고 해보자. 사실 오늘은 관리비 8만원이 자동 이체 되는 날인데, 점심에 배달 음식을 시키느라 5만원을 소비했다고 해보자. 배민에서 딱 결제를 하는 순간에 마침! 관리비도 동시에 이체가 되어서 관리비 이체와 배달 음식 결제 모두 성공한 것을 두 눈으로 확인했다...!

위와 같은 경우가 생기게 되면 잔액을 넘어서는 경우에 대해서도 결제가 정상적으로 되어버릴 것이다. 이러한 상황은 절대로 일어나서는 안되는데 왜 발생한걸까?? 

바로 계좌가 `thread-unsafe`  했기 때문이다. 이 때문에 race condition이 발생한 것이다. 

사실 정상적인 과정이었다면 배달비 결제가 먼저 되면서 잔액이 5만원으로 바뀌고, 그 다음 관리비 송금을 할 때 잔액을 확인하고 부족하기 때문에 이체에 실패해야 한다. 

즉 `계좌`는 동시에 한 군데서만 접근해서 작업을 해야하는 것이다. 이렇게 공유 데이터를 접근하는 코드 영역을 `Critical Section(CS)`라고 한다. 

우선 이러한 상황이 발생하는 경우는 앞서 말한 것 처럼 `concurrent`한 상황에서 동시에 접근하기 때문이다. 

앞선 예제를 코드로 나타내보자.

```swift
class Account {
    var money: Int = 0
    
    init(money: Int) {
        self.money = money
    }
}

struct Delevery {
    let tag: String = "배달비"
    func withdraw(account: Account, amount: Int) {
        guard account.money >= amount else {
            print("잔액 부족!!!")
            return
        }
        
        Thread.sleep(forTimeInterval: 1)
        account.money -= amount
        print("\(tag): \(amount)원 인출 성공! / 잔액: \(account.money)")
    }
}

struct Admin {
    let tag: String = "관리비"
    func withdraw(account: Account, amount: Int) {
        guard account.money >= amount else {
            print("잔액 부족!!!")
            return
        }
        
        Thread.sleep(forTimeInterval: 1)
        account.money -= amount
        print("\(tag): \(amount)원 인출 성공! / 잔액: \(account.money)")
    }
}

let 내계좌 = Account(money: 10)
let 배민 = Delevery()
let 건물주 = Admin()

DispatchQueue.global().async {
    배민.withdraw(account: 내계좌, amount: 5)
}

DispatchQueue.global().async {
    건물주.withdraw(account: 내계좌, amount: 8)
}
```

`건물주`와 `배민`이 동시에 `내계좌`에 접근하여 돈을 빼내가는 과정이다... 슬픈 예시...

아무튼 이렇게 concurrent하게 작업을 하면, 다음과 같은 상황을 마주하게 된다. 

```swift
관리비: 8원 인출 성공! / 잔액: 2
배달비: 5원 인출 성공! / 잔액: -3
```

오잉..? 분명 잔액이 없으면 인출하지 못하도록 했는데 이게 무슨 일인가....  이렇게 동시에 접근해서 돈을 빼가면 금방 신용불량자가 되어버릴 것이다.  

이를 방지하기 위해서는 `배민`과 `건물주`가 동시에 내 계좌에 접근하지 못하도록 막아줘야 한다. 즉, 한 번에 하나의 쓰레드만 자원에 접근하고 작업을 하고, 그 이후에서야 다른 쓰레드가 작업을 하도록 해줘야 한다는 것이다. 

이러한 문제를 해결할 수 있는 방법으로는 대표적으로 두 가지가 있다. 하나하나 살펴보자. 

> 1 . Serial Queue 사용

바로 Serial Queue를 이용해보는 것이다. serial queue의 특징이 뭐였는가?? 바로 하나의 큐만을 사용한다는 것이었다. 

즉 동기로 들어오건, 비동기로 들어오건 들어온 순서대로 `하나씩` 처리해준다는 특징이 있다. 

지금 상황에 너무나 적합한 queue이다. 이제 한 번에 하나만 작업할 수 있는 serial queue를 만들어, 해당 queue내 에서 결제/이체가 이루어지도록 만들 수 있다. 

```swift
let serialQueue = DispatchQueue(label: "안전한 계좌")

DispatchQueue.global().async {
    serialQueue.async {
        배민.withdraw(account: 내계좌, amount: 5)
    }
}

DispatchQueue.global().async {
    serialQueue.async {
        건물주.withdraw(account: 내계좌, amount: 8)
    }
}
```

이렇게 제어를 하게 되면, 배달비만 나가거나 관리비만 나가게 된다.

```swift
// 배달비가 나가는 경우
배달비: 5원 인출 성공! / 잔액: 5
잔액 부족!!!

// 관리비가 나가는 경우
관리비: 8원 인출 성공! / 잔액: 2
잔액 부족!!!
```

이제는 잔액이 마이너스가 될 걱정을 하지 않아도 될 듯 하다.
이제 안심하자..

> 2 . DispatchSemaphore 사용

또 다른 해결 방법으로는 semaphore를 활용하는 것이다. 아마 OS를 공부해봤다면 ME(Mutual Exclusion)에 대해 들어봤을 것이다. 

이름 그대로 `상호 배제`라는 것은 두 개 이상의 프로세스(쓰레드)가  CS(임계)영역에 진입하는 것을 막아주는 역할을 한다. 이러한 ME의 방법 중 하나로 semaphore가 있는 것이다. 

semaphore의 값을 통해 접근할 수 있는 쓰레드의 수를 제어해 줄 수 있다. 

코드로는 다음과 같이 구현해 볼 수 있다. 

```swift
let semaphore = DispatchSemaphore(value: 1)

DispatchQueue.global().async {
    semaphore.wait() // semaphore -= 1
    배민.withdraw(account: 내계좌, amount: 5)
    semaphore.signal() // semaphore += 1
}

DispatchQueue.global().async {
    semaphore.wait() // semaphore -= 1
    건물주.withdraw(account: 내계좌, amount: 8)
    semaphore.signal() // semaphore += 1
}
```

마찬가지로 동시에 인출되는 경우는 방지되고, 하나의 경우만 발생할 수 있다. 

```swift
// 배달비가 나가는 경우
배달비: 5원 인출 성공! / 잔액: 5
잔액 부족!!!

// 관리비가 나가는 경우
관리비: 8원 인출 성공! / 잔액: 2
잔액 부족!!!
```

이제 계좌를 안전하게 보호할 수 있게 되었다. 

## 고민된 점 
- concurrent한 환경에서 발생하는 race condition 문제 해결방법?

## 해결 방법 
- OS에서 쓰레드를 제어하거나, 직렬 큐를 이용하여 해결 

---

**Ref**

[Mutual Exclusion](https://leechamin.tistory.com/533?category=1012929)

[Mutual Exclusion - Semaphore](https://leechamin.tistory.com/546?category=1012929)

[야곰닷넷 - concurrency programming](https://yagom.net/courses/%eb%8f%99%ec%8b%9c%ec%84%b1-%ed%94%84%eb%a1%9c%ea%b7%b8%eb%9e%98%eb%b0%8d-concurrency-programming/)



