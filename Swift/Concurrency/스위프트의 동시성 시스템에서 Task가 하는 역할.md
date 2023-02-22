# 스위프트의 동시성 시스템에서 Task가 하는 역할

- Swift의 새로운 동시성 시스템을 사용하여 비동기 코드를 작성할 때 Task를 생성하면 <u>async로 표시된 API를 자유롭게 호출</u>하고 <u>백그라운드에서 작업을 수행할 수 있는 새로운 비동기 컨텍스트에 접근</u>할 수 있습니다.
- 비동기 코드의 캡슐화 외에도 코드의 실행, 관리 취소되는 방식을 제어할 수도 있습니다.



## Task의 역할 - 동기와 비동기 코드 사이의 격차 줄이기

- UI기반의 앱 내에서 Task를 사용하는 가장 일반적인 방법은 <u>동기적인 메인 스레드 기반의 UI 코드</u>와 <u>데이터를 가져오거나 처리하는데 사용되는 백그라운드 작업(비동기)</u> 사이의 다리 역할을 하는 것입니다.

- 뷰 컨트롤러가 렌더링 해야 할 `User` 모델을 로드하기 위해 async로 표시된 API를 사용할 수 있도록 작업하는 예시를 봅시다.

<img width="690" alt="CleanShot 2023-02-22 at 23 48 11@2x" src="https://user-images.githubusercontent.com/42647277/220658015-5a8cf7f9-d78a-4181-bd64-6e843342497b.png">

- 위의 코드에서 아주 흥미로운 것은 `self`캡쳐와  `DispatchQueue.main.async`가 없다는 것입니다. 클로저나 컴바인과 같은 도구를 이용하여 비동기 작업을 처리할 때 token이나 cancellable과 같은 retain작업을 했었는데 위의 코드에서는 없습니다.
- 그렇다면 정확히 어떻게 네트워크 호출(백그라운드 스레드에서 수행됨)을 수행한 다음 `DispatchQueue.main`을 먼저 사용하여 수동적으로 `userDidLoad`나 `handleError`와 같은 UI업데이트 메서드를 수동으로 발송하지 않고 직접 호출할 수 있을까?
- 여기서 스위프트의 새로운 `MainActor`가 등장합니다. 이는 UI관련 API(UIView나 UIViewController같은)가 메인 스레드에서 올바르게 전달되도록 보장합니다.
- 스위프트의 새로운 동시성 시스템을 사용하여 `MainActor`내에서 비동기 코드를 작성하는 한, 백그라운드 큐에서 UI가 실수로 업데이트 되는 걱정을 할 필요가 없습니다.
- 위의 구현에서 흥미로운 점이 하나 더 있습니다. 바로 로딩 작업을 완료하기 위해 수동으로 retain할 필요가 없다는 것입니다. 왜냐하면 비동기 작업은 해당 `Task`가 할당이 해제될 때 자동으로 취소되지 않고 백그라운드에서 계속 실행되기 때문입니다.

<br/>

## Referencing and cancelling a task

- 그러나 특별한 경우에는 우리는 로딩하는 작업에 대한 참조를 유지하고 싶을 것입니다. 뷰컨트롤러가 disappear될 때 loading task를 취소하고싶고, 또 시스템이 `viewWillAppear`를 호출할 때 중복된 작업이 수행되는 것을 방지하기를 원할 수 있기 때문입니다.

<img width="690" alt="CleanShot 2023-02-22 at 23 47 44@2x" src="https://user-images.githubusercontent.com/42647277/220657895-72151542-e733-466a-b5f7-1c1d8f6fbfbd.png">

> Task에는 2가지 제네릭 타입이 존재합니다. 첫 번째는 반환하는 결과의 타입을 나타내고(위 코드에서는 Void) 두 번째는 에러 타입을 나타냅니다(위 코드에서는 Task안에서 에러를 처리하기 때문에 Never타입임).

- `Task`에서 `cancel`메서드를 호출하는 것은 하위의 모든 작업을 또한 취소된 것으로 표시됩니다. 따라서 뷰 컨트롤러 내에서 최상위 loadingTask를 취소함으로써 동시에 네트워크 작업들을 암시적으로 취소합니다.
- 그러나 특정 작업을 취소하는 데 필요한 실제 취소를 처리하는 코드를 구현하는 것은 각 개별 Task에 달려있습니다. 따라서 시스템이 marking측면에서 취소에 대해 자동으로 관리하고 전파하더라도 해당 취소를 실제로 처리하는 방법을 결정하는 것은 각 Task에 달려있습니다(예를 들면, 클로저 내에서 `Task.checkCancellation`을 호출을 통해서).

