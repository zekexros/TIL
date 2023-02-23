# Memory management when using async/await in Swift

- 앱의 메모리 관리는 비동기 호출을 수행하고 처리하기 위해 시간이 지남에 따라 다양한 객체와 값을 캡쳐하고 유지해야 하는 경우가 많기 때문에 비동기식 코드의 컨텍스트 내에서 수행하기 까다로운 경향이 있습니다.
- 스위프트의 `async/await`문법은 비동기 작업을 쉽게 작성할 수 있게 해주지만, 비동기 코드와 관련된 다양한 task및 객체에 대한 메모리를 관리할 때는 여전히 주의해야 합니다.

<br/>

## Implicit captures

- `async/await`(그리고 동기 컨텍스트로부터 호출할 때 래핑에 사용하는 `Task`타입)의 흥미로운 점은 비동기 코드가 실행되는 동안 객체와 값이 암시적으로 캡처되는 경우가 많다는 것입니다.

<br/>

- 예를 들어, `DocumentViewController`에 대해 봅시다. 이것은 주어진 URL에서 다운로드하고 `Document`를 표시합니다.
- 유저에게 뷰 컨트롤러가 나타날 때 다운로드를 느리게 실행되도록 하기 위해 뷰 컨트롤러의 `viewWillAppear`메서드 내에서 해당 작업을 실행하고, 사용 가능한 경우 다운로드한 문서를 렌더링하거나 오류를 표시합니다.

<img width="813" alt="CleanShot 2023-02-23 at 11 16 07@2x" src="https://user-images.githubusercontent.com/42647277/220807181-1a70ce7b-c8a8-4887-94b9-65bbe6a496e3.png">

- 위의 코드를 살펴보면 객체가 캡쳐되지 않는 것처럼 보일 수 있습니다.
- 결국 비동기 캡쳐는 전통적으로 탈출 클로저 안에서만 발생했으며, 클로저 내에서 지역 변수나 메서드에 접근할 때마다 항상 명시적으로 self를 참조해야 합니다(여기서 self는 클래스의 인스턴스를 참조할 때).

<br/>

- 따라서 `DocumentViewController`를 표시하기 시작했지만, 다운로드가 완료되기 전에 다른 곳으로 이동하면 외부 코드(예를 들어 `UINavigationController`와 같은 부모)가 강한 참조를 유지하지 않으면 할당이 해제될 것으로 예상할 수 있습니다.
- 하지만 예상과는 달리 실제로는 그렇지 않습니다!

<br/>

- 이유는 우리가 `Task`를 생성하거나 `await`를 사용하여 비동기 호출의 결과를 기다릴 때마다 발생하는 (앞서 언급한)암시적 캡처 때문입니다.
- `Task`내에서 사용되는 모든 객체는 멤버를 참조할 때마다 `self`를 포함하여 해당 작업이 완료(혹은 실패)될 때까지 자동으로 retain됩니다.

<br/>

- 대부분의 경우, 이 동작은 문제되지 않으며 캡처된 task가 완료되면 캡쳐된 모든 객체가 결국 해제되기 때문에 실제 메모리 누수로 이어지지 않을 가능성이 높습니다.
- 그러나 `DocumentViewController`에 의해 다운로드된 문서가 잠재적으로 상당히 클 것으로 예상하고 사용자가 다른 화면으로 빠르게 탐색하는 경우 여러 뷰 컨트롤러가 계속 메모리에 남아있는 것을 원하지 않는다고 가정해 보겠습니다.

<br/>

- 이러한 종류의 문제를 해결하기 위한 고전적인 방법은 `weak self`캡처를 수행하는 것입니다. 이는 종종 클로저 내에서  `guard let self` 표현을 수반하죠(약한 참조를 강한 참조로 전환하기 위해서 해당 표현을 사용합니다).

<img width="813" alt="CleanShot 2023-02-23 at 11 35 19@2x" src="https://user-images.githubusercontent.com/42647277/220809332-ea47235d-c782-416c-855b-0b82d2ddff9f.png">

- 안타깝게도, 이 경우에는 작동하지 않습니다. 비동기로 작동하는 `URLSession`의 호출이 중단되는 동안 `self`에 대한 참조는 여전히 유지됩니다. 그리고 클로저 내의 모든 코드 실행이 완료될 때까지 `self`에 대한 참조가 유지됩니다(함수 내의 지역 변수가 스코프가 끝날 때까지 유지되는 것과 같죠).

<br/>

- 따라서 `self`에 대해 약한 참조를 하고싶다면, 클로저 전체에서 `self`에 대한 약한 참조를 일관되게 사용해야 합니다.
- `urlSession`및 `documentURL`속성을 사용하는 것을 좀 더 간단하게 만들기 위해 별도로 캡쳐할 수 있습니다. 그렇게 함으로써 뷰 컨트롤러 자체가 할당 해제되는 것을 막지 않습니다.

<img width="813" alt="CleanShot 2023-02-23 at 11 46 34@2x" src="https://user-images.githubusercontent.com/42647277/220810696-1cbee664-85f7-4a4b-ae38-bc3bb82d0cea.png">

- 좋은 소식은 다운로드가 완료되기 전에 뷰 컨트롤러가 없어지면 성공적으로 할당 해제될 것입니다.

<br/>

