
# flatMap과 compactMap의 차이점

## 211116_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### `flatMap`과 `compactMap`의 차이점

원래 `flatMap`은 배열을 펼쳐주는 역할, `compactMap`은 옵셔널 바인딩을 하여 형변환을 해주는 것으로 알고 있었다. 

하지만 `flatMap` 또한 nil을 제거해주는 역할을 하고 있었다. (왜지..)

찾아보니 flatMap은 원래 3가지 용도로 만들어졌던 것을 확인해볼 수 있었다. 

> 1. 배열을 펼쳐주기

`flatMap`을 사용하는 가장 큰 이유가 될 것이다!
```swift
let arr = [[1],[2],[3,4,5]]

print(arr.flatMap { $0 }) // [1, 2, 3, 4, 5]

print(arr.compactMap { $0 }) // [[1], [2], [3, 4, 5]]
```

이렇게 차원(?)을 줄여서 원하는대로 입맛에 맞게 활용해볼 수 있는 기능이 있다. 이름처럼 flat하게 만들어주는 그런 메서드이다. 

`map`, `compactMap`과는 다르게 `flatMap`는 `.map(transform).joined()`로 구현되어있기에 펼쳐줄 수 있는 것이다. 

> 2. 문자열을 쪼개기 

```swift
let words = ["work","find","rest"]

print(words.flatMap { $0 }) // ["w", "o", "r", "k", "f", "i", "n", "d", "r", "e", "s", "t"]
print(words.compactMap { $0 }) // ["work","find","rest"]
```

이건 몰랐던 기능인데, 문자열을 이렇게 쪼개줄 수도 있다. 

> 3. 옵셔널 처리해주기 
```swift
let arr = ["apple","2","3","of","5"]

print(arr.map { Int($0) }) // [nil, Optional(2), Optional(3), nil, Optional(5)]

// 'flatMap' is deprecated: Please use compactMap(_:) for the case where closure returns an optional value
print(arr.flatMap { Int($0) }) // [2, 3, 5]

print(arr.compactMap { Int($0) }) // [2, 3, 5]
```

이건 `compactMap`만 가능한 건줄 알았는데 아니었다. 하지만 Swift에서 권장하는 메서드는 `compactMap`이고 메시지와 같이 `flatMap` 사용을 지양하라고 하고 있다. (그래서 다른 상황에서 `flatMap` 쓸 때는 경고 메시지가 안나오고, nil이 포함될 수 있는 경우에만 메시지를 보낸 듯 하다.)

3가지 기능이나 하다니... 대단하면서 왜 3개나 기능하도록 만들었는지 의문이 들었다.

각 3가지의 내부 구현 코드를 살펴보자.

```swift
// 배열을 펼치기
@inlinable
  public func flatMap<SegmentOfResult: Sequence>(
    _ transform: (Element) throws -> SegmentOfResult
  ) rethrows -> [SegmentOfResult.Element] {
    var result: [SegmentOfResult.Element] = []
    for element in self {
      result.append(contentsOf: try transform(element))
    }
    return result
  }

// 쪼개주는 역할
@inlinable // lazy-performance
  public func flatMap<SegmentOfResult>(
    _ transform: @escaping (Elements.Element) -> SegmentOfResult
  ) -> LazySequence<
    FlattenSequence<LazyMapSequence<Elements, SegmentOfResult>>> {
    return self.map(transform).joined()
  }

// 옵셔널 벗겨주는 역할 
 @inlinable
  public func flatMap<U>(
    _ transform: (Wrapped) throws -> U?
  ) rethrows -> U? {
    switch self {
    case .some(let y):
      return try transform(y)
    case .none:
      return .none
    }
  }
```

좀 정리를 해보자면 아래와 같은 기준으로 메서드들을 선택해 볼 수 있을 것 같다.

- transform의 결과가 optional일 수 있다면 
	- `compactMap` 사용
- 그게 아니라면 나머지 경우 상황에 맞게
	- `map`, `flatMap` 사용

## 고민된 점 
- `flatMap`도 옵셔널 처리를 해줄 수 있는가?

## 해결 방법 
- 공식 문서 및 구현 코드 확인 
---

**Ref**
[swift LazySequenceProtocol - flatMap, compactMap](https://github.com/apple/swift/blob/main/stdlib/public/core/FlatMap.swift)
[swift optional - flatMap](https://github.com/apple/swift/blob/main/stdlib/public/core/Optional.swift)
[swift Sequence - flatMap](https://github.com/apple/swift/blob/main/stdlib/public/core/SequenceAlgorithms.swift)
[flatMap vs compactMap](https://stackoverflow.com/questions/49291057/difference-between-flatmap-and-compactmap-in-swift)
