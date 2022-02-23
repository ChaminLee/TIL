# SwiftUI 맛보기

## 220222_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### SwiftUI 


선언형 프로그래밍으로 Swift보다 코드에 있어서 조금 더 직관적인 것 같다. 

프로젝트를 기본적으로 생성하면 보이는 `ContentView_Previews`는 우측 미리보기에 연동해줄 뷰를 선택해주는 것으로 크게 신경쓰지 않아도 된다! 

```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```


- `View`는 struct가 view처럼 행동할 것이라는 것을 알려준다. 즉 화면을 나타내주는 역할을 하는 것이다
- `var body: some View`는 body가 뷰처럼 작동할 것이라는 것을 말해준다. 

즉, 레고처럼 뷰를 쌓을 수 있는 것이다.

- `.padding()`은 View 타입에 존재한다. 말 그대로 ui요소 기준 옵션에 따른 방향에 padding을 추가해준다.
    - `.padding()`은 View를 리턴시켜준다! 텍스트에 붙인다고 수정된 텍스트가 나오는 것이 아니라 수정된 view가 나온다! 
- `RoundedRectangle(cornerRadius: 25)`을 사용하면 네모난 뷰를 쉽게 만들 수 있다!
- `.stroke(lineWidth: )`를 하면 테두리만 채워진다! 

우측 inspector 하단의 `Add Modifier`를 누르면 여러 옵션들을 inspector에 추가해줄 수 있다! Swift에서 하던 `@IBInspectable`의 역할과 비슷한 것 같다.  

<img src="https://i.imgur.com/UxeJ84r.png" width=50%>

이제 직접적으로 뷰에 접근해서 그려보자. 
우선 body에 요소들을 선언해줄 수 있는데, `ZStack`
을 사용하면 Z축으로 View들을 쌓을 수 있게 된다. 

```swift
var body: some View {
    ZStack(content: {
        RoundedRectangle(cornerRadius: 25)
            .stroke(lineWidth: 3)
            .padding(.horizontal)
            .foregroundColor(.red)
        Text("Hi")
    })
}
```

이렇게 Z축을 기준으로 View를 쌓아서 보여줄 수 있다.


아래처럼 하면 한 번에 ZStack내의 속성들을 바꿔줄 수 있다

```swift
var body: some View {
    ZStack(content: {
        RoundedRectangle(cornerRadius: 25)
            .stroke(lineWidth: 3)
        Text("Hi")
    })
    .padding(.horizontal)
    .foregroundColor(.red)
}
```

alignment나 content를 생략해서 작성해볼 수도 있다. 

기본 alignment는 `.center`이다!
`.top`으로 주게 되면 요소들이 상단으로 이동한다.

```swift
var body: some View {
    ZStack {
        RoundedRectangle(cornerRadius: 25)
            .stroke(lineWidth: 3)
        Text("Hi")
    }
    .padding(.horizontal)
    .foregroundColor(.red)
}
```

<img src="https://i.imgur.com/WOPNf7i.png" width=50%>

뷰를 상당히 쉽게 그릴 수 있는 것 같다! 

이제 여러 장의 카드를 만들어보자. 

```swift
var body: some View {
    HStack {
        ZStack {
            RoundedRectangle(cornerRadius: 25)
                .stroke(lineWidth: 3)
            Text("Hi")
        }
        ZStack {
            RoundedRectangle(cornerRadius: 25)
                .stroke(lineWidth: 3)
            Text("Hi")
        }
        ZStack {
            RoundedRectangle(cornerRadius: 25)
                .stroke(lineWidth: 3)
            Text("Hi")
        }
        ZStack {
            RoundedRectangle(cornerRadius: 25)
                .stroke(lineWidth: 3)
            Text("Hi")
        }
    }
    .padding(.horizontal)
    .foregroundColor(.red)
}
```

4장의 카드를 만들려면 이렇게 일일이 추가해줘야할까...?
10개면 과연 다 써줄 수 있을까...

다행히도 이렇게 작성할 일은 없다. 

커스텀한 뷰(`CardView`)를 만들어주자.

```swift=
struct CardView: View {
    var body: some View {
        ZStack {
            RoundedRectangle(cornerRadius: 25)
                .stroke(lineWidth: 3)
            Text("Hi")
        }
    }
}
```

이렇게 속성을 정해줬다면 기존의 `ZStack`을 대체해 줄 수 있다. 

```swift
struct ContentView: View {
    var body: some View {
        HStack {
            CardView()
            CardView()
            CardView()
            CardView()
        }
        .padding(.horizontal)
        .foregroundColor(.red)
    }
}
```

다크모드로도 쉽게 바꿔줄 수 있다. 우측 inspector로도 가능하다.  

