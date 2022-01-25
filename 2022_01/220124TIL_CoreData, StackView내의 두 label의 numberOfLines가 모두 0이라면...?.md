# CoreData, StackView내의 두 label의 numberOfLines가 모두 0이라면...?

## 220124_TIL

## 목차 
- [학습 내용](#학습-내용) 
- [고민된 점](#고민된-점)
- [해결 방법](#해결-방법)


## 학습 내용

### CoreData

CoreData를 알아보기 전에...

DB란 데이터의 집합체 그 자체를 의미한다. CoreData는 데이터의 관리도 해주기 때문에 완전히 DB라고는 말할 수 없고, 객체 그래프를 관리하는 프레임워크라고 정의해 볼 수 있다. 

- ORM
	- 객체와 관계형 데이터 베이스의 데이터를 자동으로 매핑해주는 것
    - 연결다리 같은 것
    - 일부 맞는 말이긴 함 

CoreData는 아래의 기능들을 지원한다고 간략하게 말 할 수 있다.
- 저장
- 되돌리기 (흔들어서)
- 백그라운드 작업
- 뷰 동기화 작업
- 버전 관리, 마이그레이션

이제 `저장`, `불러오기`, `삭제` 기능을 각각 코드로 구현해보자. 

> 0. 기본 설정하기!

우선 dataModel을 만들고 entity까지 넣은 상태라고 가정하고, `persistentContainer`를 만드는 단계부터 시작해보자! 

우선 문서에서는 appDelegate에 만들어야 한다고 했지만, 싱글톤을 사용해서 타입을 만들어도 무방하다고 한다!

```swift
class PersistentManager {
    // 싱글톤 객체
    static let shared: PersistentManager = PersistentManager()
    
    // persistentContainer 객체 생성
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "Humor")
        container.loadPersistentStores { description, error in
            if let error = error {
                fatalError("persistent stores를 로드할 수 없음: \(error)")
            }
        }
        return container
    }()
    
    // managed objects를 생성,저장,불러오는 작업을 수행한다. 
    var context: NSManagedObjectContext {
        return persistentContainer.viewContext
    }
}
```


처음으로 만들어 준 `persistentContainer` 인스턴스는 `NSManagedObjectModel`, `NSManagedObjectContext`, `NSPersistStoreCoordinator`를 동시에 설정하는 역할을 가진다. 즉 아래 그림과 같이 가장 큰 벙위에 위치해있다고 볼 수 있다.

![](https://docs-assets.developer.apple.com/published/6b46a8afe9/CD-Stack~dark@2x.png)

하나씩 간략하게 알아보자!

- `NSManagedObjectModel`
	- 앱의 타입, 프로퍼티 및 관계들을 설명하는 앱의 모델 파일을 나타낸다
- `NSManagedObjectContext`
	- 앱 타입 인스턴스의 변경 내용을 추적한다. 
	- 변화를 계속해서 추적하며 업데이트해주는 역할을 한다고 볼 수 있다. 
- `NSPersistStoreCoordinator`
	- 앱 타입의 인스턴스를 저장하고(save), store에서 불러온다(fetch)

> 1. 저장

저장하기 위한 단계를 간략하게 봐보자.
- `NSManagedObjectContext`를 가져온다
- Entity를 가져온다
	- 내가 만든 모델! 
	- 아까 내가 만든 Entity!
- `NSManagedObject` 생성
- `NSManagedObject`에 원하는 값 세팅
- `NSManagedObjectContext`에 이를 저장해준다. 

코드로 살펴보자. 

우선 `NSManagedObjectContext`는 앞에 싱글톤 내부에 만들어놨으니 이를 사용하면 된다!

```swift
// CoreData 저장
func save(model: HumorData) {
	// Entity를 가져온다
    let entity = NSEntityDescription.entity(forEntityName: "Humor", in: context)

    if let entity = entity {
	    // NSManagedObject 생성
        let humor = NSManagedObject(entity: entity, insertInto: context)
	    // NSManagedObject에 원하는 값 세팅
	    // 모델의 값을 저장할 수 있도록 매칭
        humor.setValue(model.id, forKey: "id")
        humor.setValue(model.category.description, forKey: "category")
        humor.setValue(model.content, forKey: "body")
		// NSManagedObjectContext에 저장
        do {
            try context.save()
        } catch {
            print(error)
        }
    }
}
```

> 2. 불러오기 

`NSFetchRequest`의 경우 persistent store로부터 데이터를 찾아오기 위한 검색 기준에 대한 description이다. 쉽게 말하면 저장소로부터 데이터를 찾아오기 위한 역할을 한다고 볼 수 있다. 

```swift
func fetch<T: NSManagedObject>(request: NSFetchRequest<T>) -> [T] {
    do {
	    // context에서 fetch
        let humor = try context.fetch(request)
        return humor
    } catch {
        print(error)
        return []
    }
}
```

이제 위 메서드를 가지고 데이터를 불러올 수 있다. 

```swift
let fetchRequest: NSFetchRequest<Humor> = Humor.fetchRequest()
```

파라미터에는 위 fetchRequest를 넣어주면 된다. `fetchRequest()`의 경우 Entity를 NSManagedObject의 하위 클래스를 생성하면 자동으로 만들어진다!

```swift
@nonobjc public class func fetchRequest() -> NSFetchRequest<Humor> {
    return NSFetchRequest<Humor>(entityName: "Humor")
}
```

이제는 그냥 넣어주고 반환값을 적재적소에 잘 사용해주면 된다!

```swift
PersistentManager.shared.fetch(request: fetchRequest)
```

> 3. 삭제 (+전체 삭제)

특정 object를 삭제하는 방법과 context의 모든 object들을 지워는 방법을 살펴보자. 

우선 특정 object를 지우기 위해서는 해당 object를 파라미터로 받아줘야 한다. 

```swift
func delete(object: NSManagedObject) {
    context.delete(object)
    do {
        try context.save()
    } catch {
        print(error)
    }
}
```

마찬가지로 context에서 delete 작업을 수행해준다. 
삭제해줄 데이터의 경우 앞선 fetch를 이용해서 데이터를 받은 뒤 그 중 하나를 넣어주면 삭제되게 된다. 
(미리 데이터를 배열에 담아주고 원하는 데이터를 삭제해 줄 수도 있다.)

전체를 삭제하는 경우도 코드가 비슷하다!

```swift
func deleteAll<T: NSManagedObject>(request: NSFetchRequest<T>) {
    let delete = NSBatchDeleteRequest(fetchRequest: T.fetchRequest())
    do {
        try context.execute(delete)
    } catch {
        print(error)
    }
}
```
`NSBatchDeleteRequest`는 SQLite 영구 저장소의 객체를 메모리에 로드하지 않고 삭제하는 요청이다! 즉 다 지워버리겠다는 것이다.

앞선 fetchRequest를 파라미터에 넣어 지워줄 수 있다. 

```swift
let fetchRequest: NSFetchRequest<Humor> = Humor.fetchRequest()
PersistentManager.shared.deleteAll(request: fetchRequest)
```


### UILabel의 numberOfLines

horizontal stackView에서 두 label을 모두 `numberOfLines = 0`으로 설정하게 되면 stackView에서 반반 차지하게 된다. 

![](https://i.imgur.com/N82kSGq.png)

왜냐! numberOfLines가 0이 되면 UILabel은 자체 너비를 알 수 없게 된다. Layout을 참고하여 자리 잡게되는데, intrinsic size를 정하는데 혼란이 오게되는 것 같다. 

이를 막아주기 위해서는 hugging/compression priority를 주어야 한다. 

우측 Label의 content hugging priority를 `.required`로 주면 아래와 같이 예상한 모습대로, 텍스트가 길어졌을 때 유동적으로 나눠가지게 된다. 

![](https://i.imgur.com/l2k1rrh.png)

> 코드로 할 때는 defaulthigh보다는 required를 쓰는게 좋다고 한다. 아예 가장 높은 우선순위를 주어서 명확하게 해주는 게 좋다고 이해해 볼 수 있을 것 같다. 

## 고민된 점 
- CoreData 사용 방법?
- StackView내 요소들의 유동적인 크기 조절 (feat. numberoflines)

## 해결 방법 
- 공식 문서 및 스터디 자료를 통한 예제 구현 
- 리뷰어 올라프의 도움...! (감사합니다!) 
---

**Ref**

코다의 학습자료

https://developer.apple.com/documentation/coredata/setting_up_a_core_data_stack

https://zeddios.tistory.com/989?category=682195
