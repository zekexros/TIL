# Building async and concurrent versions of forEach and map

- 생각해보면 항상 우리가 작성하는 코드들은 본질적으로 데이터의 변환들로 구성됩니다.
- Model에 해당되는 데이터, 네트워크 response 혹은 유저의 입력이나 다른 이벤트들을 하나의 형태 혹은 형식으로 가져온 후 이를 이용하여 어떠한 로직을 실행하고 궁극적으로 어떠한 형태의 결과 데이터를 저장, 전송 혹은 유저에게 보여줍니다.



- 이번 글에서는 스위프트에 내장된 동시성 시스템을 이용하여  `forEach`와 `map`과 같은 기능들에 대해 데이터 변환을 실행하는 방법을 살펴보겠습니다.

<br/>

## Synchronous transformations

- 사용자가 특정 영화를 즐겨찾기로 표시할 수 있는 `MovieListViewController`를 만드는 것을 예제로 살펴보겠습니다.

- 뷰 컨트롤러는 표준 라이브러리의 `forEach`API를 사용하여 사용자가 선택한 인덱스 경로를 반복한 다음 `FavoritesManager`를 호출하여 해당 인덱스 경로에 해당하는 영화를 즐겨찾기로 표시합니다:

  <img width="809" alt="CleanShot 2023-03-02 at 15 49 15@2x" src="https://user-images.githubusercontent.com/42647277/222352457-c285705a-39f4-4e77-ab8c-3ebafb6accce.png">



- 영화를 즐겨찾기로 표시하는 것 외에도 `FavoritesManager`는 사용자가 이전에 즐겨찾기에 추가한 영화들을 불러올 수 있는 메서드를 갖고있습니다.

- 이것은 내장된 `map`메서드를 사용하여 `Database`를 검색하여 동영상 Set를 실제 모델로 변환합니다:

  <img width="808" alt="CleanShot 2023-03-02 at 15 52 31@2x" src="https://user-images.githubusercontent.com/42647277/222353096-320dbdec-27ec-43d7-924c-679702ecc4a2.png">



- 아주 좋습니다. 이제 스위프트의 새로운 동시성 시스템으로 위의 코드를 비동기, non-block방식으로 어떻게 바꾸는지 살펴봅시다.

<br/>

## Going asynchronous

- `FavoritesManager`와 `Database`타입이 비동기적으로 작업을 수행하도록 만들어봅시다:

  <img width="805" alt="CleanShot 2023-03-02 at 16 04 39@2x" src="https://user-images.githubusercontent.com/42647277/222355404-228d7560-9f2f-4182-961b-bac61b5bc0a5.png">

- 하지만 위의 코드는 컴파일 되지 않습니다. 스위프트 표준 라이브러리인 `map`은 (아직) `async`클로저를 지원하지 않기 때문이죠.
- 스위프트의 extension기능에 감사하며 이것을 이용하여 구현할 수 있습니다.



- 스위프트에서는 다양한 컬렉션(`forEach`, `map`, `flatMap`및 등등 같은 API)에서 일반적으로 수행하는 대부분의 작업은 `Sequence`프로토콜 확장을 사용하여 구현됩니다. 이 프로토콜은 기본으로 내장된 컬렉션(`Array`, `Set`, `Dictionary`)이 채택하는 프로토콜 입니다. 스위프트의 `for`루프를 구동하는데 사용되는 프로토콜이기도 하죠.



- 따라서 `Sequence`프로토콜을 확장하여 `map`의 비동기 버전을 만들어 봅시다:

  <img width="807" alt="CleanShot 2023-03-02 at 16 13 20@2x" src="https://user-images.githubusercontent.com/42647277/222357054-8cd07bdd-1145-4b4e-ba02-6741899db9c2.png">

  > 위의 메서드가 `rethrows`키워드로 표시된 것에 유의하세요. 이것은 스위프트 컴파일러에게 클로저가 throw되는 경우에만 새 메서드가 throw하는 것으로 처리하도록 지시합니다. 이는 클로저가 throw되지 않는 경우에도 항상 try로 새 메서드를 호출할 필요 없이 오직 필요할 때 에러를 throw할 수 있는 유연성을 제공합니다.



