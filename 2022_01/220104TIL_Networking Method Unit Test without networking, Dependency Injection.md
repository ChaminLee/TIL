
# Networking Method Unit Test without networking, Dependency Injection

## 22O104_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### Networking Method Test 

네트워킹 메서드 또한 하나의 기능이기 때문에 단위 테스트를 거쳐야 할 필요가 있다. 하지만 네트워킹 코드의 경우 대부분 비동기적으로 작동한다는 점, 여러 외부적인 요인들에 영향을 받는다는 부분에서 테스트가 어렵다. 

예시로 인터넷 연결, 서버, 여러 다른 시스템 문제, 성능, 로딩, 디코딩등 다양한 이유가 존재한다. 

이 때문에 protocol을 통해 MockURLSession을 만들어서 테스트를 진행하게 된다고 한다. 

즉, **실제로 네트워킹 환경이 아니라** 네트워킹이 되지 않는 환경에서 임의의 값을 가지고 테스트하게 되는 것이다. 

시작하기에 앞서 필요한 mock 객체는 다음과 같다. 

- MockData
    - 임의의 값을 가지고 있는 데이터
- MockURLSession
    - URLSession을 대신할 가짜 버전
    - 이 덕분에 네트워크 환경이 아닌 상황에서 테스트가 가능해진다. 
- MockURLSessionDataTask
    - URLSeesionDataTask를 대신할 가짜 버전

또한 테스트를 위한 전체적인 순서를 짚고 넘어가자 

---

1. URLSessionProtocol 생성 및 URLSession에 채택 (= 의존성 생성)
2. 기존 APIService 타입의 의존성을 protocol에 의존하도록 변경하며 init 구현
3. MockData 준비 
4. MockURLSessionDataTask 생성
    - URLSessionDataTask 상속
    - APIService에서 `resume()` 될 때 호출해줄 클로저 작성 
5. MockURLSession 생성
    - URLSessionProtocol 채택
    - 성공/실패 여부 표기
    - sessionDataTask 타입을 갖도록 함
    - `dataTask()` 구현
        - 성공/실패 response 커스텀하여 세팅
        - 성공/실패 매칭하여 completionHandler 호출
6. Unit Test 작성

---

바로 시작해보자!

#### 1. URLSessionProtocol 생성 및 URLSession에 채택

```swift
protocol URLSessionProtocol {
    func dataTask(with request: URLRequest, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask
}

extension URLSession: URLSessionProtocol {}
```
기본적으로 `URLSession`이 갖고 있는 `dataTask`를 프로토콜의 요구사항에 넣어 구현해준다. 이를 통해 URLSession을 일부 대변할 수 있는 `의존성`이 생긴다고 봐도 될 것 같다. 


#### 2. 기존 APIService 타입의 의존성을 protocol에 의존하도록 변경하며 init 구현


우선 네트워킹을 위한 기본적인 메서드들은 구현이 되어있다고 생각해보자. 이 때 기존에 URLSession에 직접적으로 의존하던 것을 프로토콜에 의존하도록 대상을 변경해야한다.

```swift
class APIService {
    var session: URLSessionProtocol
    
    init(session: URLSessionProtocol = URLSession.shared) {
        self.session = session
    }
    
    func getAnimalData(completion: @escaping (Result<Animal, Error>) -> ()) {
    ...
```


#### 3. MockData 준비 

임의의 mock data를 준비해준다. 

```swift
enum AnimalData {
    static var data: Data {
        Data("""
                {
                    "name": "Siamang",
                    "latin_name": "Hylobates syndactylus",
                    "animal_type": "Mammal",
                    "active_time": "Diurnal",
                    "length_min": "1.90",
                    "length_max": "2.00",
                    "weight_min": "20",
                    "weight_max": "23",
                    "lifespan": "23",
                    "habitat": "Tropical rainforest",
                    "diet": "Primarily fruit and leaves, some invertebrates",
                    "geo_range": "Malaysia and Sumatra",
                    "image_link": "https://upload.wikimedia.org/wikipedia/commons/a/a4/DPPP_5348.jpg",
                    "id": 162
                }
            """.utf8
        )
    }
}
```

