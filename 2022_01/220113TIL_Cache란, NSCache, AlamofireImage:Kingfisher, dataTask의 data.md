# Cache란, NSCache, AlamofireImage/Kingfisher, dataTask의 data

## 22O113_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. 캐시 (Cache) 

우선 캐시란 무엇일까? 
의미를 간략하게 알아보기 위해 위키를 참고해보자. 

> 캐시는 컴퓨터 과학에서 데이터나 값을 미리 복사해 놓는 임시 장소를 가리킨다. 캐시는 캐시의 접근 시간에 비해 원래 데이터를 접근하는 시간이 오래 걸리는 경우나 값을 다시 계산하는 시간을 절약하고 싶은 경우에 사용한다. 캐시에 데이터를 미리 복사해 놓으면 계산이나 접근 시간 없이 더 빠른 속도로 데이터에 접근할 수 있다.

설명되어 있는 것 처럼 데이터에 접근하는 시간을 효율적으로 개선시켜주는 역할을 한다.

조금 더 컴퓨터적인(?) 말로 설명해보면, 캐시란 프로세서 내부나 외부에 있으며 처리 속도가 빠른 프로세서와 상대적으로 느린 메인 메모리의 속도 차이를 보완하는 고속 버퍼라고 볼 수 있다. 

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FORwoe%2FbtranyteLoR%2FHDgsXiBK7jTlqLCnFkl8E1%2Fimg.png)

여기서 중요한 개념이  **캐시 히트/캐시 미스**이다.

-   **캐시 히트**  : 필요한 데이터 블록이 캐시에 존재하는 경우
-   **캐시 미스**  : 필요한 데이터 블록이 없는 경우

예를 들어서 프로세서가 A라는 데이터가 있니 라고 물었을 때, 캐시에 A라는 데이터가 없다면 이를  **캐시 미스**라고 하며, 캐시는 메인 메모리에 가서 A를 캐시 안으로 가지고 들어온다. 그리고나서 A를 프로세서에게 전달하게 된다.

캐시 미스의 과정을 살펴보자. 

> [캐시 미스] 프로세서 > 캐시 > 메인 메모리 > 캐시 > 프로세서

일전의 과정으로 이제 A는 캐시에 올라와 있게 된다. 만약 그 다음에 또 A를 필요로 한는 경우가 온다면 A는 이미 캐시에 존재하기에 굳이 메인 메모리에 가지 않고도 A를 얻을 수 있다. 이런 경우를  **캐시 히트**라고 한다. 굳이 메인 메모리까지 가는 시간을 들이지 않고 필요한 데이터를 가져다 쓸 수 있는 것이다.

> [캐시 히트] 프로세서 > 캐시 > 프로세서

사실 캐시 미스가 발생했을 때, 메인 메모리에 가서 A만 가져오지는 않고 그 주변의 데이터들도 캐시로 함께 가져오게 되는데 이 크기를 정하는 것을  **캐시 블록(라인)**이라고 한다. 캐시 블록과 실제 프로그래밍 코드를 효율적으로 짜는 것은 깊이 연관이 있는데 다음 경우를 보면 이해가 쉬울 것이다.

예를 들어서 이중 for문으로 각 요소들에 접근하는 경우를 생각해보자. 두 가지 경우가 있을 것이다.

```swift 
// arr는 5x5 배열

// 1번
for i in 0..<5 {
    for j in 0..<5 {
        print(arr[i][j])
    }
}

// 2번 
for i in 0..<5 {
    for j in 0..<5 {
        print(arr[j][i])
    }
}
```

사실 진행 방향에 차이만 있다고 느낄 수 있지만 이는  **지역성**이라는 개념과 큰 연관이 있다.

#### 지역성

지역성은 두 가지 종류가 있다.

-   **공간적 지역성**
    -   참조한 주소와 인접한 주소를 참조하는 특성