- 위의 코드를 사용하여 다시 되돌아가서 이제는 `loadFavorites`메서드의 `map`을 `asyncMap`으로 대체할 수 있으며, 새로운 비동기로 구현된  `FavoritesManager`구현이 성공적으로 컴파일되고 드디어 사용할 준비를 마쳤습니다:

  <img width="809" alt="CleanShot 2023-03-02 at 16 25 08@2x" src="https://user-images.githubusercontent.com/42647277/222359388-391b8599-c006-4df1-831d-f2200d9a6480.png">



- 다음으로 `MovieListViewController`내에서  `FavoritesManager`가 사용자가 선택한 영화를 즐겨찾기로 표시하는 동안 UI가 완전히 응답할 수 있도록  `forEach`반복을 비동기로 전환해 봅시다.

- 그렇게 하려면 비동기 버전의 `forEach`이 필요하므로 `asyncMap`을 만들었던 방법과 유사한 방식으로 구현해 봅시다:

  <img width="805" alt="CleanShot 2023-03-02 at 16 41 46@2x" src="https://user-images.githubusercontent.com/42647277/222363024-3ae3c3b7-9848-4048-b52f-84fb13d81275.png">

- 그러나 뷰 컨트롤러 내에서 새로운 `asyncForEach`메서드를 사용하기 전에 한 가지 고려해야 할 것이 있습니다. 바로 사용자가 선택한 영화를 즐겨찾기로 표시하는 호출이 이제 비동기적으로 수행된다는 것입니다. 즉, 반복이 수행되는 동안 변경이 될 수 있다는 것에 유의해야 합니다.



- 따라서 이러한 상황에서의 버그나 충돌이 발생하지 않도록 `asyncForEach`를 호출할 때 `movie`배열을 값으로 캡쳐하여 전체를 반복하는 동안 동일한 배열에서 계속 작동하도록 합시다:

  <img width="807" alt="CleanShot 2023-03-02 at 16 46 00@2x" src="https://user-images.githubusercontent.com/42647277/222363972-58f9cc4a-afee-45ee-b914-634543711f42.png">

- 이러한 변경 사항을 적용하여 이제 많은 양의 데이터 변환 작업을 비동기로 성공적으로 전환하였습니다. 이제 해당 호출이 완료되는 데 걸리는 시간에 관계없이 UI가 응답성을 유지할 수 있습니다.

<br/>

## Concurrency

- 위 작업의 속도를 높이기 위해 할 수 있는 일이 하나 더 있습니다. 그것은 바로 순서대로 실행하는게 아닌 병렬로 실행되도록 하는 것입니다.
- 위 코드는 비동기로 작동하게 구현하였지만 `asyncForEach`와 `asyncMap`은 항상 다음 작업을 계속하기 전에 결과를 기다립니다. 즉, 순차적으로 실행됩니다.



