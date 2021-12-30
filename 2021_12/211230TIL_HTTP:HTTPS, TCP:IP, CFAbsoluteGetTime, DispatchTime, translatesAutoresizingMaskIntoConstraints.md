
# HTTP/HTTPS, TCP/IP, CFAbsoluteGetTime, DispatchTime, translatesAutoresizingMaskIntoConstraints

## 211230_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### HTTP/HTTPS, TCP/IP 


통신 프로토콜이란 `상호간의 규약`이다. 상호간의 협의를 통해 유형을 통일하고 이를 통해 디코드/인코드하여 데이터를 주고받을 수 있는 것이다.즉 방식이 같아야 통신이 가능해지는 것이다. 

> 80은 http / 443은 https
> https는 http에 보안이 추가된 것!

- HTTP Request Method
    - `GET` 
           - 데이터를 `조회`할 때 (값에 대한 변경이 없는 경우)
           - 멱등하다, body가 없다(모두 URL에 쿼리로 작성)
    - `POST` 
           - 데이터를 `생성`할 때 
           - 멱등하지 않다. body에 내용 담아서 요청한다.
           - 멱등하지 않기에 패킷 유실 발생시 재전송된 걸 처리하면 안된다. 
    - `PUT` 
           - 데이터 `전체를 수정`할 때 
           - 멱등하다. 같은 수정을 여러 번 보내도 상태가 유지된다. (있다면 계속 갈아끼움)
    - `PATCH` 
           - 데이터의 `부분을 수정`할 때
           - 멱등하지 않다. 데이터의 부분을 수정하는 행위이기에 매번 상태가 변경된다.
    - `DELETE` 
           - 데이터를 `삭제`할 때 
           - 멱등 : 복수로 요청해도 결과가 같다. 
           - 여러 번 삭제를 요청하면 한 번 이후에는 이미 지워졌기에 실행되지 않아 상태가 동일하다. 
        
        
> - request-query : URL에서 `?`뒤에 등장하는 요청(파라미터와 요청값 쌍) (GET)
> - request-body(payload) : payload = 데이터 자체, body = 데이터를 담아주는 역할 (POST)

        
    - header
        - `Content-Type` 은 무엇인가요?
               - 보내는 데이터(Body)의 유형을 적어주는 곳. 응답자는 유형을 보고 알맞게 파싱한다.
        - 파일을 전송할때 주로 사용되는 `Content-Type` 은 무엇인가요?
               - Multipart/form-data

- HTTP Response status
    - 2xx (성공!)
        |`200`|`201`|`202`|`204`|
        |:---:|:---:|:---:|:---:|
        |`OK` : (데이터 전송 성공/ 보통 GET에서 사용)|`Created` : (POST에 대응, 자원의 생성이 됨) response header에 접속 가능한 link가 옴(지금 당장 돌려줄 수 없으니 확인할 수 있는 url을 주게 됨), body에 응답을 줄 수 있어서 header link는 옵셔널함 |`Accepted` : 자원을 수정할 때, 너의 요청이 정상적으로 접수되었다. 이미 생성된 자원을 Path/put으로 수정. 여기서는 header link주지 않음 |`NoContents` : 요청은 정상적이지만 줄 데이터가 없다. 파싱할 body가 없음 (하지만 에러는 아님)|
    - 4xx (client의 잘못으로 인해 발생)
           
       |`400`|`401`|`403`|`404`|`406`|
        |:---:|:---:|:---:|:---:|:---:|
        |`Bad Request` : (잘못된 요청, 문법 오류, 서버가 이해 못함, Body의 형태가 서버에서 지원하는 형식과 다름)|`Unanthorized`: (권한이 없음, 인증이 만료/유효하지 않음 / 토큰의 유효시간이 지나는 경우)|`Forbidden` : 금지 |`Not Found`: (자원을 찾을 수 없음)|`Not Acceptable`: (받아들일 수 없음, 요청을 받고 수행했는데 이미 존재하거나, 없는 경우 / 처리를 했더니 내부 정책에 맞지 않음/요청의 값들 중 이상한 게 있다.) |
    - 5xx
        |`500`|`502`|`504`|
        |:---:|:---:|:---:|
        |`Internal Server`: 서버 내부 오류  |`Error Bad gateway` : 근본적인 네트워크 설정 문제 |`Gateway timeout` : 네트워크 환경 이슈|

  
    - status 를 통해서 얻을수 있는 이점은 무엇일까?
      - 응답의 종류를 보고 간략하게 한 번에 원인을 파악할 수 있다. 이에 대응하는 로직(성공/실패)을 설계할 수 있음 
      - 단순한 성공 실패가 아니라 카테고리화 되어 있어 알맞는 대응이 가능하다. 
 

