# OS, OpenSource Library Day1

## 220121_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### OS 강의 듣기!

[Process Synchronization 3](https://github.com/ChaminLee/CS_Study/blob/main/OS/14.%20Process%20Synchronization%203.md) 강의를 수강하고 내용을 정리했다.

세마포어의 한계에 대해서 알아봤고, 이를 대체할만한 Monitor의 등장에 대해서 알아봤다. 또한 synchronization에서 이야기하는 대표적인 문제 3가지를 살펴보고 어떤식으로 해결해나가는지 의사 코드를 통해 학습해봤다. 

### 오픈 소스 라이브러리 개발 시작! Day 1!

오픈 소스 라이브러리를 만들어보고 싶다는 생각만 가지고 있다가, 너무 바빠서 시도해보지 못했었다. 오늘 조금의 여유 시간이 생겨서 도전해봤다. (당연히 아직 개발중...)

만들게 될 첫 오픈 소스 라이브러리는 custom alert를 더 쉽게 만들어주는 기능을 위주로 구현해 볼 것 같다. 이름은 `BeautyAlert`로 정했다! 기존에 애플이 제공해주는 alert도 심플하고 이쁘지만, 색이 들어가야한다거나, 스타일을 조금 다르게 만들어주고 싶을 때 사용할 수 있도록 구현해줄 것이다. 

1일차인 오늘은 UIView를 사용하여 커스텀하게 alert을 띄우는 부분까지는 구현을 했지만 아직 버튼의 종류, 액션을 관리하는 부분은 만들지 못했다! 

```swift 
@objc private func presentBeautyAlert() {
    let beautyAlert = BeautyAlert()
    beautyAlert.setAttribute(title: "Not Enough Money 💸", titleColor: .black, backgroundColor: .white, buttonType: .all)
    beautyAlert.setBackgroundShadow(style: .rightBottom)
    beautyAlert.setButtonAttribute(cancelTitle: "Cancel", confirmTitle: "OK", titleColor: .white, backgroundColor: .black)
    self.present(beautyAlert, animated: true, completion: nil)
}
```

사용자가 원하는 값만 주면 현재 만든 스타일 내에서 다양한 색상, 버튼 종류를 선택하여 보여주도록 했다.

![](https://i.imgur.com/fVAbkIh.gif)

처음 만드는 오픈 소스 라이브러리이니, 너무 힘줘서 하면 끝내기 힘들 것 같으니... 적당히 구현해보고, 추후에 더 고도화된 UI를 쉽게 그릴 수 있도록 만들어봐야겠다!

#### [ToDo]
- 커스텀하게 속성을 주는 방법 고민
- 버튼 액션을 커스텀하게 주는 방법
- alert의 디자인 고민 
	- 어떻게 선택하도록 할 것인가
- 전체적인 레이아웃 및 코드 점검

## 고민된 점 
- 오픈소스 라이브러리 만들어볼까...?
- 세마포어를 실제 현업에서 사용할까...?

## 해결 방법 
- 바로 시도! (만들어보니 사용하지도 않던 open, public을 사용해봐서 좋았다. 계속 이어나가서 퀄리티 좀 살려야겠다)
- 세마포어의 개념은 무조건 알아두면 좋을 것 같긴한데, 지난 리뷰어분의 이야기를 들어보면 현업에서는 쓰이지 않는 것 같다. 오히려 쓰이게 될 상황을 만들지 않는게 중요하다고 한다. 또한 OS에서도 이야기하듯 세마포어를 사용하게 되면 휴먼 에러도 많이 발생할 수 있기에 monitor나 다른 개념들을 사용해서 해결하기도 하는 것 같다. 
---

**Ref**

https://github.com/ChaminLee/CS_Study/blob/main/OS/14.%20Process%20Synchronization%203.md

https://github.com/ChaminLee/BeautyAlert

