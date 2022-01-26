# 몫/나머지 구하기, UILayoutPriority, OS(deadlock)

## 220126_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. `quotientAndRemainder(dividingBy: )`

```swift
func quotientAndRemainder(dividingBy rhs: Int) -> (quotient: Int, remainder: Int)
```

주어진 값으로 나눈 몫과 나머지를 반환한다. 

```swift
let number = 5
let (quotient, remainder) = number.quotientAndRemainder(dividingBy: 2)
print(quotient) // 2
print(remainder) // 1
```

코테에서 잘 쓸 수 있을 것 같다!

### 2. UILayoutPriority

기본적으로 hugging과 compression 우선순위 값을 많이 주는데 보통 아래와 같이 기존에 나뉘어진 구간으로 값을 준다. 

- `.defaultLow` : 250
- `.defaultHigh` : 750
- `.required` : 1000

수평선상에 UILabel을 두 개 놓고, 한 쪽이 길어지게 될 때 다른 한쪽이 눌리지 않게 하기 위해 compression을 주거나, 다른 하나를 늘어나게 해주기 위해 hugging의 값을 주거나 하게된다. 

이 때 유의할 점은 hugging의 기본값은 250(`.defaultLow`)이고, compression은 750(`.defaultHigh`)라는 것이다. 

즉 무조건 `.defaultHigh`를 줬다고 해서 적용이 되는 것이 아니고 상대되는 다른 UI요소의 우선순위 값을 확인해서 더 높게 주거나, 더 낮게 줬는지 잘 확인해야한다. 

그래서인지 가장 높은 `.required`를 써주는게 모든 상황에서 더 높음을 보장해 줄 수 있을 것 같다. 

### 3. OS

Deadlock에 관한 강의를 수강했다. deadlock이 발생하는 조건, 이를 해결하는 다양한 방법들에 대해 학습해봤다! 

https://github.com/ChaminLee/CS_Study/blob/main/OS/16.%20Deadlocks%201.md

https://github.com/ChaminLee/CS_Study/blob/main/OS/17.%20Deadlocks%202.md

## 고민된 점 
- UILayoutPriority가 적용되지 않을 때..?
- OS: deadlock의 발생 배경과 해결 방법

## 해결 방법 
- 올라프의 A/S와 UILayoutPriority에 대한 실험을 통한 학습
- OS강의 수강!
---

**Ref**

https://developer.apple.com/documentation/swift/int/2884924-quotientandremainder

https://github.com/ChaminLee/CS_Study/blob/main/OS/16.%20Deadlocks%201.md

https://github.com/ChaminLee/CS_Study/blob/main/OS/17.%20Deadlocks%202.md
