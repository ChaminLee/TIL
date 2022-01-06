
#  Multipart/form-data 작성

## 220106_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. Multipart/form-data 작성

서버에 데이터를 보내기 위해 양식을 확인하던 도중 Content-Type이 Multipart/form-data임을 알게 되었다. 

무엇인지 찾아보니 온갖 문자열들이 기괴하게 적혀있는 것 볼 수 있었다... 

그래도 하나하나 뜯어보니 이해는 되었는데, 어떻게 작성하는 것이고 무엇을 의미하는 것인지 살펴보자. 

아래 이미지를 예시로 들어보자.

![Http Request does not work when uploading files using multipart/form-data –  IDEs Support (IntelliJ Platform) | JetBrains](https://intellij-support.jetbrains.com/hc/user_images/j9IYoDVggnLQPDDX4TR71g.png)

임하기 전에, 포인트는 저 양식을 swift로 똑같이 만들어주면 된다고 생각해보면 쉬울 것 같다. 

우선 3번 라인을 보면 `POST` 방식으로 서버에 요청하고 있음을 알 수 있다. 또한 요청을 보내는 url도 함께 확인해볼 수 있다. 

자 이제 4번 라인부터 HTTPHeader에 해당하는 내용이 된다. 

이를 HTTPHeader에 등록해주는 방법은 간단하다. 

```swift
let url = URL(string: http://localhost:8858/file/test)!
let boundary = UUID().uuidString
var request = URLRequest(url: url)
self.addValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
```

UUID는 중복되지 않는 총 36개의 문자(32개의 문자, 4개의 하이푼)로 구성되어 있다

이제 header까지 만들었으니 body를 작성해보자. 줄바꿈에 유의해야 한다!

```swift
extension Data {
	mutating func appendString(_ string: String) {
		guard let data = string.data(using: .utf8) else {
			return
		}
		
		self.append(data)
	}	
}


var body = Data()
body.appendString("--\(boundary)\r\n")
body.appendString("Content-Disposition: form-data; name=\"\(file) filename=\"\(fileName)\"\r\n\r\n")
// imageData는 실제 이미지 데이터라고 해보자.
body.append(imageData)
body.appendString("\r\n")
body.appendString("--\(boundary)--")
```
`\r\n`은 줄바꿈을 의미한다. 

이렇게 저 양식을 따라서 한 줄 한 줄 작성하다보면 body를 완성한 것을 볼 수 있다. 

이제 이 body를 request의 httpBody에 담아서 요청을 보내면 된다!

```swift
request.httpBody = body
```

이렇게 URLRequest 인스턴스의 header와 body를 채워보았다. 이제 이를 `dataTask()`에 보내서 처리해주면 된다!

## 고민된 점 
- Multipart/form-data란? 작성법은?

## 해결 방법 
- 삽질...그리고 삽질...
- 예시들을 학습하고 직접 한 줄 한 줄 작성하기
---

**Ref**

https://medium.com/@jang.wangsu/ios-swift-uuid%EB%8A%94-%EC%96%B4%EB%96%A4-%EC%9B%90%EB%A6%AC%EB%A1%9C-%EB%A7%8C%EB%93%A4%EC%96%B4%EC%A7%80%EB%8A%94-%EA%B2%83%EC%9D%BC%EA%B9%8C-22ec9ff4e792
