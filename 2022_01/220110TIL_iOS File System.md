# iOS File System

## 220110_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### iOS File System

- Sandbox 
	- 사용할 수 있는 영역을 지정해주고, 해당 영역 이외에서는 파일을 읽고/쓰는 등 자원에 접근할 수 없게 할 수 있는 개념이다. 

> 다른 앱에 권한을 주는 것 = sandbox에 들어오게 하는 것.

sandbox 구조가 아닌 경우에는 모든 사용자 데이터, 모든 시스템 자원에 제한없는 접근이 가능하다. 샌드박스 구조일 경우에는 앱 사용자 데이터, 앱 시스템에 제한없는 접근은 가능하지만 그 외의 사용자 데이터, 시스템 자원에는 접근이 불가하다. 

- 라이브러리 영역
	- application support
		- 앱 마다의 영역을 관리한다. 
		- 애플리케이션 생성 데이터 저장 
		- 관리용 데이터 저장 
	- cache
		- 반복 사용 임시 데이터 저장 
	- preference
		- 애플리케이션 설정 데이터 저장 
		- 직접 수정 권장 x
		- userdefaults, cfpreference 사용 
- tmp 
	- 재사용하지 않는 임시 데이터 저장 

![](https://dl.dropbox.com/s/jk555owekupnw3w/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202017-07-15%20%EC%98%A4%ED%9B%84%209.08.01.png)

application support에서 말하는 보이지 않는 개념은 유저에게 파일로서 오픈되지 않는 것을 말한다.

> 임시 저장 데이터의 경우, auto-saved 폴더를 application support에 만들어서 관리하다가 사용자가 저장 버튼을 누르면 document내 saved로 만들어주는 등의 방법으로 관리할 수 있다. tmp는 맘대로 삭제될 가능성이 있기에 고민을 잘 해야한다. 

이제 각 폴더에 파일을 작성/읽기/삭제하는 방법을 알아보자.

> 1.   Application Support 

```swift
let fileManager = FileManager.default

let appSupportURL = fileManager.urls(for: .applicationSupportDirectory, in: .systemDomainMask).first!

let directoryURL = tmp.appendingPathComponent("abc.txt")
print(directoryURL)
// 쓰기
do {
    let text = "Hello, Files!"
    try text.write(to: directoryURL, atomically: false, encoding: .utf8)
} catch {
    print(error)
}

// 읽기
do {
    let readText = try String(contentsOf: directoryURL, encoding: .utf8)
    print(readText)
} catch {
    print(error)
}

// 삭제
do {
    try fileManager.removeItem(at: directoryURL)
} catch {
    print(error)
}
```

> 2. Document

```swift
let fileManager = FileManager.default

let appSupportURL = fileManager.urls(for: .documentDirectory, in: .systemDomainMask).first!

let directoryURL = appSupportURL.appendingPathComponent("abc.txt")
print(directoryURL)
// 쓰기
do {
    let text = "Hello, Files!"
    try text.write(to: directoryURL, atomically: false, encoding: .utf8)
} catch {
    print(error)
}

// 읽기
do {
    let readText = try String(contentsOf: directoryURL, encoding: .utf8)
    print(readText)
} catch {
    print(error)
}

// 삭제
do {
    try fileManager.removeItem(at: directoryURL)
} catch {
    print(error)
}
```

> 3. tmp

```swift
let fileManager = FileManager.default

let document = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
let tmpURL = try! FileManager.default.url(for: .itemReplacementDirectory, in: .userDomainMask, appropriateFor: document, create: true)

let directoryURL = tmpURL.appendingPathComponent("abc.txt")
print(directoryURL)
// 쓰기
do {
    let text = "Hello, Files!"
    try text.write(to: directoryURL, atomically: false, encoding: .utf8)
} catch {
    print(error)
}

// 읽기
do {
    let readText = try String(contentsOf: directoryURL, encoding: .utf8)
    print(readText)
} catch {
    print(error)
}

// 삭제
do {
    try fileManager.removeItem(at: directoryURL)
} catch {
    print(error)
}
```

다시 한 번 저장소의 특징들을 살펴보자.

- application support
    - 앱의 기능/관리를 위해 지속적으로 관리해야하는 파일을 저장
    - 유저에게 노출이 되지 않는다.
    - coreData의 기본 저장 경로
    - itunes, icloud에 백업
- document
    - 유저가 앱 내에서 생성한 문서/데이터, 음악 등을 저장 
    - 앱마다 document를 하나씩 가진다.
    - itunes, icloud에 백업
- tmp
    - 임시 저장이 필요한 파일만 저장 
    - 시스템이 주기적으로 파일을 삭제, 앱이 실행될 때는 삭제하지 않음
    - 백업이 안됨!

## 고민된 점 
- 로컬에서 파일은 어떻게 관리하는가?

## 해결 방법 
- application support, document, tmp의 특성 비교 및 입력/출력/삭제 기능 구현

## Todo!!!
- UICollectionView Modern cell Configuration 구현 방식 블로그 작성하기!
---

**Ref**

https://jinnify.tistory.com/26

https://developer.apple.com/documentation/foundation/filemanager/searchpathdomainmask/1408037-userdomainmask

https://developer.apple.com/documentation/foundation/filemanager

https://stackoverflow.com/questions/43743423/what-is-the-appropriatefor-parameter-for-in-filemanager-urlforinappropriatefo

https://0urtrees.tistory.com/189

https://hcn1519.github.io/articles/2017-07/swift_file_manager
