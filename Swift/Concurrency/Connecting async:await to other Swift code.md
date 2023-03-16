# Connecting async/await to other Swift code

- async/await는 애플의 플랫폼에서 비동기 코드를 작성하는 아주 기본적인 방법이 될 가능성이 높지만 여기에 도달하는 데 상당한 시간이 소요됩니다.
- 따라서 이번 글은 새로운 async/await와 기존 비동기 코드와의 갭을 채워주는 내용으로 채워보겠습니다.

<br/>

## What's asnyc/await?

- 우선 async/await에 대해 간단하게 요약해 봅시다.
- async/await는 `async`키워드를 함수(혹은 연산 프로퍼티)에 붙여 사용하며 이것은  비동기로 작동한다는 것을 알려주고 이를 호출할 때는 `await`키워드를 사용해야 합니다.
- 이 때, 시스템은 다른 외부의 코드를 block하지 않고 비동기 호출이 완료되기를 기다리는 것과 관련된 모든 복잡한 일들을 관리합니다.

<br/>

- `DocumentLoader`은 `async`로 표시된 `loadDocument`메서드를 가지고있고, 주어진 url로부터 다운로드된 데이터를 기다리는 비동기 네트워크 호출을 수행하는 Foundation의 `URLSession`API를 이용합니다:

  <img width="811" alt="CleanShot 2023-03-16 at 23 12 34@2x" src="https://user-images.githubusercontent.com/42647277/225644197-74771b17-c2f1-4bda-9ddf-7105d7f206a6.png">

- 따라서 `async`로 표시된 함수는 `await`키워드를 prefix로 두며 다른 비동기 함수를 차례로 호출할 수 있습니다.
- 그 때, local execution은 중첩된 `await`가 완료될 때까지 일시 중지되며, 그 동안 다른 코드가 실행되는 것을 차단하지 않습니다(비동기).

<br/>

## Calling async functions from a synchronous context

- 그렇다면 비동기가 아닌 컨텍스트에서 `async`로 표시된 함수를 어떻게 호출할까?

- 예를 들어, UIKIt기반의 뷰 컨트롤러에서 위의 `DocumentLoader`를 사용하고 싶다면 어떻게 해야 할까?

- 이제 여기서 `Task`가 등장합니다. 여기서 우리가 해야 할 일은 비동기 호출을 수행하기 위해 `loadDocument`에 대한 호출을 `Task`로 감싸줍니다:

  <img width="813" alt="CleanShot 2023-03-16 at 23 22 30@2x" src="https://user-images.githubusercontent.com/42647277/225646973-645f386b-0ad4-489e-8745-efe8673a9c46.png">

- 위의 패턴이 굉장히 좋은 점은 이제 비동기 호출을 수행할 때에도 스위프트의 기본 `do/try/catch`를 이용한 에러 핸들링을 사용할 수 있다는 것입니다.
- retain cycle(순환 참조)를 피하기 위해 `weak self`와 같은 것들도 더 이상 필요 없습니다.
- 수동적으로 UI 업데이트를 메인 큐에 보낼 필요도 없습니다. 이것은 이제 `main actor`에 의해 처리됩니다.

<br/>

## Retrofitting existing APIs with async/await support

- 이제 기존 비동기 코드를 새로운 async/await패턴과 호환되도록 만드는 방법입니다.

- completion handler기반의 API를 사용하여 지정된 문서에 첨부된 모든 코멘트를 로드할 수 있는 `CommetLoader`타입이 있다고 가정해 봅시다:

  <img width="807" alt="CleanShot 2023-03-16 at 23 36 40@2x" src="https://user-images.githubusercontent.com/42647277/225651173-699a2b5d-7f59-42ec-8445-82cbcd264596.png">

- async/await와 호환되기 위해 위의 API를 크게 변경해야 하는 것처럼 생각할 수 있지만 실은 그렇지 않습니다.

- 우리가 실제로 해야할 일은 새로운 `withCheckedThrowingContinuation`함수를 사용하여 기존에 있던 메서드를 `async`로 표시하여 다음과 같이 감싸는 것입니다:

  <img width="806" alt="CleanShot 2023-03-16 at 23 40 40@2x" src="https://user-images.githubusercontent.com/42647277/225652332-5ecad880-8fe5-4737-bd87-09733d993e7f.png">

  > 클로저로 전달된 continuation은 한번만 호출할 수 있습니다. 만약 실수로 두 번 호출하면 fatal error가 발생합니다. 이 경우에는 completion handler가 오직 한 번만 호출되기 때문에 그런 불상사는 절대 일어나지 않지만 이 기술을 사용할 때 꼭 명심해야 합니다.