```swift=
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            ContentView()
                .preferredColorScheme(.dark)
        }
    }
}
```

라이트 모드/ 다크 모드 화면을 한 번에 볼 수도 있다.

```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            ContentView()
                .preferredColorScheme(.dark)
            ContentView()
                .preferredColorScheme(.light)
        }
    }
}
```

조건을 줘서 뷰를 다르게 할 수 있다. 

```swift
var isFaceUp: Bool
    
var body: some View {
    ZStack {
        if isFaceUp {
            RoundedRectangle(cornerRadius: 25)
                .fill()
                .foregroundColor(.white)
            RoundedRectangle(cornerRadius: 25)
                .stroke(lineWidth: 3)
            Text("🔥")
                .font(.largeTitle)
        } else {
            RoundedRectangle(cornerRadius: 25)
                .fill()
        }
    }
}
```

카드의 앞면/뒷면을 이렇게 나타내 볼 수 있다. 

```swift
let shape = RoundedRectangle(cornerRadius: 25)
```

이렇게 변수로 담아서 대체해줄 수 있다.

- ` .onTapGesture { }`: 뷰의 터치 액션을 정의해줄 수 있다. 

뷰의 프로퍼티들은 struct내에 있기에 immutable하다. 값이 바뀌면 뷰 전체가 재구성되어야 한다. body를 재구성 하는 것!  

```swift
@State var isFaceUp: Bool = true
```

`@State`를 붙이면 mutable해진다. 값이 바뀔 수 있다는 것이다! 처음에 쓰기는 좋으나 `@State`는 별로 잘 안쓰인다고 한다!

SwiftUI에서는 for문 대신 `ForEach()`를 사용한다. 
ForEach를 통해 순회할 배열을 넣어주면 된다. 

```swift
ForEach(emojis, id: \.self) { emoji in
    CardView(content: emoji)
}
```

다만 ForEach를 사용할 때 String이 Identifiable을 채택하기를 요구한다. 왜냐하면 같은 String이 왔을 때 구분할 수 없기 때문이다. 

`id: \.self`를 넣어 각 요소들을 고유하게 식별하도록 해준다. 

- `Button(action: () -> Void, label: () -> _`를 통해 버튼을 만들 수 있다.
    - action에는 말 그대로 버튼을 터치했을 때의 액션
    - label은 버튼의 텍스트를 의미한다

`Spacer()`를 두면 양 UI 요소 사이에 공백을 넣을 수 있다.

```swift
HStack {
    Button {
        emojiCount += 1
    } label: {
        VStack {
            Text("Add Card")
        }
    }
    Spacer()
    Button {
        emojiCount -= 1
    } label: {
        VStack {
            Text("Remove Card")
        }
    }
}
```

<img src="https://i.imgur.com/JepDjDa.png" width=50%>


이런식으로 버튼 사이 간격이 멀어진다

코드가 길어지니 변수로 빼두면 다음과 같이 사용할 수 있다.

```swift
var add: some View {
    Button {
        if emojiCount < emojis.count {
            emojiCount += 1
        }
    } label: {
        VStack {
            Image(systemName: "plus.circle")
        }
    }
}

var remove: some View {
    Button {
        if emojiCount > 1 {
            emojiCount -= 1
        }
    } label: {
        VStack {
            Image(systemName: "minus.circle")
        }
    }
}

HStack {
    add
    Spacer()
    remove
}
```

이미지의 경우도 `.font()`로 조정할 수 있다 

이제 grid 모양으로 뷰를 구성해보자. 

- `LazyVGrid`
    - 자식 뷰들을 그리드에 배열하는 컨테이너 뷰이다.
    - Lazy가 붙었으니, items를 미리 생성하지 않고, 필요할 때가 되면 생성한다.
    ```swift
    LazyVGrid(columns: [GridItem(),GridItem(),GridItem()]) {
        ForEach(emojis[0..<emojiCount], id: \.self) { emoji in
            CardView(content: emoji).aspectRatio(2/3, contentMode: .fit)
        }
    }
    ```
- `ScrollView { }` : 스크롤뷰를 간단하게 만들 수 있다
- `.aspectRatio(2/3, contentMode: .fit)`
    - 2/3 크기로 fit하게 크기를 조정할 수 있다.
- `GridItem(.adaptive(minimum: 80))` 
    - adaptive를 사용하면 flexible하게 아이템들을 넣을 수 있다.
    - 최소 사이즈를 지정해주면 된다.


대략적인 최종 결과는 다음과 같다!

<img src="https://i.imgur.com/YmQyqyN.gif" width=50%>


## 고민된 점 
- SwiftUI 사용법

## 해결 방법 

---

**Ref**

https://www.youtube.com/watch?v=bqu6BquVi2M

https://www.youtube.com/watch?v=3lahkdHEhW8
