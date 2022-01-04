
# Tableview/CollectionView, SwiftLint, URLSession completion hanlder/delegate 

## 220103_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### !복습 내용!

- UITableviewController를 사용하면 확장성이 조금 떨어진다. viewcontroller로 만들어야 추후 추가적인 요소들을 기능개발할 수 있다. 

- https는 http 프로토콜에서 TLS와 SSL 프로토콜을 사용하여 세션 데이터를 암호화한다. 

- UICollectionLayoutListConfiguration을 활용하여 컬렉션 뷰에서 리스트 모양을 보여줄 수 있다. ios14부터 지원한다.

- UICollectionViewLayout이 콜렉션뷰의 레이아웃을 커스텀할 때 사용할 때 사용하는 클래스이다. 

- drag and drop : 보통은 A 테이블 뷰 -> B 테이블뷰로 이동하는 경우를 말한다. 동일한 테이블 뷰 내에서 이동은 "cell내의 이동"이라고 부르는게 적합하다고 한다. 

### 1. UITableviewController와 ViewController

#### 공통점
- 관련된 정보를 리스트 형태로 보여준다.
- section, header, footerview 등 여러 요소를 동일하게 사용한다.
- Delegate, DataSource 프로토콜을 사용한다.

#### 차이점
- 컬렉션뷰는 기본 스타일을 제공하지 않는다. (테이블뷰는 제공함)
- 테이블뷰는 가로스크롤을 지원하지 않는다.
- 테이블뷰는 row, 컬렉션뷰는 item이라고 부른다. 
- 컬렉션뷰는 레이아웃을 지정할 수 있다.
- 테이블뷰의 상위호환 느낌으로 컬렉션이 테이블 뷰에 비해 더 많은 기능을 제공해 자유롭게 커스텀하기가 좋다.

### 2. SwiftLint 라이브러리

SwiftLint란 코드 컨벤션을 지키기 위해 도움을 주는 라이브러리이다. 