즉, 주소를 한 번 참조하면 그 다음에 그 주변 주소를 찾을 확률이 높다는 것을 의미한다. 예를 들어 코드를 짰을 때, 1번 라인이 실행된 다면 그 다음에 주변인 2번 라인이 실행될 가능성이 높다는 것과 같다.

-   **시간적 지역성**
    -   참조한 주소를 곧 다시 참조하는 특성

예를 들어 for문의 경우 총 2번 반복한다고 하면 1번째 이후 2번째에 다시 참조할 가능성이 높다는 것. 빙글빙글 반복되는 경우에 시간적 지역성이 해당된다.

아무튼 이 지역성이 무슨 연관이 있을까?

앞서 말했던 것 처럼 캐시는 블록 사이즈에 따라 얼만큼의 데이터를 메인 메모리에서 캐시로 가져올지를 정하게 된다.

예를 들어 캐시 블록이 5인 경우라고 생각해보고 위에 작성한 코드의 진행 방식을 보면 다음과 같다.

**[1번 경우]**

 ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcpXN0G%2FbtranasFlxo%2FzXekP88DYVAkr9Ac3NZYkk%2Fimg.png)

처음에만 캐시 미스가 발생하고, 그 다음부터는  **공간적 지역성**  덕분에 캐시 히트의 장점을 누릴 수 있다. 행이 바뀔 때만 캐시 미스가 발생하게 되며 나머지 경우는 모두 캐시 히트하여 효율적으로 운영할 수 있다.

**[2번 경우]**
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FUFzMc%2Fbtran97Hnp8%2FojCdY47PqAic8rDkaYOcZk%2Fimg.png)

`[j][i]`로 인덱싱하는 경우 캐시 미스가 많이 나게 된다. 결론적으로 캐시 히트가 많은 1번 경우가 더 메모리 효율적인 것이다.

그래서  **지역성**은  **캐시 적중률**과 밀접하다는 것이다. 그리고 주변 것들을 읽는데  **캐시가 효과적**이라는 것이다!

아무튼 이처럼 자주 사용될 것 같은 것들을 캐싱해두어서 사용하면 효율적으로 작업을 처리해 볼 수 있을 것이다. 

iOS에서는 크게 2가지 종류의 캐시를 사용하게 된다. 

- 메모리 캐시
	- `NSCache`를 통해 사용 가능하다
	- 처리속도가 빠르지만 저장 공간이 작다
	- 앱이 종료되면 캐시 또한 사라진다. 
- 디스크 캐시 
	- `FileManager`를 통해 사용 가능하다
	- 기기 내부에 데이터를 저장하기 때문에 앱이 종료되어도 캐시가 사라지지 않는다. 
	- 저장 공간은 상대적으로 크지만, 파일 I/O로 인한 시간 때문에 처리 속도가 메모리 캐시보다 느리다. 
	- ex. 카톡의 이미지 보관	

#### [NSCache](https://developer.apple.com/documentation/foundation/nscache)

리소스가 부족할 때 제거될 수도 있는 임시 키-값 쌍을 임시로 저장하는데 사용하는 변경 가능한(mutable) collection이다. 

```swift
class  NSCache<KeyType, ObjectType> : NSObject where KeyType : AnyObject, ObjectType : AnyObject
``` 

캐시 객체는 변경 가능한 collection들과 몇 가지 다른 점들이 있다. 

- `NSCache` 클래스는 캐시가 시스템 메모리를 너무 많이 사용하지 않도록 다양한 자동 제거 정책을 포함하고 있다. 다른 어플리케이션에서 메모리가 필요한 경우, 이러한 정책은 캐시에서 일부 항목을 제거하여 메모리 설치 공간을 최소화한다. 
- 캐시를 직접 잠그지(lock)않고도 서로 다른 쓰레드에서 캐시에 데이터를 추가, 제거, 조회할 수 있다. 
- `NSMutableDictionary`객체와는 달리 캐시는 삽입된 key 객체들을 복사하지 않는다. 