<br/>

## Context inheritance

- 주어진 `Task` 와 `Task`의 부모와의 관계는 `@MainActor`로 표시된 클래스(예를 들어 view나 view controller)내에서 매우 중요할 수 있습니다.
- 이유는 자식 Task는 부모와 연결될 뿐만 아니라 부모가 사용하는 것과 동일한 실행 컨텍스트를 자동으로 상속하기 때문입니다.
- 이것이 문제가 될 수 있는 경우를 설명하기 위해 `ProfileViewController`가 네트워크가 아닌 로컬 데이터베이스에서 `User`모델을 로드하고 있고 Database API가 완전히 동기화되어 있다고 가정해 봅시다.
- 언뜻 보기에 구현이 괜찮아 보일 수 있습니다. 비동기 작업이 여전히 백그라운드 스레드에서 실행될 것으로 예상할 수 있기 때문이죠.

<img width="689" alt="CleanShot 2023-02-22 at 23 47 16@2x" src="https://user-images.githubusercontent.com/42647277/220657744-351d78df-7e2c-42c8-aca9-a264f2847eda.png">

- 그러나 위의 `Task`는 실제로 비동기로 수행되더라도 `MainActor`를 사용하여 전달되기 때문에 여전히 메인 스레드에서 실행됩니다(생성된 `viewWillAppear`메서드에서 해당 컨텍스트를 상속함).
- 따라서 기본적으로 위의 `Task`는 `DispatchQueue.main.async`클로저 안에서 데이터베이스 호출을 수행하는 것과 거의 동일합니다.
- 데이터베이스 호출을 메인 스레드가 아닌 곳에서 수행하고 싶다면, `detached task`를 사용해야 합니다(이것은 자체적인 컨텍스트 내에서 실행됩니다).
- 사용하면, 뷰 컨트롤러의 메서드를 다시 호출할 때 `await`을 사용해야 합니다. 해당 메서드는 `MainActor`에 의해 격리되기 때문입니다(더 이상 task내에서 `loadingTask`속성을 nil로 직접 설정할 수 없습니다).

<img width="693" alt="CleanShot 2023-02-22 at 23 46 49@2x" src="https://user-images.githubusercontent.com/42647277/220657647-c070a61b-4522-45c2-925b-60114b43147e.png">

- 일반적으로, 자체적으로 실행되는 컨텍스트를 사용하는 새로운 top-level 작업을 명시적으로 생성하려는 경우에만 `detached task`를 사용하는 것이 좋습니다.
- 다른 상황에서는 단순히 `Task{}`를 사용하여 비동기 코드를 캡슐화하는 것이 권장됩니다.

<br/>

## Awaiting the result of a task

- 마지막으로, 주어진 `Task`인스턴스의 결과를 어떻게 `await`하는지 알아봅시다.
- 예제로 네트워크를 통해 현재 사용자의 이미지를 로드하기 위해 데이터베이스 기반 뷰 컨트롤러 구현을 확장해봅시다.
-  그러기 위해선, 우리의 `detached task`를 다른 `Task`인스턴스로 감싸고, `await`키워드를 사용하여 이미지 다운로드를 진행하기 전에 데이터베이스 로드 작업이 완료될 때까지 기다립니다.

<img width="695" alt="CleanShot 2023-02-22 at 23 46 16@2x" src="https://user-images.githubusercontent.com/42647277/220657464-c50a4fa2-4698-4a00-a72d-b770149bc043.png">

- 이전과 마찬가지로 `MainActor`로 바인딩이 되었기 때문에 top-level `Task`내에서 뷰 컨트롤러의 메서드를 다시 한 번 직접 호출할 수 있다는 점에 유의하세요.
- 실수로 잘못된 스레드에서 UI 업데이트를 수행하는 것에 대해 걱정할 필요 없이 main queue 안팎에서 수행되는 작업을 이제 얼마나 원활하게 혼합할 수 있는지 보여줍니다.

<br/>

## Conclusion

- 스위프트의 새로운 `Task`타입을 사용하면 비동기 작업 단위를 캡슐화, 관찰, 제어할 수 있습니다. 이를 통해 `async`로 표시된 API를 호출하고 완전히 동기적인 코드 내에서도 백그라운드 작업을 수행할 수 있습니다.
- 그렇게하면 새로운 기능을 염두에 두고 설계되지 않은 앱 내에서도 `async` 기능과 나머지 Swift의 새로운 동시성 시스템을 점진적으로 도입할 수 있습니다.

<br/>


## 출처

- https://www.swiftbysundell.com/articles/the-role-tasks-play-in-swift-concurrency/
