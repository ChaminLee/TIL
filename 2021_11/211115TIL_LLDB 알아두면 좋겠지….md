# LLDB 알아두면 좋겠지...


## 211115_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점 ](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### LLDB

LLDB를 이해하기 전에 기본적인 부분부터 짚고 넘어가보자!

- Compile이란?
	- 사람의 언어를 기계의 언어로 변환하는 것
	- 소스 코드 -> 목적 코드 
	- swift는 빌드 할 때 컴파일이 됨! 
- Link란?
	- 소스파일 A와 B를 연결시켜주는 작업 
- Build란?
	- 소스코드 파일을 실행가능한 소프트웨어 산출물로 만드는 일련의 과정 

> Note
> 1. 컴파일 언어: 
>     - 실행속도는 빠르나 버그 발견시 빠른 수정이 어려움 
>     - 배포에 약점이 있다. 
>  2. 인터프리터 언어:
>      - 사용자 실행에 따라 코드가 실행되기에 순간순간 번역하여 사용하는 것과 같음. 
>      - 배포에 장점이 있지만, 속도가 조금 더 느리다

LLDB를 알아보기 전에 또 한 가지 LLVM을 알아봐야 한다. 

> LLVM은 아키텍처별로 분리된 모듈식 미들엔드-백엔드를 중점으로 하고 있다. 프론트엔드가 여러가지 [프로그래밍 언어](https://namu.wiki/w/%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%20%EC%96%B8%EC%96%B4 "프로그래밍 언어")들을 중간 표현 코드로 번역하고, LLVM은 그 중간 표현 코드를 각각의 아키텍처에 맞게 최적화하여 실행이 가능한 형태로 바꾸는 방식이다. 처음에는 [GCC](https://namu.wiki/w/GCC "GCC")를 프론트엔드로 하고 LLVM은 미들엔드-백엔드로 사용하는 LLVM-GCC 프로젝트(DragonEgg)가 있었으나, LLVM의 자체 프론트엔드인 Clang이 등장한 이후 컴파일의 전 과정을 LLVM 툴체인으로 진행할 수 있게 되었다.  [나무위키](https://namu.wiki/w/LLVM)

즉 LLVM은 중간자 역할을 한다고 봐도 될 것 같다. 

이제 LLDB에 대해서 알아볼 수 있는 조건이 갖춰진 것 같다. 왜냐? LLDB는 LLVM의 프론트엔드에 대응하는 대표적인 디버거이기 때문이다. LLDB는 XCode 5.0 이상에서부터 이용할 수 있다고 한다. 지금 우리 버전에서는 크게 걱정할 문제는 아닌듯 싶다!

자 이제 LLDB를 이용해보자. 

XCode에서 여러 프로젝트를 진행하다가 보면 에러가 났을 때나, 일시중지 되었을 때 콘솔에 `(lldb)`라고 뜬 걸 본 기억이 있을 것이다. (지금까지 11db라는 줄 알고 이해하지도 않고 넘어갔었다..)

아무튼 lldb는 디버거인데, 디버깅을 하기 이전에 디버깅을 왜 해야할까? 말 그대로 "해충박멸"이라는 단어와 의미가 같을 수 있다. 코드에서 발생하는 버그를 잡아내는 것이 주 목적이며, 프로젝트의 규모가 클 경우 빌드 시간 또한 비례하여 커지게 될 텐데, 이러한 빌드 시간을 줄여주는 역할을 할 수도 있다. 

이제 어느정도 왜 디버깅을 해야하는지 납득이 되었으니 간단한 명령어 정도만 손에 익혀보자. 

디버깅.. lldb를 알기 전에 했던 디버깅이라고 해봤자 print 찍어보는게 다였다.. 
더 해봤자 라인 넘버 클릭해서 breakpoint를 만드는 정도...?

쉽게쉽게 생각해서 breakpoint를 생성하는 것 부터 lldb로 진행해보자. 

> 1. breakpoint 만들기 

breakpoint를 만들기 전에 우선 먼저 lldb를 띄워야(?)한다. 띄우는 방식은 생각보다 간단하다. 디버깅을 하기 위함이기 때문에, 원하는 지점에 breakpoint를 하나 찍고 빌드하거나 중지 시키게 되면 lldb 콘솔을 마주할 수 있게 된다. 

이제 breakpoint를 만드는 명령어를 보자. 
git이나 다른 터미널 명령어를 몇몇 봤겠지만 풀(full)로 늘여쓴 케이스가 있고 줄일대로 줄인 케이스가 있다. 이에 우선 최대한 많은 케이스를 봐보자. 

우선 가장 직관적인 방법, 함수에 breakpoint를 거는 방법을 보자. 

- `breakpoint set -file ViewController.swift -line 23` 
	- ViewController라는 파일의 23번째 라인에 breakpoint를 만들게!
- `br s -f ViewController.swift -l 23`
	- 이렇게 앞글자만 따서 간략한 명령어로도 동일한 작업을 수행할 수 있다. 

근데 만약에 라인 넘버에 breakpoint를 거는게 아니라, 특정한 이름으로 시작되는 함수에  breakpoint를 걸고 싶다면 어떻게 해야할까? 

- `breakpoint set --name specificFunc`
	- `specificFunc`라는 함수에 breakpoint를 만들기 
- `b -n specificFunc`
	- 짧은 버전 
- `breakpoint set --func-regx '^specific'` 
	- specifit으로 시작하는 모든 함수에 breakpoint를 만들기
- `br s -r '^specific'`
	- 역시나 짧게 가능
- `rb '^specific'`
	- 이렇게 짧게해도 되나 싶을 정도로 짧게도 가능

breakpoint를 걸 때에 조건(condition)도 줄 수 있다. XCode UI상에서는 breakpoint를 더블 클릭 혹은 우클릭 이후 Edit breakpoint하면 조건을 달 수 있다. 하지만 lldb로 달아보는게 연습이 될 것이니 lldb 명령어로 보자. 

- `br s -n "specificLabel" -condition specificLabel.text == "text"`
	- `specificLabel.text`가 text일 경우 break 하도록 설정 
- `br s -n "specificLabel" -c specificLabel.text == "text"`
	- 마찬가지로 짧게도 사용 가능 

> 2. breakpoint list 확인

이제 만들어 둔 breakpoint들도 확인할 수 있어야 한다. 

- `breakpoint list`, `br list`
	- 전체 목록 출력
- `br list -b` 
	- breakpoint list 간략하게 출력

> 3. breakpoint 삭제

만들 줄 알면, 삭제할 줄도 알아야 한다. 

- `breakpoint delete`, `br de`
	- breakpoint 모두 삭제
- `br de 1`
	- 1번 breakpoint 삭제
- `breakpoint disable`, `br di`
	- breakpoint 비활성화 
- `br di 1`
	- 1번 breakpoint 비활성화

> 4. 요소 출력/ 변경

어떤 요소를 출력하기 위해 아래와 같은 명령어를 사용한다. 

- `expression titleText.text`
- `expr titleText.text`
- `e titleText.text`

변경 또한 마찬가지이며, 바꾸고 싶은 값만 넣어주면 된다. 

- `expression titleText.text = "바꿀 텍스트"`
- `expr titleText.text = "바꿀 텍스트"`
- `e titleText.text = "바꿀 텍스트"`

> TIP. breakpoint 무시할래? 말래? 
> `expr -i false -- specificFunc()`: breakpoint 만나면 멈추기 
> `expr -i true -- specificFunc()`: breakpoint 만나도 멈추지 말고 진행

`po`는 `expression -O --`의 줄임말로, object의 description을 나타내는 명령어이다. 


### LLDB 문제 풀기 

> 1. ViewController.swift 파일의 23번째 줄에 브레이크 포인트를 설정하려면 입력해야 하는 LLDB 명령어는?

```git
br s -f ViewController.swift -l 23
```

> 2. `changeTextColor`라는 심볼에 브레이크 포인트를 설정하기 위해 입력해야 하는 LLDB 명령어는?

```git
br s -n changeTextColor
```
> 3. Breakpoint Navigator를 통해  `titleLabel`의  `text`가  `"두 번째 뷰 컨트롤러!"`인 경우에만 작동을 일시정지하고  `titleLabel`의  `text`를 출력하는 액션을 실행하도록 설정해보세요

breakpoint 생성 후 더블 클릭하여 condition과 action부분에 알맞게 조건과 해야 할 행동을 적어준다. 

condition = `titleLabel.text == "두 번째 뷰 컨트롤러!"`
action = `po titleLabel.text`

or 

```git
br s -n viewDidLoad -c 'titleLabel.text == "두 번째 뷰 컨트롤러!"' -C "po titleLabel.text" -G1
```

> 4. 오류(Error) 혹은 익셉션(Exception)이 발생한 경우 프로세스의 동작을 멈추도록 하는 방법에 대해 알아봅시다

https://cocoacasts.com/debugging-applications-with-xcode-swift-error-and-exception-breakpoints

살펴보기... 😵‍💫
    
> 5. View Controller의 뷰 위에는 사용자 눈에 보이지 않는 뷰가 있습니다. 이 뷰의 오토레이아웃 제약을 확인해서 알려주세요

하단 툴바 `Debug View Hierarchy` 버튼 클릭 후 contraints 확인 

  
> 6. 디버그 모드로 실행중인 상태에서 사용자 눈에 보이지 않는 뷰의 색상을 분홍색으로 변겅해보세요
        
```git
e self.view
e $R0!.backgroundColor = UIColor.systemPink
c
```

> 7. 두 번째 뷰 컨트롤러의 뷰가 화면에 표시된 상태에서, 두 번째 뷰 컨트롤러 까지의 메모리 그래프를 캡쳐해보세요
    
하단 툴바 `Debug Memory Graph` 버튼 클릭 후 contraints 확인 


> 8. LLDB의 특정 명령어의 별칭을 설정해줄 수 있는 명령어는 무엇일까요?
    
```git
command alias 별칭 "줄이고 싶은 명령어"
```

> 9. LLDB의  `v`,  `po`,  `p`  명령어의 차이에 대해 알아봅시다

- `p` = `expression --object-description --`
- `po` = `expression --`
- `v` = `frame variable`

```git
// po
(lldb) po testObject.name
"테스트 인스턴스입니다"

// p
(lldb) p testObject.name
(String) $R1 = "테스트 인스턴스입니다"

// v
(lldb) v self.testObject
(LetsDebug.TestObject) self.testObject = (name = "테스트 인스턴스입니다")
```

- `po`는 object description을 활용하여 출력하고, `p`와 `v`는 object를 콘솔에 보여주기 위해 data formatter를 사용한다.
- `po`와 `p`는 코드를 컴파일 한다.
- `v` 는 코드를 컴파일하거나 실행하지 않기 때문에 속도가 빠르다

## 고민된 점 

- LLDB... 넌 뭐하는 놈인지...

## 해결 방법 

- LLDB 강의 수강 및 예제 풀이 + 예제 만들어서 실행해보기
---

**Ref**
[yagom-lldb](https://yagom.net/courses/start-lldb/)
[yagom-lldb-exercise](https://github.com/yagom-academy/ios-lets-debug)
[LLDB-po,p,v](https://medium.com/@dubemike/level-up-your-debugging-skills-with-lldbs-v-p-and-po-commands-fec76c1ffee)
[LLVM](https://namu.wiki/w/LLVM)


(+) ToDo
- LLDB 디버깅 세션 들어보기 
	- [beyond po](https://developer.apple.com/videos/play/wwdc2019/429/)
	
