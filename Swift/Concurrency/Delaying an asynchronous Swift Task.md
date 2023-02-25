# Delaying an asynchronous Swift Task

- 우리 대부분 다양한 비동기 작업이 생성된 후 가능한 빨리 시작되기를 원하지만 가끔은 약간의 지연을 두고 실행하고 싶을 때도 있습니다.
- 다른 작업이 먼저 완료될 때까지의 시간을 주거나 어떠한 형태의 "debouncing"을 주기 위해서일 것입니다.

<br/>

- 스위프트의 `Task`에 지연시키도록 하는 직접적인 방법은 없지만 task에게 nanoseconds동안 sleep하도록하여 해당 동작을 수행할 수 있습니다.

<img width="813" alt="CleanShot 2023-02-25 at 14 18 55@2x" src="https://user-images.githubusercontent.com/42647277/221339883-b0272bfc-72fa-4944-b35e-28d733566eec.png">

> `Task.sleep`을 호출하는 것은 시스템에서 제공하는 `sleep`과는 매우 다릅니다. `Task`버전은 다른 코드와 관련하여 완전히 non-blocking하기 때문입니다.

- `Task.sleep` 에 `try`키워드가 붙은 이유는 sleep하고있는 동안에 task가 취소되면 에러를 던지기 때문입니다.
- 예를 들어 살펴보자면, 비동기 작업이 완료되는데 150milliseconds 이상 걸리는 경우 뷰 컨트롤러가 로딩 스피너를 표시하도록 구현하려면 다음과 같습니다:

​	<img width="814" alt="CleanShot 2023-02-25 at 14 23 42@2x" src="https://user-images.githubusercontent.com/42647277/221340030-37ca72e2-3dd7-4f01-8f6d-5c847b299b14.png"> 

> 위의 내용은 `Task`를 사용하여 view controller의 컨텐츠를 로드하는 방법에 대한 완전한 예제가 아닙니다. 예를 들어, 새 작업을 시작하기 전에 기존 로드 작업이 이미 진행 중인지 확인하고 싶을 수 있습니다. 자세한 내용은 '스위프트의 동시성 시스템에서 Task가 하는 역할'을 참고하세요.

- 주어진 코드에서 지연된 작업을 많이 사용하려는 경우, 쉽게 생성할 수 있도록 간단한 추상화를 하는 것이 좋습니다.
- 예를 들어, nanoseconds를 사용하지 않고 표준 `TimeInterval`값을 사용하여 second기반의 delay를 정의할 수 있습니다.

<img width="813" alt="CleanShot 2023-02-25 at 14 30 59@2x" src="https://user-images.githubusercontent.com/42647277/221340250-39265c70-93ef-4590-979f-858a7ff4fb2d.png">

> sleep task에 대해 명시적으로 `Task<Never,Never>`로 표시하는 이유는 해당 메서드가 정확한 한정된 `Task`에서만 사용할 수 있고, 확장 범위 내에서 `Task`심볼이 현재 한정을 참조하기 때문입니다.  -> 정확히 이해하지 못함..

- 위의 확장을 사용하면, 이제 간단하게  `Task.delayed`호출을 통해 어디서든 지연된 task를 생성할 수 있게 됩니다.
- 한 가지 단점이 있다면, task클로저 안에 `self`를 수동적으로 캡쳐해야만 한다는 점입니다:

<img width="813" alt="CleanShot 2023-02-25 at 14 41 37@2x" src="https://user-images.githubusercontent.com/42647277/221340644-021e16e7-de41-4b0e-b2f9-8c23ac0ed987.png">

- 그 사소한 문제를 해결할 수 있는 방법이 있습니다. 바로 "semi-public"인 `_implicitSelfCapture`속성을 사용하는 것입니다.
- 이것은 스위프트 표준 라이브러리에 내장된 모든  `Task` 클로저가  `self`를 자동으로 캡쳐하도록 사용하는 것입니다.

<img width="816" alt="CleanShot 2023-02-25 at 14 45 57@2x" src="https://user-images.githubusercontent.com/42647277/221340793-20045e4c-df0b-4283-9d27-626b453cfaf2.png">

- 그러나 위의 속성은 아직 Swift의 public API의 공식적인 부분이 아니기 때문에 프로덕션 코드에서 사용하는 것은 권장하지 않습니다. 이 말은 즉, 언제든지 중단될 수 있다는 뜻입니다.

<br/>

- `Task`의 API뿐만 아니라 GCD나 타이머, Objective-C런타임과 같은 예전 도구들을 사용하여 이러한 delay를 구현할 수 있습니다.
- 그러나 Swift의 새로운 동시성 시스템을 사용하여 비동기 코드를 작성할 때, `Task`타입을 직접적으로 사용하여 delay를 구현한다면 정말 편리할 것입니다.