일반적으로 생성하는데 비용이 많이 드는 임시 데이터 객체를 일시적으로 보관하기 위해 `NSCache` 객체를 사용한다. 저장된 객체를 재사용하면 연산을 다시할 필요가 없어서 성능상의 이점을 제공해 줄 수 있다. 하지만, 저장된 객체는 어플리케이션에 크게 중요하지 않으며, 메모리가 부족할 경우 제거될 수 있다. 만약 제거된다면, 추후에 조회할 때 다시 연산하여 생성해줘야 한다. 

`NSCache`를 사용하여 `UIImage`를 저장하는 예시를 살펴보자. 

순서는 아래와 같다.

1. `NSCache` 인스턴스 생성
2. 캐시에 원하는 key에 해당하는 값이 있는지 확인
3. 있다면 바로 캐시에서 이미지 꺼내서 사용
4. 없다면 캐시에 이미지 등록 후 사용

전체 코드를 보고 하나하나 살펴보자.

```swift
class ImageLoader {
    private static let imageCache = NSCache<NSString, UIImage>()
    
    static func loadImage(url: String, completion: @escaping (UIImage?) -> Void) {
        // url이 비어있다면 nil처리
        if url.isEmpty {
            completion(nil)
            return
        }
         
        // URL 형식으로 변환
        let realURL = URL(string: url)!
        
        // 캐시에 있다면 바로 반환
        if let image = imageCache.object(forKey: realURL.lastPathComponent as NSString) {
            print("캐시에 존재 😎")
            // UI는 메인 쓰레드에서 진행
            DispatchQueue.main.async {
                completion(image)
            }
            return
        }
        
        // 캐시에 없다면
        DispatchQueue.global(qos: .background).async {
            print("캐시에 없음 🥲")
            // 데이터 타입 변환
            if let data = try? Data(contentsOf: realURL) {
                // 이미지 변환
                let image = UIImage(data: data)!
                // cache에 추가
                self.imageCache.setObject(image, forKey: realURL.lastPathComponent as NSString)
                // UI는 메인 쓰레드에서 진행
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

우선은 `NSCache` 객체를 생성해준다. 

```swift
private static let imageCache = NSCache<NSString, UIImage>()
```

이제 캐시에 원하는 key가 있는지 확인한다. 

```swift
if let image = imageCache.object(forKey: realURL.lastPathComponent as NSString) {
    print("캐시에 존재 😎")
    // UI는 메인 쓰레드에서 진행
    DispatchQueue.main.async {
        completion(image)
    }
    return
}
```

캐시에 원하는 데이터가 있는 경우이기 때문에 별도의 과정 없이 image를 바로 사용할 수 있다. 

반면에 캐시에 없는 경우라면 url을 직접 `UIImage`로 변환해주고 캐시에 저장하고 사용하면 된다. 

```swift
DispatchQueue.global(qos: .background).async {
            print("캐시에 없음 🥲")
            // 데이터 타입 변환
            if let data = try? Data(contentsOf: realURL) {
                // 이미지 변환
                let image = UIImage(data: data)!
                // cache에 추가
                self.imageCache.setObject(image, forKey: realURL.lastPathComponent as NSString)
       // UI는 메인 쓰레드에서 진행
       DispatchQueue.main.async {
           completion(image)
       }
   } else {
       DispatchQueue.main.async {
           completion(nil)
       }
   }
}
```

간략하게 `NSCache`를 사용해서 메모리 캐시를 구현해봤다. `object()`를 통해 캐시를 조회하고, `setObject()`를 통해 캐시에 데이터를 추가하는 것이 핵심이라고 볼 수 있을 것 같다. 

> ### UIImage init
> - `UIImage(named:)`는 기본적으로 시스템 캐시를 사용한다. 이 메서드는 시스템 캐시를 체크하여 특정한 이름의 이미지가 있는지 확인해준다. 만약 캐시에 원하는 이미지가 없다면 이 메서드는 이미지를 asset catalog나 디스크로부터 불러와서 생성해준다. 또한 메모리가 부족할 경우 캐시가 제거될 수 있는데 현재 사용하고 있는 이미지의 경우 예외가 된다! 
> 
> 이에 캐싱을 원하지 않을 경우 `UIImage init(contentsOfFile:)`를 사용할 것을 권장한다.
> - `UIImage init(contentsOfFile:)`: 이미지를 불러오나 시스템 캐시를 사용하지 않는 방법이다. 
> 
> 이에 일회성 이지미를 시스템 이미지 캐시에 보관하면 잠재적으로 앱의 메모리 사용성을 향상 시킬 수 있다! 
> 상황에 맞게 적재적소에 쓰면 좋을 것 같다. 

`FileManager`를 활용한 디스크 캐싱은 다음에 알아보자...

[ToDo]
https://developer.apple.com/documentation/foundation/url_loading_system/accessing_cached_data
도 읽어보자..!

### 2. AlamofireImage VS Kingfisher

-  AlamofireImage
	-  이미지를 별도로 캐싱해주어야 한다. (기본적으로 캐싱을 해주지 않는다)
	- memory warning을 받으면 `removeAllImages()`가 호출되어 캐시를 비운다. 
	- 캐시가 만료되는 정책은 따로 없다. 

> URLCache is quite powerful and does a great job reasoning through the various cache policies and Cache-Control headers.

```swift
let imageCache = AutoPurgingImageCache()
let avatarImage = UIImage(data: data)!

