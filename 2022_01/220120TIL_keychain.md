# keychain

## 220120_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### !!복습 내용!!

- 함수(메서드) 콜스택의 부하를 줄이기 위해 컴파일 시점에 함수의 코드를 호출 부분에 끼워넣는 방식 
	- inline(인라이닝) `@inlinable` 으로 표시해준다. 
- tmp. cache는 백업하지 않음 
- document 영역은 사용자에게 노출 
- cache는 물리적인 개념(L1, L2 등) 뿐만 아니라 소프트웨어적(메모리) 캐싱도 사용한다. 
- 캐시는 로컬 뿐만 아니라 서버에서도 일어난다. (서버의 입장에서도 빈도있는 데이터를 보내줘야하는 경우 캐싱하기도 한다)
- NSCache는 메모리 캐시 전용 
- 키체인은 대표적으로 키-값 쌍의 데이터를 안전하게 보관할 수 있는 방법 중 하나이다. 
- 키체인은 앱을 삭제해도 유지할 수 있다!

### 키체인이란?

키체인은 유저를 대신하여 데이터 일부를 안전하게 저장하는 역할을 해준다. 주로 암호나 암호키와 같이 보안이 필요한 정보를 저장하고 싶을 때 keychain item으로 묶어준다. 

키체인은 비밀번호 뿐만 아니라, 사용자가 보안이 필요하다고 생각할 수 있는 신용카드 정보나 짧은 노트 글도 포함될 수 있다. 

![](https://docs-assets.developer.apple.com/published/0ddea9db46/1c9e8103-fae2-45f4-832c-c528d2e0c2f6.png)

또한 사용자가 필요로 하지만 인지하지 못하는 항목들을 저장할 수 있다. 예를 들어, 키 혹은 신뢰 서비스(trust service)로 관리하는 인증서나 암호화 키를 사용하면 사용자가 보안 통신에 참여하고 다른 사용자 및 기기와 신뢰를 구축할 수 있다. 이러한 아이템들은 키체인(열쇠꾸러미)을 사용하여 저장할 수 있는 것이다. 

keychain item이란 item에 넣어진 비밀 정보, 즉 키체인에 저장되는 정보를 말한다. 비밀번호나 암호 키를 저장하기 위해 키체인 아이템으로 묶어준다. 데이터와 함께 아이템의 접근성을 제어하고, 검색할 수 있도록 공개적으로 표시되는 속성 집합을 제공한다. 아래 그림과 같이 키체인 서비스는 디스크에 저장된 암호화된 데이터베이스인 키체인에서 데이터 암호화 및 키체인에 저장하는 역할을 한다.

![](https://docs-assets.developer.apple.com/published/0ddea9db46/0304151a-f84e-44b1-8632-6698ec59854b.png)

 이후 인증 프로세스는 키체인 서비스를 사용하여 아이템을 찾고 해당 데이터의 암호를 해독하게 된다. 

#### Keyed Archiver, User Defaults, CoreData 차이점
- **CoreData**
   - 데이터베이스가 아닌 객체 그래프를 관리하는 프레임워크이다.
   - 복잡하고 큰 유저데이터를 저장하기에 적합하다.
   - Thread Safe 하지 않다. 
- **Keyed Archiver**
   - 파일의 저장에 적합한 아키텍처 독립적인 형식으로 인코딩하는 방법을 제공한다.
   - NSKeyedArchiver는 Core Data보다 구현이 훨씬 간단하다.
   - 암호화가 되지 않고, objc-c에서 쓰던 방식이다.
- **User Defaults**
   - 사용자 기본 설정과 같은 데이터를 간단하고 편리하게 저장합니다. 
   - Thread safe하다.

키체인은 위 방법들과 달리 중요 정보를 암호화하여 보관한다. 

#### iOS keychain VS macOS keychain
- ios의 키체인
	- 하나의(단일) 키체인을 가진다. 
	- 앱은 자신의 키체인 아이템에만 접근할 수 있으며, 앱이 속한 그룹과 공유할 수도 있다. 
 
- Mac의 키체인
	- 여러 키체인을 가진다. (앱마다의 키체인을 가질 수 있다)

#### 키체인 서비스에서 k 접두어의 의미

k-접두어가 붙은 것은 Core Foundation에서의 상수임을 나타내기 위한 것이다. 헝가리안 표기법에서 constant를 의미하는 "c"를 적어줘야 하나, 이미 예약어로 있어서 발음이 비슷한 "k"를 썼다는 유래가 있다..!

### Core Foundation VS Foundation

- Core Foundation 
	- C 언어 기반
	- 성능에 초점을 맞춘 저수준의 프레임워크다.
- Foundation 
	- Objective-C기반
	- Core Foundation을 가진다. 
	- Core Foundation을 추상화한 프레임워크다.

## 고민된 점 
- 키체인이란?
- 다른 DB와 다른 점은?

## 해결 방법 
- 공식문서 탈탈 털기
---

**Ref**

https://developer.apple.com/documentation/security/keychain_services/keychains

https://developer.apple.com/documentation/security/keychain_services/keychain_items

https://developer.apple.com/library/archive/referencelibrary/GettingStarted/RoadMapiOS-Legacy/chapters/SurveytheMajorFrameworks/SurveytheMajorFrameworks/SurveytheMajorFrameworks.html

https://developer.apple.com/documentation/corefoundation

https://developer.apple.com/documentation/foundation

https://metalkin.tistory.com/20

야곰의 강의 


