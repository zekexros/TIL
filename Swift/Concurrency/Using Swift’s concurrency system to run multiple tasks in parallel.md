# Using Swift’s concurrency system to run multiple tasks in parallel

- 스위프트에 내장된 동시성 시스템의 장점 중 하나는 비동기 작업을 병렬로 수행하는 것이 아주 쉽다는 것입니다.
- 이번 글에서는 병렬로 작업하는 몇 가지 다른 방법과 각 기술이 유용할 수 있는 경우들을 살펴보겠습니다.

<br/>

## From asynchronous to concurrent

- 설명을 위한 예제로, 다양한 제품을 표시하는 쇼핑 앱을 작업 중이고 아래와 같은 비동기 API를 사용하여 다양한 제품 컬렉션을 로드할 수 있는 `ProductLoader`을 구현한다고 가정해 봅시다.<img width="812" alt="CleanShot 2023-02-26 at 21 43 46@2x" src="https://user-images.githubusercontent.com/42647277/221411211-8d8dcb41-86a1-4c28-a8eb-327d9ff8e032.png">

- 대부분의 경우 위의 메서드를 별도로 호출하지만, `ProductLoader`의 3가지 메서드의 결과를 결합한 `Recommendations`모델을 만든다고 가정해 봅시다<img width="806" alt="CleanShot 2023-02-26 at 21 47 48@2x" src="https://user-images.githubusercontent.com/42647277/221411334-23a2da36-5cda-48f0-8dcb-92832e9459a8.png">





- 한 가지 방법은 `await`키워드를 사용하여 각각의 메서드를 호출한 다음 각각의 결과를 사용하여 `Recommendations`모델을 만드는 것입니다:

  <img width="808" alt="CleanShot 2023-02-26 at 21 49 48@2x" src="https://user-images.githubusercontent.com/42647277/221411430-dfa2c0bc-15a4-4f8b-bcdf-a86d5ffc4c24.png">

- 위의 구현은 확실히 잘 작동합니다. 하지만 3가지의 로딩을 가진 실행들은 완전히 비동기로 작동하지만 순서대로 수행되는 것도 사실입니다.
- 따라서 `loadRecommendations`메서드가 앱의 다른 코드와 동시에 수행되고 있지만 사실 결과물들의 통합을 위해 동시성을 활용하지 않고 있습니다.

<br/>

- 위의 로딩 메서드들은 서로에게 의존성이 없으므로 순서대로 호출되야 할 이유가 없습니다. 따라서 완전시 동시에 실행되도록 어떻게 만드는지 살펴봅시다.

<br/>

- 코드를 바꾸기 위한 첫 번째 아이디어는 위의 코드를 단일 표현으로 줄이는 것일 수도 있겠네요. 하나의 `await`키워드를 사용하여 각 작업이 완료될 때까지 기다릴 수 있습니다:

  <img width="810" alt="CleanShot 2023-02-26 at 21 59 40@2x" src="https://user-images.githubusercontent.com/42647277/221411897-ea0fd42f-4c4c-48f9-a1ed-11fd1a81b949.png">

- 그러나 위의 코드가 동시성처럼 작동하게 보일지라도, 사실은 이전과 같이 여전히 순서대로 작동합니다.

<br/>

- 위 방법 대신에 각 로딩 작업을 병렬로 수행하도록 동시성 시스템에게 알려야하기에 스위프트의 `async let` 바인딩을 사용해야 합니다.
- `async let`을 사용하면 작업이 완료될 때까지 기다리지 않고 백그라운드 스레드에서 비동기 작업을 시작할 수 있습니다.

<br/>

- 그런 다음에 로드된 데이터를 실제로 사용하는 지점(`Recommendations`를 초기화 할 때)에서 이를 단일 `await`키워드와 결합하면, 상태 관리나 data races 걱정할 필요 없이 로딩 작업을 병렬로 실행하는 모든 이점을 얻을 수 있습니다:

  <img width="809" alt="CleanShot 2023-02-26 at 22 09 25@2x" src="https://user-images.githubusercontent.com/42647277/221412392-4841ef8c-dbd1-43cd-8d05-fdbcd06cc53f.png">