[SwiftLint의 Default Rules](https://realm.github.io/SwiftLint/rule-directory.html)

위 링크를 들어가보면 SwiftLint가 제공하는 여러 컨벤션들을 볼 수 있다. 

가장 많이 들어본 force unwrapping에 대해서도 규칙이 있다. SwiftLint를 통해 이를 적용시켜주면 원하는 범주의 파일 내에서는 force unwrapping을 쓰지 못하게 된다. 왜냐! SwiftLint가 자체적으로 빌드가 되지 않게 컴파일 에러를 내기 때문이다! 정하는 수준에 따라서 컴파일 에러를 발생시키거나, 경고 메시지 등으로 그칠 수 있지만 우선 개발자에게 경고를 줄 수 있다는 것이 큰 메리트인 것 같다. 

해당 라이브러리 설치는 사실 매우 간단하다. 
우선 [SwiftLint Github Repository](https://github.com/realm/SwiftLint)에 접근한다

설치는 튜토리얼 따라, 공식문서 따라하면 쉽게 할 수 있다. 유의해야할 점은 SwiftLint를 위한 `.yml` 파일을 만들 때 프로젝트 > 앱폴더 내가 아니라 프로젝트 바로 아래에 생성해야한다. 

![](https://i.imgur.com/fMmPBj9.png)

이런 식으로 본인의 프로젝트의 바로 아래에 생성해줘야 한다. 

아무튼! 설치를 마쳤다고 보고 살펴보자.

공식 문서를 쭉 내려서 보다보면 "Configuration"이라는 챕터가 나오는데, 이를 따라서 `.yml` 파일을 구성해주면 된다. 

> 파일명은 `.swiftlint.yml`로 하여 숨김 파일로 생성한다. 

주로 사용하는 것은 4가지 정도 될 것 같다. 

- disabled_rules
    - 기본적으로 검수하는 규칙을 제외시켜준다
- opt_in_rules
    - 기본적으로 검수하지 않는 규칙을 사용한다
- included
    - lint를 적용할 path를 나타내준다.
- excluded
    - lint 적용을 제외할 path를 나타내준다. 

아래와 같이 작성해볼 수 있다. 

```yaml
disabled_rules:
 - line_length
 - trailing_whitespace

included:
 // folder/file
 
opt_in_rules:
 - force_unwrapping
 - empty_count

excluded:
 - Pods
 - "projectName"/AppDelegate.swift
 - "projectName"/SceneDelegate.swift

```

이런식으로 원하는 lint를 선택/해제할 수 있고 원하는 파일/폴더를 대상으로 lint를 적용시킬 수 있다. 

우리가 짠 코드가 아닌 라이브러리에 대해서도 lint를 적용하기 때문에 `Pods` 폴더를 lint 대상에서 제외시켜주는게 기본적인 듯 하다. 

> ### Tip. 
> cmd + shift + . 숨김 폴더 보기 

### 3. URLSession 

URL Session을 통해 data task를 생성하고 메모리에 직접적으로 데이터를 받아보자.

서버와 자그마한 상호작용을 위해서는 `URLSessionDataTask` 클래스를 사용하여 메모리에 response data를 받을 수 있다. 만약 데이터를 파일 시스템에 다운로드 받아야 한다면, `URLSessionDownloadTask`를 사용하면 된다. data task는 웹 서비스를 호출할 때 이상적인 방법이다. 

task를 생성하기 위해 URL session 인스턴스를 사용하게 된다. 만약 간단한 경우라면 `URLSession` 클래스의 `shared` 인스턴스를 사용할 수 있다. 만약 delegate의 콜백을 통해 상호작용 해야한다면, `shared` 인스턴스를 쓰기 보다는 session을 생성할 필요가 있다. 세션을 생성할 때는 `URLSessionConfiguration` 인스턴스를 사용하고, 또한 `URLSessionDelegate` 또는 하위 프로토콜 중 하나를 구현하는 클래스도 전달한다. (이 부분은 나중에 코드와 함께 보면 이해가 갈 것이다!) 세션은 여러 tasks를 생성하기 위해 재사용 될 수 있으며 이에 각각에 해당하는 유일한 configuration이 필요하고, 세션을 생성하고 프로퍼티로 저장해서 사용해야 한다. 

> 필요 이상의 세션을 생성하지 않도록 주의해야한다. 예를 들어, 유사하게 구성된 세션을 필요로 하는 경우 하나의 세션을 만들고 이를 계속 공유하여 사용하면 된다. 

세션을 보유하고 있다면 `dataTask()` 메서드를 통해 data task를 생성할 수 있다. task는 suspended 상태로 생성되기 때문에 `resume()`을 호출하여 시작시켜줘야 한다.

#### 3-1. Completion Handler를 활용하여 결과 받기 

데이터를 받기 위한 가장 간단한 방법은 data task를 생성하고 completion handler에서 사용하는 것이다. 이에 task는 서버의 response, data, 가능한 error를 completion handler 블록에 전달해준다. 아래의 그림을 보면 세션과 task간의 관계, 어떻게 completion handler에 결과가 전달되는지 알 수 있다. 

![](https://docs-assets.developer.apple.com/published/c7124fb5d7/bf4501ff-82b2-4dd4-9ec3-243ef0e70d21.png)

completion handler를 사용하여 data task를 생성할 수 있다(`dataTask()` 호출) 이제 completion handler는 3가지 일을 해야한다.

1. `error` 파라미터가 `nil`인지 확인한다. 만약 `nil`이 아니라면 전송 에러가 발생한 것으로 에러를 처리하고 종료시켜준다. 
2. `response` 파라미터를 확인하여 성공을 나타내는 상태 코드를 확인하고 MIME 타입이 원하는 값인지 확인해야한다. 만약 아니라면 서버 에러를 처리하고 종료한다. 
3. `data` 인스턴스를 필요에 따라 사용하면 된다. 


물론 문서의 예제가 제일 좋지만 직접 실험해보고 적용해본 코드로 살펴보자. 

간단하게 살펴보자면 [Zoo Animal API](https://zoo-animal-api.herokuapp.com/)를 활용하여 랜덤한 동물을 화면에 보여주도록 한 것이다. 

먼저 모델을 만들어준다. 

```swift
struct Animal: Codable {
    let name, latinName, animalType, activeTime: String
    let lengthMin, lengthMax, weightMin, weightMax: String
    let lifespan, habitat, diet, geoRange: String
    let imageLink: String
    let id: Int

    enum CodingKeys: String, CodingKey {
        case name
        case latinName = "latin_name"
        case animalType = "animal_type"
        case activeTime = "active_time"
        case lengthMin = "length_min"
        case lengthMax = "length_max"
        case weightMin = "weight_min"
        case weightMax = "weight_max"
        case lifespan, habitat, diet
        case geoRange = "geo_range"
        case imageLink = "image_link"
        case id
    }
}
```

모델을 만드는 부분까지 간단하다. 

뷰는 만들었다고 가정하고, 핵심이 되는 네트워킹 부분을 살펴보자. 우선 첫 번째로 볼 방법은 completion handler를 활용하는 것이니 이에 포커스를 맞춰보자. 

1. url, request 생성
2. shared 인스턴스에 dataTask() 호출
3. completion handler에 넘어오는 data, response, error 처리
4. 순서대로 error -> response -> data 검증
5. 데이터 디코딩 성공시 completion handler 실행!
6. 생성한 task resume 시키기

```swift
class APIService {
    func getAnimalData(completion: @escaping (Animal) -> ()) {
        guard let url = URL(string: "https://zoo-animal-api.herokuapp.com/animals/rand") else {
            return
        }
        let request = URLRequest(url: url)


        let task = URLSession.shared.dataTask(with: request) { data, response, error in
            // error 검증
            if let error = error {
                print(error.localizedDescription)
                return
            }
            
            // response status code 검증
            guard let httpResponse = response as? HTTPURLResponse,
                  (200..<300).contains(httpResponse.statusCode) else {
                      print("error")
                      return
                  }

            // data 검증 및 사용
            if let data = data {
                do {
                    let receivedData = try JSONDecoder().decode(Animal.self, from: data)
                    completion(receivedData)
                } catch {
                    print(error.localizedDescription)
                }
            }

        }
        
        task.resume()
    }
}

```

url로 넘어오는 이미지도 `UIImage`로 변환해주는 코드를 추가해보자. 

```swift
class ImageLoader {
    static func loadImage(from url: String, completion: @escaping (UIImage?) -> ()) {
        // 입력된 url 체크
        if url.isEmpty {
            completion(nil)
            return
        }
        
        // URL 타입으로 변경
        guard let imageURL = URL(string: url) else {
            return
        }
        
        
        DispatchQueue.global(qos: .background).async {
            // URL -> Data
            if let data = try? Data(contentsOf: imageURL) {
                // Data -> UIImage
                let image = UIImage(data: data)!
                
                // UI 업데이트를 위한 main 쓰레드 사용
                DispatchQueue.main.async {
                    completion(image)
                }
            } else {
                DispatchQueue.main.async {
                    completion(nil)
                }
            }
        }
    }
}
```

간단히 보자면 여기도 마찬가지로 `@escaping`를 활용하여 문자열을 `UIImage`로 변환시키고 있음을 알 수 있다. 

내부 로직을 잠깐 살펴보자. 

1. url 문자열 검사
2. string -> URL
3. URL -> Data
4. (성공시) Data -> UIImage & UI업데이트
5. (실패시) nil 

자 이제 모든 준비가 되었다!

뷰 컨트롤러에서 세팅을 마치고 버튼의 액션으로 서버에 요청해서 UI 세팅을 하는 기능을 추가해주자. 

```swift
@objc func setupData() {
    let group = DispatchGroup()

    apiHandler.getAnimalData { animal in
        DispatchQueue.main.async(group: group) {
            ImageLoader.loadImage(from: animal.imageLink) { animalImage in
                self.imageView.image = animalImage
            }
        }

        group.notify(queue: DispatchQueue.main) {
            self.nameLabel.text = "동물 이름 : \(animal.name)"
            self.lifeLabel.text = "평균 수명 : \(animal.lifespan)년"
        }
    }
}
```

화면에는 간단하게 사진과 동물의 이름, 평균 수명 그리고 다음 랜덤한 동물을 나타내기 위한 버튼만을 추가해주었다. 

이제 실행해보면 다음과 같은 결과를 얻을 수 있다. (아직 이미지 / 레이블의 동기화 순서를 잡아주지는 않았다..!)

![](https://i.imgur.com/7JBml9Y.gif)

조금 딜레이가 있지만 서버에 데이터를 요청해서 잘 받아오고 있는 모습을 볼 수 있다. 

중요한 점은 completion handler가 task가 생성된 곳이 아니라 GCD(메인큐)를 활용하였다는 것이다. 그러므로 data나 error에 대해서 UI를 업데이트 해줄 수 있다(위 예제에서는 error에는 GCD 적용X)

#### 3-2. Delegate를 활용하여 세부 정보와 결과를 전송받기

data task를 생성할 때에, task의 활동에 더 높은 수준의 접근을 위해 데이터 작업을 작성할 때 completion handler를 제공하는 것 대신에 세션에 delegate를 설정할 수 있다. 아래 그림에서 이를 나타내주고 있다. 

![](https://docs-assets.developer.apple.com/published/8b22355c7f/730c8e1b-654f-4eb9-9c63-d439a69ac5d2.png)

이 방법을 사용하면 전송이 완료되거나 오류가 발생할 때까지 `URLSessionDataDelegate`의 `urlSession(_:dataTask:didReceive:)` 메서드에 데이터 일부가 제공된다. Delegate는 전송이 진행됨에 따라 다른 종류의 이벤트도 수신가능하다. 

우선 delegate 방식으로 접근하기 위해서는 간단한 `shared` 인스턴스가 아닌, `URLSession` 인스턴스를 생성해야한다. 새로운 세션을 생성하는 것은 클래스가 세션의 delgate 역할을 가능하게 하며 이는 위 그림을 통해 나타나있다. 

클래스 구현시에 하나 혹은 그 이상의 delegate 프로토콜(`URLSessionDelegate`,`URLSessionTaskDelegate`,`URLSessionDataDelegate`,`URLSessionDownloadDelegate`)을 정의해준다. 그리고 ` init(configuration:delegate:delegateQueue:)`를 활용하여 URL session의 인스턴스를 생성한다. configuration 인스턴스는 이 이니셜라이저를 통해 커스터마이징 할 수 있다. 예를 들어서, `waitsForConnectivity`를 `true`로 두는 좋은 생각을 해볼 수 있다. 그렇게 되면 세션은 요구되는 연결이 불가능할 때 바로 실패하기 보다는, 적합한 연결을 기다리게 된다.

아무튼 이제 가장 처음해야하는 `URLSession` 생성을 먼저해주자. 

```swift
// VC
private lazy var session: URLSession = {
    let config = URLSessionConfiguration.default
    config.waitsForConnectivity = true
    return URLSession(configuration: config, delegate: self, delegateQueue: nil)
}()
```

이제 앞으로 볼 코드에서 data task를 시작시키고, 받은 data와 error를 처리하기 위해 delegate 콜백을 활용하는 방법을 볼 것이다. 

우선 메서드의 역할부터 보고 넘어가자

- `urlSession(_:dataTask:didReceive:completionHandler:)`
    - response가 성공적인 HTTP 상태 코드를 가지고 있는지 검증하고 MIME 타입이 text/html인지 text/plain인지 확인한다. 만약 두 경우 모두 아니라면 task는 취소되며, 정상인 경우는 진행된다. 
- `urlSession(_:dataTask:didReceive:) `
    - task를 통해 받은 각 `Data` 인스턴스를 `receivedData`라고 불리는 버퍼에 저장한다. 
- `urlSession(_:task:didCompleteWithError:)`
    - 먼저 전송 오류가 발생했는지 확인한다. 
    - 오류가 없다면 `receivedData` 버퍼를 원하는 모델 타입에 맞게 디코딩해준다. 

```swift
// VC
func startLoad() {
        let url = URL(string: "https://zoo-animal-api.herokuapp.com/animals/rand")!
        receivedData = Data()
        let task = session.dataTask(with: url)
        task.resume()
    }

func setupDataByDelegate(from data: Animal) {        
    ImageLoader.loadImage(from: data.imageLink) { image in
        self.imageView.image = image
    }

    DispatchQueue.main.async {
        self.nameLabel.text = "동물 이름 : \(data.name)"
        self.lifeLabel.text = "평균 수명 : \(data.lifespan)년"
    }
}

extension ViewController: URLSessionDataDelegate {
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive response: URLResponse, completionHandler: @escaping (URLSession.ResponseDisposition) -> Void) {
        guard let response = response as? HTTPURLResponse,
              (200..<300).contains(response.statusCode) else {
                  completionHandler(.cancel)
                  return
              }
        
        // allow the load operation to continue
        completionHandler(.allow)
    }
    
    func urlSession(_ session: URLSession, dataTask: URLSessionDataTask, didReceive data: Data) {
        self.receivedData?.append(data)
    }
    
    func urlSession(_ session: URLSession, task: URLSessionTask, didCompleteWithError error: Error?) {
        if let error = error  {
            print(error.localizedDescription)
        } else if let receivedData = self.receivedData {
            do {
                let data = try JSONDecoder().decode(Animal.self, from: receivedData)
                setupDataByDelegate(from: data)
            } catch {
                print(error.localizedDescription)
            }
        }
    }
}
```

![](https://i.imgur.com/ObnAkDX.gif)

이 방법도 마찬가지로 동일한 결과를 보여준다. 

이외에도 다양한 delegate 프로토콜은 인증 문제, 리다이렉션, 기타 특수한 경우를 처리하기 위해 위에 나타나 있는 코드 이상의 방법을 제공한다. URLSession 문서를 더 살펴보면 다양한 콜백에 대해서 알아볼 수 있다. 

### 읽어봐야 할 문서 

https://developer.apple.com/documentation/foundation/operation

https://developer.apple.com/documentation/uikit/uicollectionviewdatasourceprefetching/prefetching_collection_view_data

## 고민된 점 
- `URLSession`이란
- `URLSession`을 통해 데이터를 받아오는 경우
    - 일반적인 completion handler
    - delegate method 

## 해결 방법 
- 공식 문서 확인 및 예제 코드 구현 

---

**Ref**

[Fetching Website Data into Memory
](https://developer.apple.com/documentation/foundation/url_loading_system/fetching_website_data_into_memory)

https://zoo-animal-api.herokuapp.com/

https://developer.apple.com/documentation/foundation/urlsession

https://developer.apple.com/documentation/foundation/urlsessionconfiguration#1660412

https://developer.apple.com/documentation/foundation/url_loading_system

https://developer.apple.com/documentation/foundation/urlsessiontask

https://developer.apple.com/documentation/foundation/urlsession/1411554-datatask