- 이제 위의 코드를 사용하면 기존 시스템이 제공하는 비동기 API를 호출하던 것과 같이 `await`키워드를 사용하여 `loadCommentsForDocument`메서드를 쉽게 호출할 수 있습니다.

- 예를 들어 아래의 코드는 `DocumentLoader`를 업데이트하여 각 문서에 대한 코멘트를 자동으로 가져오는 방법입니다:

  <img width="812" alt="CleanShot 2023-03-16 at 23 49 29@2x" src="https://user-images.githubusercontent.com/42647277/225654916-b5e4e4e7-1319-4ba1-b23e-b6221aac2072.png">

- async/await의 아주 좋은 점은 중첩된 호출들 추가하더라도 코드가 복잡해지지 않는다는 것입니다.
- 그저 예전부터 계속 사용해오던 동기 코드에 몇 가지 `await`키워드가 추가된 것으로 아주 간단하고 쉽게 보입니다.

<br/>

## Adapting single-output Combine publishers

- 마지막으로 Combine기반 코드를 async/await와 호환 가능하게 만드는 방법을 살펴보겠습니다.
- 스위프트의 새로운 동시성 시스템은 시간이 지남에 따라 값을 방출하는(async sequences와 streams같은) Combine과 유사한 방법을 포함합니다. 
- 우리가 하려는 것이 Combine 파이프라인에서 단일 비동기 값을 기다리는 것이라면 이를 실현할 수 있는 방법은 몇 가지 있습니다.

<br/>

- 한 가지 방법은 publisher가 방출한 첫 번째 값으로 continuation을 resume하는 singleOutput메서드를 가지도록 Combine의 `Publisher`를 확장하기 위해 이전에 사용했던 `continuation`기반의 기술을 사용하는 것입니다.

- 또한 스위프트의 클로저 캡쳐 메커니즘을 사용하여 작업이 완료될 때까지 Combine subscription의 `AnyCancellable` 인스턴스를 유지합니다:

  <img width="806" alt="CleanShot 2023-03-17 at 00 15 42@2x" src="https://user-images.githubusercontent.com/42647277/225662489-1c8bedc8-a49f-472b-9c68-011e37afd8e6.png">

- iOS15, macOS Monterey 혹은 애플의 2021 OS를 최소 배포 대상으로 사용하는 앱에서 작업하는 경우, 좀 더 간단한 버전을 사용할 수 있습니다.

- `Publisher`프로토콜에서 current publisher를 async sequence로 변환하는 새로운 `values`프로퍼티를 통해 말이죠:

  <img width="805" alt="CleanShot 2023-03-17 at 00 20 54@2x" src="https://user-images.githubusercontent.com/42647277/225664060-718f7ac5-23e3-425a-9489-12ebd73363e5.png">





- 위의 두 확장 중 하나를 사용하여 이전에 사용했던 `CommentLoader`에 Combine기반 API가 있다고 가정하면 쉽게 async/await를 새로운 `singleOutput`메서드를 사용하여 호출할 수 있습니다:

  <img width="805" alt="CleanShot 2023-03-17 at 00 23 57@2x" src="https://user-images.githubusercontent.com/42647277/225664967-6ef244f0-4611-42bb-89f6-0abc6b9c3c09.png">

- 이름에서 알 수 있듯, 새로운 `singleOutput`메서드는 주어진 Combine publisher가 내보낸 첫 번째 값만 반환하므로 시간이 지남에 따라 여러 값을 생성할 것으로 예상되지 않는 publisher에서만 사용해야 합니다.

<br/>

## 결론

- async/await는 스위프트에서 비동기 코드를 작성하는 새롭고 재밌는 방법을 제공하며 앞으로 애플의 API설계에서 매우 중요한 부분이 될 것입니다.
- 그러나 이전 버전과 호환되지 않고 아직 async/await를 사용하지 않는 다른 코드와 같이 쓸 가능성이 높기 때문에 스위프트의 새로운 동시성 시스템과 연결하는 방법을 찾는 것이 많은 팀들에게 점점 더 중요해질 것입니다.

<br/>

## 출처

- https://www.swiftbysundell.com/articles/connecting-async-await-with-other-swift-code/#adapting-single-output-combine-publishers