- 그러나 이것이 해당 task가 자동으로 취소된다는 의미는 아닙니다. 이는 문제가 되지 않을수 있지만 네트워크 호출이 일종의 부작용을 초래하면 뷰 컨트롤러가 할당이 해제된 후에도 해당 코드가 계속 실행되어 결과적으로 버그나 예상치 못한 동작으로 이어질 수 있습니다.

<br/>

## Cancelling tasks

- `DocumentViewController`의 메모리가 부족해지면 진행 중인 다운로드 작업이 실제로 취소되도록 하는 한 가지 방법은 해당 작업에 대한 참조를 저장한 다음 뷰 컨트롤러가 할당 해제될 때 해당 취소 메서드를 호출하는 것입니다.

<img width="812" alt="CleanShot 2023-02-23 at 12 01 54@2x" src="https://user-images.githubusercontent.com/42647277/220812401-d318d690-a2eb-4254-9598-409b18bcd117.png">

- 이제 모든 것이 예상대로 작동하고 뷰 컨트롤러가 dismiss되면 뷰 컨트롤러의 모든 메모리와 비동기 상태가 해제되면 자동으로 정리됩니다. 그러나 그 과정에서 코드도 상당히 복잡해졌습니다.
- 비동기 작업을 수행하는 모든 뷰 컨트롤러에 대해 모든 메모리 관리 코드를 작성하는 것은 지루하고 `async/await`가 combine, delegate, closure에 비해 이점을 제공하는지에 대해 의문을 제기할 수도 있습니다.

<br/>

- 감사하게도 많은 코드와 높은 복잡성을 가진 위의 코드와는 다른 패턴으로 구현할 수 있습니다.

-  long-running `async`메서드가 취소되면 에러를 발생시키는 것이므로 뷰 컨트롤러가 dismiss되려고 하면 `loadingTask`를 간단히 취소할 수 있습니다. 이것은 Task가 에러를 던지고 종료하고 캡처된 객체들을 해제하도록 합니다.

- 그렇게 하면 더 이상 자기 자신을 약하게 캡처하거나 다른 종류의 수동으로 메모리 관리 작업을 수행할 필요가 없습니다.

  ![CleanShot 2023-02-23 at 12.24.14@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_y6zIfZ8FST/CleanShot 2023-02-23 at 12.24.14@2x.png)

- Task가 취소되면 이제 `showErrorView`메서드가 호출됩니다(에러가 발생하고 `self`는 해당 지점에서 메모리가 남아 있기 때문).
- 그러나 이러한 추가 메서드 호출은 성능 측면에서 완전히 무시할 수 있어야 합니다.

<br/>

## Long-running observations

- 위의 메모리 관리 테크닉들의 집합은 일종의 비동기 시퀀스 혹은 스트림에 대한 long-running 관찰을 설정하기 위해 `async/await`를 사용하기 시작하면 훨씬 더 중요해집니다.
- 예를 들어, `UserListViewController`가 `UserList`클래스를 관찰하도록 만들어서 `User`모델의 배열이 변경되면 테이블 뷰 데이터를 다시 로드합니다.

<img width="811" alt="CleanShot 2023-02-23 at 13 38 47@2x" src="https://user-images.githubusercontent.com/42647277/220822530-76a6c80d-a6a3-41aa-8ef1-3719a5dbef5c.png">

- 위의 구현은 이제 아까 `DocumentViewController`내에 구현한 Task 취소 로직이 포함되지 않으며, 이 경우 실제로 메모리 누수가 발생합니다.
- 그 이유는 `UserList` 관찰 Task가 에러를 던지거나 완료할 수 없는 `Publisher`기반의 비동기 시퀀스를 반복하기 때문에 무한정으로 계속 실행되기 때문입니다. 

<br/>

- 다행인 것은 `DocumentViewController`가 메모리에 유지되는 것을 방지하기 위해 이전에 사용했던 것과 똑같은 기술을 사용하여 메모리 누수를 고칠 수 있다는 점입니다. 즉, 뷰 컨트롤러가 disappear될 때 관찰 task를 취소합니다.

<img width="814" alt="CleanShot 2023-02-23 at 18 21 38@2x" src="https://user-images.githubusercontent.com/42647277/220866836-8c91b41b-c0f5-45c7-95e6-6a029b5d527f.png">

- `deinit`내에서 취소를 수행하는 것은 실제 메모리 누수를 다루기 때문에 위의 경우에는 작동하지 않습니다. 즉, 관찰 task의 무한 루프를 끊지 않는 한 deinit은 절대 호출되지 않습니다.

<br/>

## 결론

- 처음에는 `Taks`와 `async/await` 같은 기술이 비동기, 메모리 관련 이슈를 과거의 일로 만드는 것(이젠 더이상 메모리 관련 이슈를 안다뤄도 된다는 말)처럼 보일 수 있지만 불행히도 다양한 종류의 `async`로 표시된 것들을 수행할 때 객체를 캡쳐하고 유지하는 방법에 대해 여전의 주의해야 합니다.
- 실제 메모리 누수 및 retain cycle은 combine이나 클로저를 사용할 때보다는 쉽게 발생하지 않을 수 있지만 여전히 객체와 task가 유지하기 쉬운 방식으로 탄탄하게 코드를 관리해야 합니다.



<br/>

## 출처

- https://www.swiftbysundell.com/articles/memory-management-when-using-async-await/