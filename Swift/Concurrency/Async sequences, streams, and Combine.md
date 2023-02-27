# Async sequences, streams, and Combine

- `for`루프를 이용하여 스위프트 표준 컬렉션을 반복할 때 반복 코드에 전달할 요소를 결정하는 두 가지 핵심 요소가 있습니다. 바로 sequence와 iterator입니다.
- 예를 들어, 스위프트의 표준 `Array`타입은 sequence이고, iterator 타입으로 `IndexingIterator`를 사용합니다.

- 스위프트 코드를 작성할 때 시퀀스와 직접적으로 상호 작용하는 경우가 아주 많지만 `for` 루프를 사용할 때마다 언어가 해당 인스턴스를 작동으로 관리하기 때문에 iterator를 직접 처리할 필요는 거의 없습니다.

- 반복이 전용 타입에 의해 명시적으로 제어된다는 것은 아주 강력합니다. 왜냐하면 `Array`, `Dictionary`, `Set`과 같은 기본으로 제공되는 타입을 반복할 때 똑같은 방식으로 반복할 수 있는 완전한 커스텀 sequence를 작성할 수 있기 때문이죠.

- 이번 글에서 이 시스템이 작동하는 방식과 스위프트 동시성 세계에서는 어떻게 확장되는지(시간이 지남에 따라 값을 제공하는 완전히 비동기로 작동하는 sequence와 stream을 생성하는 방법을) 살펴보겠습니다.

<br/>

## Sequences and iterators

- URL들로부터 데이터를 다운로드하는 커스텀 시퀀스를 만들고 싶어한다고 가정해 봅시다.

- 이를 위해선 `Sequence`프로토콜을 따르고 필요한 `makeIterator`메서드를 사용하여 iterator를 생성하는 커스텀 타입을 구현해야 합니다:

  <img width="812" alt="CleanShot 2023-02-27 at 11 27 45@2x" src="https://user-images.githubusercontent.com/42647277/221458216-cc02ffb7-c33d-456b-b84a-add7b607ceff.png">

- 그리고 시스템에서 새 요소를 요청할 때마다 `index`속성을 증가시켜 반복을 수행하는 `RemoteDataIterator`타입을 구현합니다:

  <img width="806" alt="CleanShot 2023-02-27 at 11 31 38@2x" src="https://user-images.githubusercontent.com/42647277/221459458-d22be5e7-26bb-4e28-82ca-1ebecd7afdb9.png">

  > 새로운 `for`루프가 시작될 때마다 시스템이 (위의 시퀀스의 `makeIterator` 메서드 사용하여)새로운 iterator를 자동으로 생성하기 때문에 다중, 동시 iteration에 대해 걱정할 필요가 없습니다.

- 위의 코드를 사용하면 이제 배열을 반복할 때 사용하는 것과 똑같은 구문을 사용하여 다운로드한 모든 데이터를 반복할 수 있습니다:

  <img width="806" alt="CleanShot 2023-02-27 at 11 37 17@2x" src="https://user-images.githubusercontent.com/42647277/221460290-6ad0f8c0-9921-460e-a42b-a02a0fd14d43.png">

- 아주 멋진 코드지만, 데이터를 동기적으로 다운로드하는 것은 그닥 좋은 아이디어는 아닙니다. 그렇게 하게되면 모든 다운로드가 완료될 때까지 현재 스레드를 완전히 차단하게 됩니다.
- 이제 위의 코드를 비동기 시퀀스로 바꾸는 방법을 살펴봅시다.

<br/>

## Asynchronous iterations

- 스위프트 5.5의 새로운 동시성 시스템은 동기 시퀀스와 거의 동일한 방식으로 정의되는 비동기 시퀀스 및 iterator개념을 도입합니다.

- 따라서 `RemoteDataSequence`를 비동기적으로 만들려면 `AsnycSequence`를 채택하고 `makeAsyncIterator`메서드를 구현하기만 하면 됩니다:

  <img width="811" alt="CleanShot 2023-02-27 at 11 46 35@2x" src="https://user-images.githubusercontent.com/42647277/221461357-c40e8250-ced8-4644-8249-b7b59d0ef52c.png">

  > 컴파일러가 시퀀스의 `Element`타입을 유추할 수 있어야 하지만 XCode13.0에서는 그렇지 않아도 되는 것 같으므로 위 `typealias`는 꼭 필요한 구문은 아닙니다.