// Add
imageCache.add(avatarImage, withIdentifier: "avatar")

// Fetch
let cachedAvatar = imageCache.image(withIdentifier: "avatar")

// Remove
imageCache.removeImage(withIdentifier: "avatar")

```

-   Kingfisher 
	- 이미지를 url로부터 다운로드 받으면 기본적으로 메모리/디스크 캐시에 저장된다.
	- 메모리 캐싱을 위해 `NSCache`를, 디스크 캐싱을 위해 `FileManager`를 사용하고 있다. 
	- 캐시 만료 일자 및 크기 제한 설정이 가능하다. 
	- AlamofireImage와 동일하게 memory warning을 받게 되면 `clearMemoryCache`를 호출하여 메모리 캐시에 있는 데이터를 모두 지운다. 

> Kingfisher will download the image from url, send it to both memory cache and disk cache, and display it in imageView

```swift
let url = URL(string: "https://example.com/image.png")
imageView.kf.setImage(with: url)

```

### 3. URLSession `dataTask()`의 `data`

`dataTask()` 메서드를 실행하고 `resume()` 해줌에 따라 data, response, error를 얻을 수 있는데, 이 때 data를 String 타입으로 변환시켜주면 서버의 메시지를 얻을 수 있다고 한다. 
(신나, 나무 감사합니다!)

사실 이는 프로덕션 코드에서는 크게 사용될 일은 없을 것 같고 디버깅할 때 LLDB에서 유용하게 사용할 수 있을 것 같다.

```swift
po String(data: data, encoding: .utf8)
```

서버에서 code와 message를 내려준다. 이 때 메시지를 참고하면 어떠한 이유로 에러가 발생했는지 원인을 파악해 볼 수 있을 것 같다. 

## 고민된 점 
- 캐시란?
- NSCache란? 구현 방법은?

## 해결 방법 
- 이미지 메모리 캐싱 방법 구현!

---

**Ref**

https://github.com/ChaminLee/SpacePicture/blob/master/SpacePicture/SpacePicture/ImageLoader.swift

https://leechamin.tistory.com/503?category=1012929#%EC%A-%--%EC%--%AD%EC%--%B-

https://jryoun1.github.io/swift/Cache/

https://developer.apple.com/documentation/uikit/uiimage/1624112-init

https://developer.apple.com/documentation/uikit/uiimage/1624123-imagewithcontentsoffile

https://developer.apple.com/documentation/uikit/uiimage/1624146-init
