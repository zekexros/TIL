# How the SwiftUI View Lifecycle and Identity work

## The benefits of a data-driven framework 

- 더 이상 거대한 뷰 컨트롤러들이 필요 없어졌다.
- 뷰를 업데이트하기 위해 이벤트의 로직을 정의할 필요가 없다.
- 인터페이스는 데이터와 결합되어 코드 라인이 줄어들고 가독성이 향상된다.

<br/>

## The challenges of using SwiftUI

- 새로운 프레임워크에는 항상 장단점이 있습니다.
- 위의 내용으로 장점만 나열했는데 단점으로는 새로나온 프레임워크인 만큼 학습이 필요하고 동일한 코드에 대해서 iOS14와 iOS15가 다르게 작동하는 경우도 있어 호환성에 문제가 있을수도 있습니다.

<br/>

## View in SwiftUI

- SwiftUI에는 3가지 컨셉이 있습니다.
  - View as a function of state
  - View's identity
  - View's lifetime
- 그리고 state가 바뀌면 뷰가 업데이트 된다는 것을 알게되었다면 이어지는 질문들도 자연스럽게 궁금해집니다.
  - How exactly does the data get updated?
  - How does the view understand what exactly needs to change?
  - Does it create a new view every time a small piece of data changes?
  - How efficient and costly are the data updates?
- 이어지는 내용들을 살펴보고 위의 질문에 대한 해답을 얻어봅시다.

<br/>

## About SwiftUI’s view hierarchy

- 스유의 메인 UI요소는 View입니다.
- View의 정의 및 state를 효율적으로 작성 여부에 따라 앱의 퍼포먼스와 비쥬얼의 퀄리티가 달라집니다.

```swift
struct FruitsView: View {
    var body: some View {
        BananaView()
    }
}
 
struct BananaView: View {
    var body: some View {
        Text("I am banana!")
            .padding()
    }
}
```

- 여기선 FruitsView는 표시할 내용을 알아야 하기에 body를 통해 BananaView를 요청하고 이어 BananaView는 body에 Text를 요청합니다.
- 그럼 아래와 같은 재귀 호출의 모습이 나타납니다.

![Figure 4: A sequence of View’s body calls](https://doordash.engineering/wp-content/uploads/2022/05/view-view-11-295x1024.jpg)

- SwiftUI는 좋은 성능을 내기위해 그것을 짧게 자르고 어떻게든 재귀를 끊어내야 합니다.
- 위의 경우에는 스유가 body에 있는 Text를 요청할 때 재귀가 종료됩니다.
- Text는 스유의 기본 요소이므로 또다른 body에 대해 요청하지 않고 그려낼 수 있습니다. 이것은 Never타입으로 달성됩니다.
  ```swift
  extension Text : View {
      public typealias Body = Never
  }
  
  extension Never : View {
      public typealias Body = Never
      public var body: Never { get }
  }
  
  ```

- 스유가 기본 타입을 특별한 방식으로 처리하기 때문에 아래의 그림처럼 기본 타입에 도달하면 재귀가 종료됩니다.

  ![Figure 5: The end of recursion when Never type is reached ](https://doordash.engineering/wp-content/uploads/2022/05/view-never-11-416x1024.jpg)

- 기본 타입들은 모든 뷰 계층구조의 기본이 됩니다. 다른 기본 타입들은 아래와 같습니다.
  - Text
  - Image
  - Spacer
  - ZStack
  - VStack
  - HStack
  - List
  - Etc.

<br/>

## State Management System

- 스유의 state를 다루는 다양한 프로퍼티 래퍼들 중 environmentObject만 다루겠습니다(다른 글에서 이미 다뤘음).

### @EnvironmentObject

- 이것은 객체 자체를 생성하거나 할당하지 않습니다.

- 대신 뷰 계층 구조의 environment를 모니터링하는 메커니즘을 제공합니다.

  ![Figure 6: An example of a view hierarchy ](https://doordash.engineering/wp-content/uploads/2022/05/parent-view-11-1024x921.jpg)

- View C와 D는 데이터에 의존해야하는 상황일 때, View A와 B는 데이터가 필요하지 않은데도 불구하고 지속적으로 데이터를 주입받아 하위 뷰에 전달해야 합니다.
- 부모 뷰중 하나에서 `environmentObject(_:) `modifier를 사용하면 해당 뷰의 모든 하위 뷰들에서 observed object에 접근할 수 있어 불필요한 보일러플레이트 코드를 없앨 수 있습니다.
- 스유의 state관련 여러 프로퍼티 래퍼들은 데이터를 업데이트하고 view에 반영하는데 사용하는 도구입니다.
- 데이터 흐름에 대한 작은 변화로 인해 여러 View들의 body에 대한 계산을 발생시킬 수 있습니다.
- 이것은 잠재적으로 성능에 영향을 미칠 수 있지만 스유는 변경사항을 감지하고 업데이트의 영향을 받은 view의 일부분만 다시 그리도록 똑똑하게 만들어졌습니다.
- redrawing은 데이터 및 관련된 View에 대한 dependency graph를 빌드하고 분석하기 위해 스유에서 사용하는 내부 요소인  `AttributeGraph`의 도움을 받습니다.

<br/>

## Identity of a view

- UIKit에서 View들은 클래스이고 클래스들은 View들을 식별하는 포인터가 있습니다.
- 그러나 스유에서의 View들은 구조체입니다. 포인터가 존재하지 않죠.
- 효율성 및 최적화를 위해 스유는 View가 같은지 다른지 알고있어야 합니다.
- View의 값 중 일부가 변경된 후 올바른 전환과 렌더링을 하기 위해 프레임워크가 View를 식별하는 것도 중요합니다.
- View의 <u>identify</u>는 스유 렌더링 마법에 빛을 가져다주는 개념입니다.
- 앱 전체에 수천개의 변경사항이 있을 수 있고 body프로퍼티가 재계산이 계속해서 이뤄질수도 있습니다.
- 그러나 영향을 받는 View는 항상 전체를 리렌더링하는 것은 아닙니다.
- View의 identity가 이것을 이해하기 위한 핵심입니다.
- 스유의 View를 식별하는 방법에는 2가지가 있습니다.
  - <u>explicit identity</u>
  - <u>structural identity</u>

<br/>

## Explicit identity