- `async`와 `throws`키워드가 추가된 `next`메서드(데이터 다운로드가 실패할 때 에러를 발생시키도록 하기 위해 throw포함)를 포함하는 비동기식 `RemoteDataIterator`를 만들어 봅시다.

- 그리고 기본으로 제공되는 `URLSession` 네트워크 API를 이용하여 데이터를 완전히 비동기식으로 다운로드할 수 있습니다:

  <img width="808" alt="CleanShot 2023-02-27 at 11 54 16@2x" src="https://user-images.githubusercontent.com/42647277/221462261-989b9cb8-1fda-4cd2-b8ed-a0bcf01e3974.png">

- 위의 변경 사항을 적용하면 `RemoteDataSequence`는 이제 완전히 비동기적인 시퀀스로 전환되어 요소를 반복할 때 데이터가 백그라운드에서 다운로드되기 때문에 await을 사용해야 합니다. 또 준비가 되면 `for`루프로 전달됩니다:

  <img width="812" alt="CleanShot 2023-02-27 at 11 58 01@2x" src="https://user-images.githubusercontent.com/42647277/221462713-9769f5a1-ada0-4536-8ef3-0e02a66680c5.png">

- 비동기적인 iterator가 에러를 던지는 것은 굉장히 강력합니다. 에러가 발생한 경우 이러한 에러를 수동으로 추적할 필요 없이 자동으로 `for`루프를 종료할 수 있기 때문입니다.
- 물론 모든 비동기 시퀀스가 던질 수 있어야 한다는 의미는 아닙니다.
- iterator의 `next`메서드를 선언할 때 `throws`키워드를 생략하면 시퀀스가 에러를 던지지 않는 것으로 간주됩니다. 추가로 요소를 반복할 때 `try`키워드를 더 이상 사용할 필요가 없게되죠.

<br/>

## Asynchronous streams

- 비동기로 작동하는 시퀀스를 완전히 커스텀으로 정의할 수 있는 것은 아주 강력하지만, 표준 라이브러리에는 자체 타입을 정의할 필요없이 이러한 시퀀스를 생성할 수 있는 두 가지 stant-alone(독립적인) 타입을 제공합니다.
-  바로 `AsyncStream`과 `AsyncThrowingStream`입니다. 전자는 에러를 던지지 않는 비동기 시퀀스를 생성하게 하고 후자는 에러를 던지는 옵션을 제공합니다.

- URL들을 이용하여 데이터를 다운로드하는 예제로 다시 돌아가서 새로운 커스텀 타입을 정의하는 것 대신에 `AsyncThrowingStream`을 사용하여 똑같은 기능을 어떻게 구현하는지 살펴봅시다.

- 아래와 같이 코드를 작성하면, 다운로드된 모든 데이터를 `yield`하는 비동기 `Task`를 시작하고 맞닥뜨린 에러를 리포팅하면서 `finish`하게 됩니다: 

  <img width="815" alt="CleanShot 2023-02-28 at 00 07 24@2x" src="https://user-images.githubusercontent.com/42647277/221600550-00630d40-9cb3-4f04-a232-07c24b5da24e.png">

- 위의 구현은 완벽하게 작동하지만 `AsyncThrowingStream`이니셜라이저 함수를 사용하면 더 단순하게 만들 수 있습니다.

