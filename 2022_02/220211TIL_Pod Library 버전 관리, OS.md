# Pod Library 버전 관리, OS

## 220211_TIL

백신 맞아서 그런지... 뻐근하다.. 몸관리를 잘 해야겠다..!

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### 1. Pod Library 버전 관리 

만약 pod library에 변경사항이 생겨서 버전을 업데이트 해야한다면 어떻게 해야할까? 

1. `.podspec` 파일 수정 
2. 버전에 맞는 git tag 생성
3. podspec 파일 검사
4. 배포!

#### 1.  `.podspec` 파일 수정 

우선은  `.podspec` 파일 내에 있는 version을 수정해줘야 한다. 

기존에 '0.1.1'이었다면 '0.1.2'으로 간단하게 수정하면 된다. 

#### 2. 버전에 맞는 git tag 생성

git tag란 git에서 각 커밋에 달 수 있는 꼬리표 같은 것인데, 주로 release 버전을 표시하는데 이용된다고 한다. 

태그는 간단하게 생성할 수 있다. 

```
// git tag 생성
git tag 0.1.2

// git tag 원격 브랜치 반영
git push origin 0.1.2
```

tag를 생성하고 원격 브랜치에 반영시켰다. 

#### 3. podspec 파일 검사

```
pod spec lint {라이브러리 이름}.podspec
```

이렇게 입력만 해주면 알아서 podspec에 이상이 없는지 확인해준다. 

만약 이상이 있다면 에러 메시지를 잘 읽어보고 podspec의 값을 수정해주면 된다! 

![](https://i.imgur.com/wd6kSQI.png)

#### 4. 배포

```
pod trunk push {라이브러리 이름}.podspec
```

처음에 라이브러리를 배포했던 것 처럼 동일하게 입력해준다. 

![](https://i.imgur.com/KQp5OG8.png)

그러면 업데이트에 성공했다는 메시지가 뜬다! 

이렇게 pod library의 버전관리를 할 수 있는 것이다! 


### 2. OS 

memory management 챕터를 모두 수강했다. 

주소 변환에 OS가 하는 일이 없었다는 게 조금 충격이었다. 이유는 OS가 개입하게 되면 cpu가 프로세스 사이에서 왔다갔다 하게 되는데, 이것 자체가 말이 안된다고 한다... 그래서 하드웨어적으로 이루어지는 일이라고 한다..!

[Memory Management 4](https://github.com/ChaminLee/CS_Study/blob/main/OS/21.%20Memory%20Management%204.md)

## 고민된 점 
- Pod Library 버전 관리는 어떻게 할까?
- 메모리 주소 변환의 과정은 어떻게 이루어질까?

## 해결 방법 
- 내가 만든 Pod library 버전 업데이트 해보기
- OS 강의 수강 
---

**Ref**

야곰닷넷-오픈소스 라이브러리 만들기

https://github.com/ChaminLee/CS_Study/blob/main/OS/21.%20Memory%20Management%204.md