[Zoo Random API](https://zoo-animal-api.herokuapp.com/)를 통해 얻은 임의의 데이터이다. 이 중에 하나를 임의로 저장해두고 마치 네트워킹으로 얻은 데이터처럼 생각하고 이를 비교하여 테스트하게 된다. 

#### 4. MockURLSessionDataTask 생성

이제 URLSessionDataTask를 대신해줄 MockURLSessionDataTask를 생성해줘야 한다. MockURLSessionDataTask 또한 URLSessionDataTask 타입이기 때문에 상속을 먼저 해준다. 

이에 URLSessionDataTask의 `resume()` 메서드를 사용할 수 있는데, 재정의하여 MockURLSessionDataTask의 클로저를 넣어주자. 이후에 `getAnimalData()` 메서드 내에서 `resume()`이 호출될 때 넣어준 클로저도 실행되게 된다. 

```swift
class MockURLSessionDataTask: URLSessionDataTask {
    var mockResume: () -> () = {}
    
    override func resume() {
        mockResume()
    }
}

```

#### 5. MockURLSession 생성

이제 실제 URLSesssion을 대신할 MockURLSession 객체를 만들어주게 된다. 

우선 해당 task가 실패할지 성공할지를 임의로 먼저 정해준다. 그 이후 `dataTask()` 메서드를 구현해준다. 이전에 APIService 타입에서 구현해줄 때는 실제로 네트워킹이 동작하도록 해줬다. 

하지만 지금은 테스트를 위한 객체이고, 네트워킹이 필요하지 않기 때문에 임의의 데이터를 넣어 대신해준다. 이 때문에 우리가 정의한 `dataTask()` 메서드는 기존 URLSession의 `dataTask()`와는 달리 비동기적으로 작동하지 않음을 알 수 있다. 

```swift
class MockURLSession: URLSessionProtocol {
    var isSuccess: Bool
    
    init(isSuccess: Bool = true) {
        self.isSuccess = isSuccess
    }
    
    var sessionDataTask: URLSessionDataTask?
    
    // 가짜 dataTask
    func dataTask(with request: URLRequest, completionHandler: @escaping (Data?, URLResponse?, Error?) -> Void) -> URLSessionDataTask {
        
        // 가짜 성공 response
        let successResponse = HTTPURLResponse(url: request.url!, statusCode: 200, httpVersion: "2", headerFields: nil)
        
        // 가짜 실패 response
        let failureResponse = HTTPURLResponse(url: request.url!, statusCode: 400, httpVersion: "2", headerFields: nil)
        
        // 가짜 task
        let sessionDataTask = MockURLSessionDataTask()
        
        // resume시 불릴 가짜 completion handler
        sessionDataTask.mockResume = {
            if self.isSuccess {
                completionHandler(AnimalData.data, successResponse, nil)
            } else {
                completionHandler(nil, failureResponse, nil)
            }
        }
        
        self.sessionDataTask = sessionDataTask
        return sessionDataTask
    }
}

```

아무튼 위 코드를 보면 이제 짜맞추기(?)식으로 성공/실패시 데이터를 설정했으니 아까 만들었던 MockURLSessionDataTask의 클로저를 정의해주면 된다. `dataTask()` 메서드가 APIService에서 호출되고 `resume()` 되었을 때 해당 클로저 또한 실행되어야 하기 때문에 이 시점에서 정의해준다. 

데이터는 앞서 만들었던 mock data를 넣고 response에는 임의로 만들었던 성공/실패 response를 넣어준다. 

#### 6. Unit Test 작성

이제 모든 준비는 끝이 났고 unit test 코드만 작성해주면 된다. 

테스트 대상 객체인 APIService 타입을 프로퍼티에 지정해두고 초기화하면서 MockURLSession을 넣어주면 된다. 

```swift
class networkingTestTests: XCTestCase {
    var sut: APIService!
    
    override func setUpWithError() throws {
        sut = APIService(session: MockURLSession())
    }
    
    func test_Animal_데이터_불러오기_성공() {
        // 비교 대상 데이터(mockData)
        let response = try? JSONDecoder().decode(Animal.self, from: AnimalData.data)
        
        // 마치 네트워킹 하는 것 처럼 테스트 (네트워크 환경 X)
        sut.getAnimalData { animalData in
            switch animalData {
            case .success(let data):
                // 성공시 mockData와 비교
                XCTAssertEqual(data.name, response?.name)
            case .failure:
                // 바로 실패해주기
                XCTFail()
            }
        }
    }
}
```

앞서 말했던 것 처럼 실제 네트워킹이 아니고 임의로 만들어준 `dataTask()`를 사용하고 있기에 비동기가 아닌 동기적으로 작동할 것이다. 이에 비동기 메서드를 테스트하기 위한 `XCTestExpectation()`은 필요하지 않을 것 같다. 

코드를 보면 mock data를 대기시켜두고, `getAnimalData` 메서드를 호출하고 이후 코드 블록에서 성공 시에 데이터와 비교해주고 있다. 

만약 비동기 메서드로 만들어줬다면 아래와 같이 구현해주면 된다. 

```swift
func test_Animal_데이터_불러오기_성공() {
    let expectation = XCTestExpectation()
    let response = try? JSONDecoder().decode(Animal.self, from: AnimalData.data)

    sut.getAnimalData { animalData in
        switch animalData {
        case .success(let data):
            XCTAssertEqual(data.name, response?.name)
        case .failure:
            XCTFail() // 바로 실패해주기
        }
        expectation.fulfill() // expectation이 충족됨을 알림
    }

    wait(for: [expectation], timeout: 5.0) // expectation fulfill이 되기를 5초간 기다림 (비동기적으로 실행되기에 일정시간 기다려 줄 필요가 있음)
}
```

반대로 실패같은 경우도 유사하게 테스트해준다. 

```swift
func test_Animal_데이터_불러오기_실패() {
    sut = APIService(session: MockURLSession(isSuccess: false))

    sut.getAnimalData { result in
        switch result {
        case .success:
            XCTFail()
        case .failure(let error):
            XCTAssertEqual(error.localizedDescription, "unknownError")
        }
    }
}
```

실패하는 경우이기에 mock data는 필요하지 않고, 에러로 비교해주면 된다. 

우리가 제어할 수 없는 네트워킹에서 발생하는 에러들을 모두 제외해두고 **로직**만을 테스트하고 있다는 것이 큰 장점이 될 것 같다. 

또한 의존성 만들고, 의존성을 주입하여 테스트를 한다는 것이 이 과정의 핵심이 될 것 같다. APIService의 URLSession에 대한 의존성을 낮춰주었다는 것도 하나의 장점으로 가져갈 수 있을 것 같다!


## 고민된 점 
- 네트워크 환경이 아닌 상황에서 네트워킹 메서드 Unit test하는 방법

## 해결 방법 
- 배민의 기술 블로그 참고!
- 실제로 의존성 생성/주입을 하며 예제에 적용

---

**Ref**

https://techblog.woowahan.com/2704/

https://www.swiftbysundell.com/articles/testing-networking-logic-in-swift/
