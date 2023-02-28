# Automatically retrying an asynchronous Swift Task

- 개발하다보면 우리는 가끔씩 비동기적인 작업이 실패했을 때 자동으로 retry하길 원합니다. 좀 더 구체적인 예시로 일시적인 네트워크에 문제를 해결하거나 특정 형태의 연결을 다시 설정하기 위해 retry를 사용하고 싶죠.

<br/>

- 아래 코드에서는 애플의 네트워크를 구현하기 위해 Combine 프레임워크를 사용할 때 retry를 사용하고 있으며 발생한 에러를 처리하기 전에 3번의 retry를 합니다:

  <img width="811" alt="CleanShot 2023-02-28 at 21 09 21@2x" src="https://user-images.githubusercontent.com/42647277/221850682-93b1f144-9983-4e39-a785-085ed56507cd.png">

  > 위의 코드는 발생한 에러의 종류와 관계없이 로딩 작업을 최대 3회로 무조건 재시도합니다.

- 하지만 이것을 스위프트의 Concurrency를 사용하여 비슷하게 구현하려면 어떻게 해야할까요?
- 컴바인의 `Publisher`프로토콜은 `retry`오퍼레이터를 기본적으로 제공하고 있지만 스위프트의 새로운 동시성 API는 비슷한 것을 제공하지 않으므로(적어도 이 글이 작성될 때에는) 창의력을 발휘하여 만들어야 합니다.

<br/>

- 스위프트의 동시성 시스템, 특히 `async/await`에서 특별히 좋은 점은 표준 제어 구성들과 다양한 비동기 호출들을 섞어 사용할 수 있다는 것입니다. 예를 들어 `if`구문이나 `for`루프 같은 것들과 말이죠.

- 따라서 `await`으로 표시된 호출에 대한 자동으로 재시도를 구현하는 한 가지 방법은 실행하려는 비동기 코드를 일정 범위에 대해 반복하는 루프 내에 배치하는 것입니다. 이 범위는 우리가 얼만큼 재시도하는지에 대한 횟수를 나타내죠:

  <img width="809" alt="CleanShot 2023-02-28 at 21 18 49@2x" src="https://user-images.githubusercontent.com/42647277/221852462-48db6f74-da9b-4788-a43d-3ee77d8cc2d9.png">

- 위의 코드는 아주 잘 작동합니다만 제네릭하지 않죠. 우리는 프로젝트 내에 여러 곳에서 재시도할 수 있는 범용적인 로직을 추가하고 싶습니다. 해당 코드를 쉽게 재사용할 수 있도록 어떠한 형태의 유틸리티로 옮길 가치가 있습니다.

<br/>

- 그렇게 하는 한 가지 방법은 스위프트의 `Task`타입을 확장하여 자동으로 재시도하는 task를 빠르게 생성하는 편리한 API를 만드는 것입니다.

- 편리한 API를 위해 앞으로 작성할 로직은 위에 작성한 로직에다 재시도하는 최대 횟수를 파라미터로 받고, cancellation에 대한 지원도 추가하도록 구현합니다:

  <img width="810" alt="CleanShot 2023-02-28 at 21 25 32@2x" src="https://user-images.githubusercontent.com/42647277/221853759-c474c913-9a21-4ca8-9018-186ae1909d25.png">

- 위의 구현만으로도 이미 유용하고 재시도가 가능한 구현이지만 한 단계 더 나아가 봅시다.

<br/>

- 비동기 작업을 재시도할 때 각 재시도 사이에 약간의 딜레이를 추가하는 것은 아주 일반적입니다. 호출을 다시 시도하기 전에 외부 시스템(서버와 같은)에게 에러를 복구할 일종의 기회(?)를 주기 위해서일 것입니다.

- 딜레이를 지원하도록 코드를 추가하는 것은 기본으로 제공하는 `Task.sleep`을 사용하여 쉽게 구현이 가능합니다:

  <img width="812" alt="CleanShot 2023-02-28 at 21 32 28@2x" src="https://user-images.githubusercontent.com/42647277/221855092-8c02f6a5-2b2e-44c4-8546-9af45d5bb6f9.png">

  > 만약 작업이 취소되면 `Task.sleep`이 자동으로 에러를 던지기 때문에 `for`루프에서 `checkCancellation`호출을 제거할 수 있게 되었습니다. 자세한 내용은 `Delaying an asynchronous Swift Task`에서 확인하세요.

- 원한다면 우리는 "semi-public"으로서 `@_implicitSelfCapture`속성을 `operation`클로저에 추가할 수도 있습니다. 그렇게 되면 클로저를 `Task`타입 자체에 직접 전달할 때와 동일한 암시적인 self 캡처하는 동작을 제공할 수 있습니다.

<br/>

- 그러나 이는 권장하는 방법은 아닙니다(언더스코어 속성은 공식적이지 않고 표준 라이브러리에도 작성되지 않는 실험적이고 베타 성격을 띈 기능이기에 언제든지 바뀔 수 있기 때문!).

- 따라서 이전의 `SettingsLoader`예제를 리팩토링하여 새로운 `Task` 확장을 사용하여 재시도를 하는 것으로 마무리 하겠습니다:

  <img width="811" alt="CleanShot 2023-02-28 at 21 40 45@2x" src="https://user-images.githubusercontent.com/42647277/221856825-99493c17-d352-47ef-bbe2-974698d4fbf2.png">





- 주어진 task의 `value`프로퍼티를 사용하여 반환하는 값(혹은 에러)를 관찰하는 방법으로 구현하였으니 참고하세요.

<br/>

## Conclusion

- 위에서 소개한 `Task`의 확장 외에도 더 많은 방법들이 당연히 존재합니다. 예를 들어, 특정 에러에 대해서만 재시도 하도록 구현하는 방법이 있겠네요.
- 그러나 본질은 이 글이 당신에게 자동으로 재시도하는 방법에 대한 아이디어를 제공했기를 바라는 것 입니다.

<br/>

## 출처

- https://www.swiftbysundell.com/articles/retrying-an-async-swift-task/