- 아주 깔끔해졌네요. 따라서 `async let`은 우리가 유한하고 알고있는 작업의 집합이 있을 때 여러 작업을 동시에 실행할 수 있는 방법을 제공합니다.

<br/>

## Task groups

- 이제 네트워크를 통해 이미지들을 로드하는 `ImageLoader`를 작업한다고 가정해 보겠습니다. 주어진 `URL`을 통해 단일 이미지를 로드하기 위해 아래와 같은 메서드를 사용할 것입니다:

  <img width="811" alt="CleanShot 2023-02-26 at 22 14 16@2x" src="https://user-images.githubusercontent.com/42647277/221412605-56e2814b-efd9-4be5-8cb0-11f8db169bcf.png">





- 이미지들을 간편하게 한 번에 가져오기 위해 URL배열을 받고 URL과 이미지를 각각 키와 값으로 갖는 dictionary를 반환하는 편리한 API를 만들었습니다:

  <img width="815" alt="CleanShot 2023-02-26 at 22 17 45@2x" src="https://user-images.githubusercontent.com/42647277/221412732-47a02ea2-6d63-4780-9de6-927bba619610.png">

- 이전에 `ProductLoader`작업할 때와 마찬가지로 우리는 위의 `loadImages`메서드를 각각의 이미지가 순서대로 다운로드하는게 아닌 동시에 실행하고 싶다고 가정해 보겠습니다.

- 그러나, 이번에는 수행해야 하는 작업의 수를 컴파일 타임에는 알 수 없기에  `async let`을 사용할 수 없습니다.
- 감사한 것은 스위프트의 동시성 도구 상자에는 동적으로 바뀌는 숫자의 작업을 병렬로 실행할 수 있는 도구가 있습니다. 바로 task group입니다.

- task group을 구성하려면 task내에서 에러를 던지는지 아닌지 여부에 따라 `withTaskGroup`이나 `withThrowingTaskGroup`을 사용합니다.
- 이 경우 `loadImage`메서드가 `throws`키워드로 표시되어 있으므로 후자를 선택하겠습니다.

- 이전과 마찬가지로 각 URL을 반복합니다. 다만 이번에는 각각의 이미지 로드 task가 완료될 때까지 직접 기다리지 않고 그룹에 추가합니다. 대신 각 task를 추가한 후 그룹의 결과를 개별적으로 기다리면 이미지 로드 작업을 완전히 동시에 실행할 수 있습니다:

  <img width="814" alt="CleanShot 2023-02-26 at 22 37 06@2x" src="https://user-images.githubusercontent.com/42647277/221413777-b2546be8-7b8a-47ec-a003-55b3b759b296.png">

  > `for try await` 구문에 대해 더 알아보고 싶다면 [“Async sequences, streams, and Combine”](https://www.swiftbysundell.com/articles/async-sequences-streams-and-combine)을 확인하세요.





- `async let`을 사용할 때와 마찬가지로 task가 어떤 종류의 상태든 직접 변경하지 않는 방식으로 동시성 코드를 작성할 때의 가장 큰 장점은 locking이나 serialization코드를 섞지 않아도 된다는 것. 즉, 완전히 data race이슈를 피할 수 있다는 것입니다.

- 따라서 가능한 각각의 동시 작업이 완전히 별도의 결과를 반환하도록 한 다음 최종 데이터 세트를 형성하기 위해 결과를 순차적으로 기다리는 것이 가장 좋은 접근 방식입니다.
- 향후 아티클들에서는 data race를 피하는 방법을 훨씬 자세히 살펴보겠습니다(스위프트의 `actor`를 이용하여).

<br/>

## Conclusion

- `async`로 표시되었다고 해서 반드시 동시성으로 작동하는 것은 아니라는 걸 기억하는게 중요합니다. 대신 의도적으로 코드를 작성하여 병렬로 실행해야 합니다. 그리고 이것은 <u>독립적으로 실행할 수 있는 일련의 작업을 수행할 때만 의미가 있습니다.</u>

<br/>

## 출처

- https://www.swiftbysundell.com/articles/swift-concurrency-multiple-tasks-in-parallel/