- 모든 작업을 병렬로 실행하는 방식으로 `forEach`를 수행하기 위해서 기본으로 제공되는 `TaskGroup` API를 활용하여 새로운 방식을 보여드리겠습니다:

  <img width="808" alt="CleanShot 2023-03-02 at 22 07 25@2x" src="https://user-images.githubusercontent.com/42647277/222437242-7bb8262b-d745-45b2-b1f8-fbee214757cc.png">

  > `TaskGroup`에 대한 자세한 내용은 [“Using Swift’s concurrency system to run multiple tasks in parallel”](https://www.swiftbysundell.com/articles/swift-concurrency-multiple-tasks-in-parallel)를 확인하세요.



- 여러개의 동시성 작업과 스레드를 분산하는 작업에는 오버헤드와 딜레이가 수반된다는 점 때문에 <u>모든</u> `forEach`호출을 동시에 수행하고 싶지 않을 것입니다.
- 그러나 `MovieListViewController`내에서 영화를 즐겨찾기로 표시하는 경우 사용자가 많은 수의 영화를 선택하면 속도가 향상된 것을 체감할 수 있으며 이제 동시성을 이용하여 반복문을 수행할 수 있습니다.



- 따라서 이전의  `asyncForEach`를 새로운 `concurrentForEach`메서드로 바꿔봅시다:

  <img width="809" alt="CleanShot 2023-03-02 at 22 21 42@2x" src="https://user-images.githubusercontent.com/42647277/222440248-a2d7135a-141f-41d4-8a54-0255c211672c.png">

  > 클로저 내에 `FavoritesManager`에 대한 참조를 캡쳐하는 것에 유의하세요. 모든 `self`와 관련된 캡처를 명시적으로 만들어야 하는 탈출 클로저를 처리하고 있기 때문입니다. 따라서 `self`를 캡쳐하는 대신, 작업을 수행하는데에 필요한 실제 객체만 캡쳐합니다.



- 위의 변경 사항을 적용하면 이제 즐겨찾기 표시 로직이 여러 영화에서 병렬로 작동하여 상당항 속도 향상을 제공합니다. 물론 `asyncForEach`를 사용할지 `concurrentForEach`를 사용할지 여부는 각각의 상황에서 실제 성능에 어떠한 영향을 미치는지를 기반으로 결정해야 합니다.



- 주어진 작업들을 동시에 실행하기 전에 고려해야 할 또 다른 중요한 사항은 데이터가 동시 접근을 지원하는지에 대한 여부입니다. 다음 글에서 스위프트의 actor타입을 사용하여 변경 가능한 상태에 대한 직렬로 접근하는 방법에 대해 자세히 알아볼 예정입니다.



- `FavoritesManager`내의 mapping작업으로 다시 이동하여 반복을 동시에 수행할 수 있는 방법을 살펴보겠습니다. 이를 위해 `concurrentMap`이라는 새로운 버전의 `map`을 만들어 봅시다.

  

- 그러나 `concurrentForEach`와는 다르게 새롭게 만드는 메서드는 `TaskGroup`을 사용하지 않습니다. 완료되는 순서를 보장하지 않기 때문에 즉, 입력과는 순서가 다르게 출력되어 결과가 나올 가능성이 있기 때문에 사용하지 않습니다.

- 그래서 대신 각 요소에 대한 비동기 `Task`를 만들어서 `asyncMap`메서드를 사용하여 각 작업의 결과를 순서대로 기다립니다:

  <img width="809" alt="CleanShot 2023-03-02 at 22 51 49@2x" src="https://user-images.githubusercontent.com/42647277/222447372-bf309eab-b0dd-459a-800c-09917b0b43c4.png">

  > 위와 다른 접근 방법은 출력 순서를 추적하는 dictionary와 `TaskGroup`을 함께 사용하는 것입니다. 그러나 이러한 구현은 더 복잡할 뿐만 아니라 현재 구현에 비해 성능도 크게 향상되지 않습니다.

- 새로운 `concurrentMap`메서드를 사용하여 `FavoritesManager`가 완전히 동시적인 방법으로 모든 즐겨찾기를 불러올 수 있습니다.

- 이는 사용자가 즐겨찾기 목록에 점점 더 많은 영화를 추가함에 따라 효과는 커집니다:

  <img width="812" alt="CleanShot 2023-03-02 at 22 55 20@2x" src="https://user-images.githubusercontent.com/42647277/222448482-dee17a04-26ca-48f5-8128-6bb79f4473cb.png">





- 이제 코드를 비동기로 변경했을 뿐만 아니라 동시에 실행되도록 구현하였습니다.
- 그 과정에서 완전히 재사용 가능한 4개의 `Sequence`API를 만들었습니다.
  - `asyncForEach`
  - `concurrentForEach`
  - `asyncMap`
  - `concurrentMap`

<br/>

## Conclusion

- 이 글이 스위프트의 새로운 동시성 시스템을 사용하여 스위프트의 반복을 비동기 또는 동시성을 갖는 작업으로 전환할 수 있는 방법에 대해 통찰력을 제공했기를 바랍니다.
- 동시성을 사용하는 시기와 사용하면 안될 때를 구분하고 이러한 결정은 실제 성능에 기반하여 시도하는 것이 항상 중요합니다.
- 결국 완전한 동기적으로 작동하는 코드가 항상 구현 및 유지하기 쉽고 디버깅하기 편리합니다.

<br/>

## 출처

- https://www.swiftbysundell.com/articles/async-and-concurrent-forEach-and-map/