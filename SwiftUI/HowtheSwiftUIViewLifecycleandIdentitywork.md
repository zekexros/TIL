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

- 커스텀 혹은 데이터 기반 식별자를 이용하여 View를 식별할 수 있습니다.

- UIKit에서 사용되는 포인터 식별자는 View를 식별하는데 사용되기 때문에 explicit identity의 예시입니다.

- For-each루프에서 View를 반복하면서 예제를 본적이 있을겁니다.

- explicit identity는 식별자를 직접 사용하여 제공할 수 있습니다.

  - .id(~~~)를 사용하여 제공

- View의 identity를 Hashable한 값에 바인딩합니다.

  ```swift
  extension View {
          @inlinable public func id<ID>(_ id: ID) -> some View where ID : Hashable
  }
  ```

<br/>

```swift
struct Fruit {
    let name: String
    let color: Color
}
```

- 이름과 색을 가진 Fruit을 정의해봅시다.

```swift
struct FruitListView: View {
    let fruits = [Fruit(name: "Banana", color: .yellow),
                      Fruit(name: "Cherry", color: .red)]
    
    var body: some View {
        ScrollView {
            ForEach(fruits) { fruit in
                FruitView(fruit: fruit)
            }
        }
    }
}
 
struct FruitView: View {
    let fruit: Fruit
    
    var body: some View {
        Text("\(fruit.name)!")
            .foregroundColor(fruit.color)
            .padding()
    }
}
```

- ScrollView에 ForEach를 사용하여 뷰를 구성합니다.
- 그러나 위 코드는 `Referencing initializer 'init(_:content:)' on 'ForEach' requires that 'Fruit' conform to 'Identifiable'` 와 같은 컴파일 에러를 발생시킬 것입니다.
- 이 문제는 `Fruit`에 `Identifiable`프로토콜을 구현하거나 keypath를 제공하면 고쳐질 것입니다.
- 어느쪽이든 FruitView가 가져야하는 explicit identity에 대해 스유가 알게될 것입니다.

<br/>

```swift
struct FruitListView: View {
    let fruits = [Fruit(name: "Banana", color: .yellow),
                      Fruit(name: "Cherry", color: .red)]
    
    var body: some View {
        ScrollView {
            ForEach(fruits, id: \.name) { fruit in
                FruitView(fruit: fruit)
            }
        }
    }
}
```

- 위의 새로운 코드는 정상적으로 컴파일되고 `FruitView`는 name에 의해 식별될 것입니다. 왜냐하면 fruit의 name은 유니크하도록 설계되었기 때문이죠.

<br/>

```swift
struct ContentView: View {
    let headerID = "header"
    
    let fruits = [Fruit(name: "Banana", color: .yellow),
                      Fruit(name: "Cherry", color: .red)]
    
    var body: some View {
        ScrollView {
            ScrollViewReader { proxy in
                Text("Fruits")
                    .id(headerID)
 
                ForEach(fruits, id: \.name) { fruit in
                    FruitView(fruit: fruit)
                }
 
                Button("Scroll to top") {
                    proxy.scrollTo(headerID)
                }
            }
        }
    }
}
```

- explicit identity가 사용되는 또다른 케이스는 스크롤 뷰의 섹션 중 하나로 수동적으로 스크롤을 수행하는 기능입니다.
- 위의 예제는 버튼을 탭하면 view를 탑으로 스크롤하게 됩니다. `.id()` 확장자는 view에 커스텀 식별자를 제공하여 explicit identity를 제공하는데 사용됩니다.

<br/>

## Structural identity

- 모든 스유의 View에는 식별자(identity)가 있어야 합니다.
- 만약 View가 explicit identity가 없다면 대신하여 structural identity가 있을겁니다.
- structural identity는 View가 뷰 계층 구조에서 타입과 위치를 사용하여 식별되는 경우입니다.
- 스유는 뷰 계층 구조를 사용하여 View에 대한 implicit identity를 생성합니다.

<br/>

```swift
struct ContentView: View {
    @State var isRounded: Bool = false
    
    var body: some View {
        if isRounded {
            PizzaView()
                .cornerRadius(25)
        } else {
            PizzaView()
                .cornerRadius(0)
        }
 
        PizzaView()
            .cornerRadius(isRounded ? 25 : 0)
        
        Toggle("Round", isOn: $isRounded.animation())
            .fixedSize()
    }
}
```

- 위의 예시에서는 PizzaView에 대해 corner radius에 애니메이션을 주는 2가지 접근 방식이 있습니다.

- 첫 번째는 bool값에 따라 서로 완전히 다른 View를 만듭니다. 실제로 스유는 뒤에서 `ConditionalContent view` 의 인스턴스를 생성합니다.

- `ConditionalContent`는 조건에 따라 다른 View를 표시하는 역할을 합니다.

- 그리고 pizza view들은 조건절이 사용되었으므로 서로 다른 id를 갖습니다.

- 이 경우 스유는 토글이 변경되면 View를 다시 그리고 아래 애니메이션처럼 fade in/out을 적용합니다.

- 이것은 동일한 PizzaView가 아니라는 것을 이해하는 것이 중요합니다.

- 이들은 두 개의 다른 View이며 각각 자기만의 structural identity를 갖습니다.

  ![image1](https://user-images.githubusercontent.com/42647277/218096885-22f683dd-b54e-4e8f-97ed-28b2db547202.gif)

<br/>

```swift
PizzaView()
            .cornerRadius(isRounded ? 25 : 0)
```

- PizzaView는 조건절이 아닌 단순히 view modifier를 통해서도 구현이 가능합니다.
- 이것은 structural identity가 동일하게 유지됩니다. 그리고 스유는 fade in/out 전환을 적용하지  않습니다.
- 속성값만 다를 뿐 같은 뷰이기 때문에 corner radius변경을 애니메이션으로 표시합니다.
- 이 경우에는 structural identity가 변경되지 않습니다. Apple은 if/else문이나 switch같은 조건절을 사용하는 대신 modifier안에 조건을 넣어 identity를 보존할 것을 권장합니다.
- 앱을 최적화하고 버그가 없도록 만드는 것의 핵심은 바로 structural identity와 이에대한 이해입니다.

```swift
// Please don't use this:
extension View {
    @ViewBuilder
    func applyIf<M: View>(condition: Bool, transform: (Self) -> M) -> some View {
        if condition {
            transform(self)
        } else {
            self
        }
    }
}
```

- 위와 같이 왜 conditional modifier를 사용하는 것이 좋지 않은지를 알려줍니다.

- 더 나은 성능을 내기위해 염두해야할 몇 가지 사항이 있습니다.
  - View의 identity를 유지합니다. 이를 위해 조건문을 사용하지 마세요.
  - 명시적으로 제공된 경우 View에 안정적인 identifier를 사용하세요.
  - 가능한 AnyView사용을 피하세요.



## 출처

- https://doordash.engineering/2022/05/31/how-the-swiftui-view-lifecycle-and-identity-work/
- https://www.objc.io/blog/2021/08/24/conditional-view-modifiers/