- 앞서 말한 이니셜라이저는 이미 `async`로 표된 클로저를 제공하며, 그 안에서 스트림의 다음 요소를 반환하는데 집중하도록 구현되어 있습니다:

  <img width="808" alt="CleanShot 2023-02-28 at 00 09 56@2x" src="https://user-images.githubusercontent.com/42647277/221601202-705106d7-cac0-4f12-bf80-8237e6d672ce.png">

  > 위에서 우리는 스트림 클로저 내에서 지역 변수인 `index`를 캡쳐하여 반복 상태를 추적하는 데 사용할 수 있습니다. 더 자세한 내용은 [“Swift’s closure capturing mechanics”](https://www.swiftbysundell.com/articles/swifts-closure-capturing-mechanics)을 확인하세요. 





- 위의 두 가지 구현 중 하나를 사용하면 이전에 커스텀으로 구현했던 `AsyncSequence`를 반복했던 것처럼 새로운 비동기 스트림을 반복할 수 있습니다:

  <img width="806" alt="CleanShot 2023-02-28 at 00 15 20@2x" src="https://user-images.githubusercontent.com/42647277/221602658-76dee508-8d07-40d3-8218-cc99963c61a1.png">

- 따라서 `Array`가 동기적으로 동작하는 `Sequence`프로토콜의 구체적인 구현인 것처럼, `AsyncStream`과 `AsyncThrowingStream`은 `AsyncSequence`프로토콜의 구체적인 구현으로 볼 수 있습니다.
- 대부분의 상황에서 스트림을 사용하는 것이 아마 가장 간단한 구현일 것입니다. 하지만 주어진 반복에 대한 완전한 제어를 하고싶다면 커스텀 `AsyncSequence`를 만드는 것이 좋겠죠.

<br/>

## How does all of this relate of Combine?

- 만약 애플의 Combine 프레임워크로 작업해봤다면, 시간에 따라 값을 방출하고 관찰할 수 있다는 점을 공통적으로 가진 새로운 비동기 시퀀스 API와 Combine이 어떤 관련이 있는지 궁금할 것입니다.

- 이것은 이미 이전에 WWDC 아티클인 [“What Swift’s new concurrency features might mean for the future of Combine”](https://wwdcbysundell.com/2021/the-future-of-combine)에서 한번 다뤘었고, 저자가 보는 방식은 Combine은 fully-featured reactive programming framework라는 것이고, 반면에 새로운 비동기 시퀀스 시스템은 모든 종류의 비동기 시퀀스를 구성하기 위한 좀 더 low-level수준의 API를 제공한다는 것입니다.

- 하지만 좋은 소식은 이제 Combine이 `AsyncSequence`와 완벽하게 호환되어 모든 `Publisher`를 이러한 비동기 시퀀스로 전환할 수 있다는 것입니다.

- 예를 들어, 위에서 구현한 데이터 다운로드 기능의 Combine버전은 다음과 같습니다:

  <img width="810" alt="CleanShot 2023-02-28 at 00 33 23@2x" src="https://user-images.githubusercontent.com/42647277/221607532-4e656bf9-ccbe-4985-8187-7be65a799280.png">

- 그런 다음 위의 함수가 반환하는 `AnyPublisher`를 `AsyncSequence`로 변환하려면 `values`프로퍼티에 접근하기만 하면 됩니다. 그러면 나머지는 시스템이 처리합니다:

  <img width="808" alt="CleanShot 2023-02-28 at 00 35 18@2x" src="https://user-images.githubusercontent.com/42647277/221608058-67b310dc-b5b8-4113-b1af-3f52f47bb223.png">

- 아주 깔끔하네요. 위의 API는 스위프트의 새로운 동시성 시스템을 채택할 때에도 해당 코드를 계속 사용할 수 있게 해주기 때문에 기존 Combine을 코드 베이스로 있는 곳에서는 아주 유용합니다.

- `async`로 표시된 API를 Combine 기반의 코드 내에서 사용하려면 [“Calling async functions within a Combine pipeline”](https://www.swiftbysundell.com/articles/calling-async-functions-within-a-combine-pipeline)를 확인하세요.

<br/>

## Conclusion

- 이 글이 스위프트의 새로운 `AsyncSequence`와 `AsyncStream` API가 어떻게 작동하는지, 다양한 비동기 시퀀스를 구현하는 데 어떻게 사용될 수 있는지, 이러한 새로운 API가 Combine과 어떻게 관련되는지에 대해 통찰력을 제공했기를 바랍니다.

<br/>

## 출처

- https://www.swiftbysundell.com/articles/async-sequences-streams-and-combine/