- URL 과 URI
    -  URI : 통합 자원 식별자 / 리소스를 가리키는 유일한 주소(문자열) URL과 URN을 포함
    - URL : 리소스의 정확한 위치를 나타낸다. 웹에서 정해진 유일한 자원의 주소 / 주소로 찾는다(path)
    - URN : 프로토콜 필요가 없다(http 안붙여도 됨) 실제 이름을 갖고 찾는다 (위치가 아님)



### `CFAbsoluteGetTime`의 단점? 대안은? `DispatchTime`

해당 메서드는 현재 절대 시간(absolute time)을 반환한다. 

절대 시간이란 달력 시간과도 유사한 말로, 연속된 시간에서 한 지점을 의미한다. 문서를 보면 "1월 1일 2001 00:00:00 GMT"에 대해 상대적인 시간을 측정한 것을 의미한다. 양수의 값은 기준이 되는 일자보다 뒤의 일자를 나타내고, 음수의 값은 기준 일자보다 앞선 일자를 나타내는 것이다. 
-32940326이라면 1999년 12월 16일 17:54:34를 의미하는 것이다. 

하지만 이 메서드를 반복 호출한다고 해서 단조롭게 값이 증가함을 보장하지는 않는다. 시스템 시간은 외부 시간 참조와의 동기화 또는 사용자가 시스템 시간을 변경하는 경우 감소할 수 있다. 

이러한 단점 때문에 어떠한 실행 시간을 측정하거나, 검사할 때 오차가 발생하게 된다. 예를 들어 앱에서 시작 시간과 종료시간의 차이를 활용하여 수행시간을 구한다고 할 때, 도중에 사용자가 시스템 시간을 바꿔버리면 문제가 발생할 수 있다는 것이다. 

이에 다른 방법으로는 `DispatchTime`을 사용해볼 수 있다. 

`DispatchTime`의 경우 기본적인 시간과 사애적인 관계가 있으며, 나노 세컨드의 정밀도를 가지고 있다. 

애플 플랫폼에서 이 기본 시간은 Mach absolute time unit에 기반하고 있다. 

```swift
let startTime = DispatchTime.now()

// code ~~

let endTime = DispatchTime.now()
let totalTime = endTime.uptimeNanoseconds - startTime.uptimeNanoseconds
```

하지만 나노세컨드로 나오기 때문에 일반 초 단위로 변환해줘야 할 필요가 있다. 10억을 곱해주면 일반 초 단위로 변경할 수 있다!

물론 시간초를 측정하는 다른 방법(`CACurrentMediaTime()`, `Foundation.clock()` 등)들도 많겠지만 우선 `CFAbsoluteGetTime`가 가지는 단점에 주목하여 다른 방법을 사용하는 것이 더 안전할 것 같다는 것이 결론이다!

### `translatesAutoresizingMaskIntoConstraints` 란?

`view`의 autoresizing mask가 Auto Layout 제약으로 변환되는지 결정하는 bool 타입의 프로퍼티이다. 

결론 먼저 말하면 Auto Layout을 적용해주려면 해당 프로퍼티를 `false`로 두어야 하고, autoresizing mask를 쓰려면 `true`로 해줘야 한다는 것이다. 

![](https://i.imgur.com/VGreipQ.png)

autoresizing mask는 스토리보드에서 볼 수 있다. 우측 그림을 참고해가며 레이아웃을 잡아주는 것이다. 
가장 중요한 점으로 autoresizing mask가 적용되고 있을 때는 auto layout이 적용되지 않는다는 것이다!! 

양립할 수 없고 단독으로만 사용해야하기 때문에 코드로 auto layout을 구성할 때 이 프로퍼티를 false로 주게 되는 것이다. 

기본적으로 코드로 뷰를 생성하게 되면 `true`의 기본값을 갖게 된다. 스토리보드(인터페이스 빌더)에서는 시스템이 자동으로 해당 프로퍼티를 `false`로 만들어줘서 편하게 auto layout을 바로 적용할 수 있는 것이다. 

## 고민된 점 
- `CFAbsoluteGetTime`의 단점이 발생할 수 있는 상황은?
- `translatesAutoresizingMaskIntoConstraints` 에 대하여

## 해결 방법 
- 공식 문서 읽기
- 오류가 발생할 수 있는 상황에 대해 가정해봄
---

**Ref**

노루의 컴퓨터 네트워크 자료

https://zeddios.tistory.com/474

https://forums.swift.org/t/recommended-way-to-measure-time-in-swift/33326

https://developer.apple.com/documentation/dispatch/dispatchtime

https://developer.apple.com/documentation/corefoundation/1543542-cfabsolutetimegetcurrent

