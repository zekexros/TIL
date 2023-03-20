# Throwing and asynchronous Swift properties

- 스위프트 5.5에서 [“effectful read-only properties”](https://github.com/apple/swift-evolution/blob/main/proposals/0310-effectful-readonly-properties.md)라고 불리는 새로운 컨셉을 소개하였습니다.
- 이는 기본적으로 연산 프로퍼티가 값을 계산할 때 에러 및 비동기 작업과 같은 제어 흐름 메커니즘을 활용할 수 있음을 의미합니다.

<br/>

## Throwing properties

- 이제 스위프트의 표준 에러 처리 메커니즘을 사용하여 연산 프로퍼티가 에러를 어떻게 던지는지 알아봅시다.

- 예를 들어, 현재 기본적으로 제공되는 `Result`타입의 `get`메서드를 사용하여 결과의 래핑된 값을 추출하거나 포함되어있는 에러를 발생한다고 가정합시다:

  <img width="810" alt="CleanShot 2023-03-20 at 13 52 00@2x" src="https://user-images.githubusercontent.com/42647277/226249724-3867d1dd-a7ea-4647-a7b8-6bb1219cf7da.png">

- 이 `get`메서드는 실제로 어떤 종류의 작업도 수행하지 않고 대신 `try`키워드를 사용하여 `Result`값의 압축을 풀 수 있기 때문에 이제 아래와 같이 프로퍼티로 선언할 수 있습니다:

  <img width="806" alt="CleanShot 2023-03-20 at 13 54 40@2x" src="https://user-images.githubusercontent.com/42647277/226250054-af19d782-fe1d-4d63-a09e-0495febdb145.png"> 

- 이제 연산 프로퍼티가 에러를 발생시킬 수 있다는 사실이 특정 상황에서 정말 유용하지만 프로퍼티와 함수 간의 의미론적 차이를 염두에 두는 것이 중요합니다.
- 따라서 위의 `value`프로퍼티는 프로퍼티로서 완전히 의미가 있을 수 있지만 많이 에러를 던지는 작업이라면 여전히 함수로 하는게 더 잘 구현될 것입니다.

<br/>

## Asynchronous properties

- 새로운 동시성 시스템과 함께 스위프트 5.5는 연산 프로퍼티를 완전히 비동기로 작동하는 것이 가능해졌습니다.

- 이제 프로퍼티가 `throws`키워드를 사용할 수 있는 방법과 비슷하게 `async`가 달린 모든 프로퍼티는 이제 다른 비동기 API를 자유롭게 호출할 수 있으며 이러한 프로퍼티에 액세스하는 코드는 내재된 비동기 작업이 모두 완료될 때까지 시스템에서 일시 중단됩니다.

- `DatabaseEntity`는 `isSynced`프로퍼티에 접근할 때 싱크되었는지 비동기적으로 부모의 `Database`를 체크합니다:

  <img width="807" alt="CleanShot 2023-03-20 at 14 13 18@2x" src="https://user-images.githubusercontent.com/42647277/226252381-fdb7b873-00ea-4d7d-8b4c-0d05c89ce992.png">

- 비동기 코드의 경우 프로퍼티 기반 API뒤에서 구현한 작업 종류에 대해 훨씬 더 주의를 기울여야 합니다.
- 예를 들어, 우리는 프로퍼티를 연산하는 것의 일부로 네트워크 호출 혹은 오래걸리는 작업과 같은 일들을 하고싶지 않을 것이지만 별도의 디스패치 큐로 들어가는 작업이나 특정 형태의 파일 I/O를 포함하는 작업의 경우 `async`표시된 프로퍼티가 매우 유용할 수 있습니다.

<br/>

## Protocol requirements

- 마지막으로 프로토콜의 일부로서 선언될 때 `effectful proerties`가 어떻게 보이는지 살펴보겠습니다.

- 프로토콜이 요구 사항 목록의 일부로 프로퍼티를 정의할 수 있는 방법과 마찬가지로 프로토콜은 이제 해당 프로퍼티를 비동기로 요구할 수 있으며, 선택적으로 에러를 던질 수 있습니다.

- 예를 들어 위의 `isSynced`프로퍼티를 프로토콜의 일부로 정의한다면 아래와 같습니다:

  <img width="807" alt="CleanShot 2023-03-20 at 14 28 13@2x" src="https://user-images.githubusercontent.com/42647277/226254221-f8a4e9f4-c74a-42a1-9563-7dc179c6239b.png">

- 그런 다음 `Syncable`을 준수하는 타입이 `isSynced` 에 대해 에러를 던지도록 하려면 단순히 `throws`키워드를 추가하면 됩니다:

  <img width="807" alt="CleanShot 2023-03-20 at 14 36 51@2x" src="https://user-images.githubusercontent.com/42647277/226255332-2c8c7d10-5a45-4a17-975c-e38ea677e3f6.png">

- 다른 에러를 던지는 프로토콜의 요구 사항과 마찬가지로 `throws`로 표시된 연산 프로퍼티의 구현은 실제로 에러를 던질 필요는 없으며 필요에 따라 던져도되고 아니어도되는 옵션입니다.

<br/>

## Conclusion

- 스위프트 5.5부터 이제 연산 프로퍼티에 `throws`와 `async` 키워드를 표시하는 방법에 대해 알아보았습니다.

<br/>

## 출처

- https://www.swiftbysundell.com/articles/throwing-async-properties/