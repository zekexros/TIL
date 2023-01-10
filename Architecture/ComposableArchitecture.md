# Composable Architecture에 대한 설명

- State management(상태 관리)
    - 간단한 값 타입들로 어플리케이션의 상태를 관리하는 방법, 상태를 공유하여 화면에서 일어나는 변화(Mutation)를 다른 화면에서 즉시 관측(Observe)하는 방법을 제공합니다.
- Composition(합성)
    - 기능을 여러 개의 독립된 모듈로 추출하는 방법, 이 모듈을 다시 합쳐서 거대한 기능을 작은 컴포넌트의 집합으로 구성하는 방법을 제공합니다.
- Side effects(사이드 이펙트)
    - 어플리케이션 바깥세상과 접촉하는 작업을 테스트할 수 있고 이해하기 쉽게 작성하는 방법을 제공합니다.
- Testing(테스팅)
    - 아키텍처 내부의 기능을 테스트하는 방법뿐만 아니라 여러 파트로 구성된 기능의 통합 테스트를 작성하는 방법, 사이드 이펙트가 어플리케이션에 끼치는 영향에 대해 전체 테스트를 작성하는 방법을 제공합니다. 이 테스트 방식은 여러분의 비즈니스 로직이 예상대로 잘 작동하는지에 대한 강한 보증도 제공합니다.
- Ergonomics(인체 공학)
    - 위의 내용들을 간단한 컨셉을 적용한 단순한 API로 제공합니다.
    
    
    
    <br/>
    
    <br/>

### 기본적인 사용법

TCA를 통해 기능을 만들기 위해선 도메인을 구성하는 몇 가지 타입을 정의해야 합니다.

![https://koenig-media.raywenderlich.com/uploads/2021/06/Raywenderlich-TCA-1-V2.png](https://koenig-media.raywenderlich.com/uploads/2021/06/Raywenderlich-TCA-1-V2.png)

- State
    - 비즈니스 로직을 수행하거나 UI를 그릴 때 필요한 데이터를 설명하는 타입입니다.
- Action
    - 사용자가 하는 행동이나 노티피케이션 등 어플리케이션에서 생길 수 있는 모든 행동을 나타내는 타입입니다(다시 말하지만 유저 액션에만 국한되지 않습니다).
- Environment
    - API 클라이언트나 애널리틱스 클라이언트와 같이 어플리케이션이 필요로 하는 의존성(Dependency)을 가지고 있는 타입입니다.
    - 이를테면, 네트워크, 데이터베이스, 애널리틱스, 파이어 베이스, OAuth SDK 등등이 여기에 속합니다.
- Reducer
    - 어떤 행동(Action)이 주어졌을 때 지금 상태(State)를 다음 상태로 변화시키는 방법을 가지고 있는 함수입니다. 또한 리듀서는 실행할 수 있는 이펙트(Effect, 예시: API 리퀘스트)를 반환해야 하며, 보통은 `Effect` 값을 반환합니다.
- Store
    - 실제로 기능이 작동하는 공간입니다. 우리는 사용자 행동(Action)을 보내서 스토어(Store)는 리듀서(Reducer)와 이펙트(Effect)를 실행할 수 있고, 스토어(Store)에서 일어나는 상태(State) 변화를 관측(observe)해서 UI를 업데이트할 수도 있습니다.
    
    <br/><br/>

### 예시

- 상황
    - 화면에 숫자와 이 숫자를 증가할 수 있는 + 버튼, 감소할 수 있는 - 버튼이 있다고 해보겠습니다. 더 다양한 행동을 위해 탭 하면 API 호출을 해서 숫자에 관한 무작위 사실을 알림창으로 보여주는 버튼도 추가합니다.

    <br/>

- Feature

    - 기능을 구현하기 위해 ReducerProtocol을 준수하여 기능의 도메인과 동작을 수용할 새로운 유형 `Feature`을 만들었습니다.

    ```swift
    import ComposableArchitecture
    
    struct Feature: ReducerProtocol {
    }
    ```

    <br/>

- State

    ```swift
    struct Feature: ReducerProtocol {
      struct State: Equatable {
        var count = 0
        var numberFactAlert: String?
      }
    }
    ```

    - Feature의 State를 정의해야 합니다. State타입이 관리할 구체적인 값들은 아래와 같습니다.

      - 화면에 띄워줄 숫자를 나타내는 count

      - 숫자에 대한 부연설명을 나타내는 numberFactAlert, 하지만 알림창이 없을 때는 nil 값이 필요하므로 옵셔널로 선언

      

      <br/>

- Action

    ```swift
    struct Feature: ReducerProtocol {
      struct State: Equatable { … }
      enum Action: Equatable {
        case factAlertDismissed
        case decrementButtonTapped
        case incrementButtonTapped
        case numberFactButtonTapped
        case numberFactResponse(TaskResult<String>)
      }
    }
    ```

    - Feature의 Action을 정의해야 합니다. Action타입에 속하는 구체적인 case들은 아래와 같습니다.
      - 증가 버튼이나 감소 버튼을 누르는 행동(`incrementButtonTapped`, `decrementButtonTapped`), 무작위 설명을 요청하는 버튼을 누르는 행동(`numberFactButtonTapped`)
      - 알림창을 닫거나(`factAlertDismissed`) 무작위 사실 API 리퀘스트 결과를 받았을 때 발생하는 행동(`numberFactResponse(Result<String, ApiError>)`)

    <br/>

- ~~Environment~~ 

    - ReducerProtocol이 도입됨에 따라 Envrionment를 Feature마다 선언하지 않아도 됩니다(보일러플레이트 코드들이 없어짐ㅎㅎ)
    - 따라서 아래는 ReducerProtocol을 도입하기 전 버전을 사용하고있을 때 참고바랍니다.

    ```swift
    struct AppEnvironment {
      var mainQueue: AnySchedulerOf<DispatchQueue>
      var numberFact: (Int) -> Effect<String, ApiError>
    }
    ```

    - 다음은 화면이 제대로 작동하기 위해 필요한 의존성(Dependency)을 관리하는 환경(Environment) 차례입니다.
    - 숫자에 관한 사실을 가져오는 경우 네트워크 리퀘스트를 요약해서 `Effect`값으로 만드는 작업이 있겠네요. 이 작업의 의존성은 `Int`를 받아서 `Effect<String, ApiError>`를 반환하는 함수가 되겠습니다. 여기서 String은 리퀘스트의 response입니다. 즉 숫자에 대한 부연 설명이죠.
    - 이펙트는 통상적으로 백그라운드 스레드에서 작업을 처리하게 될 것입니다(`URLSession`이 하는 것처럼요). 저희는 이펙트의 값을 메인 큐에서 받을 방법이 필요합니다. 메인 큐 스케줄러를 사용해야 테스트를 작성할 수 있습니다. `AnyScheduler`를 사용해서 프로덕션에선 `DispatchQueue`를 사용하고 테스트 시엔 테스트 스케줄러를 사용해봅시다.

    <br/>

- Reducer

    ```swift
    struct Feature: ReducerProtocol {
      struct State: Equatable { … }
      enum Action: Equatable { … }
      
      func reduce(into state: inout State, action: Action) -> EffectTask<Action> {
        switch action {
          case .factAlertDismissed:
            state.numberFactAlert = nil
            return .none
    
          case .decrementButtonTapped:
            state.count -= 1
            return .none
    
          case .incrementButtonTapped:
            state.count += 1
            return .none
    
          case .numberFactButtonTapped:
            return .task { [count = state.count] in
              await .numberFactResponse(
                TaskResult {
                  String(
                    decoding: try await URLSession.shared
                      .data(from: URL(string: "http://numbersapi.com/\(count)/trivia")!).0,
                    as: UTF8.self
                  )
                }
              )
            }
    
          case let .numberFactResponse(.success(fact)):
            state.numberFactAlert = fact
            return .none
    
          case .numberFactResponse(.failure):
            state.numberFactAlert = "Could not load a number fact :("
            return .none
        }
      }
    }
    ```

    - 실제 로직 및 동작들을 처리하는 reduce메서드를 구현합니다.
    - 현재 상태(State)를 변화시켜서 다음 상태로 만드는 방법에 대한 설명과 어떤 이펙트(Effect)가 실행돼야하는지에 대한 설명이 필요합니다.
    - 만약 어떠한 이펙트도 실행이 필요하지 않은 경우엔 `.none`을 반환하면 됩니다.

    <br/>

- View

    ```swift
    struct FeatureView: View {
      let store: StoreOf<Feature>
    
      var body: some View {
        WithViewStore(self.store, observe: { $0 }) { viewStore in
          VStack {
            HStack {
              Button("−") { viewStore.send(.decrementButtonTapped) }
              Text("\(viewStore.count)")
              Button("+") { viewStore.send(.incrementButtonTapped) }
            }
    
            Button("Number fact") { viewStore.send(.numberFactButtonTapped) }
          }
          .alert(
            item: viewStore.binding(
              get: { $0.numberFactAlert.map(FactAlert.init(title:)) },
              send: .factAlertDismissed
            ),
            content: { Alert(title: Text($0.title)) }
          )
        }
      }
    }
    
    struct FactAlert: Identifiable {
      var title: String
      var id: String { self.title }
    }
    ```

    - 마지막으로 이 기능이 작동될 뷰를 정의합니다. `StoreOf<Feature>`가 있으면 모든 상태 변화를 관측하고 UI를 다시 그릴 수 있으며, store에 사용자 행동을 보내서 상태를 변화시킬 수 있습니다.
    - `.alert` View Modifier가 요구하는 대로 숫자에 관한 사실을 구조체로 한 번 감싸서  `Identifiable` 을 따르게 만들겠습니다.

    <br/>

- 사용

    ```swift
    import ComposableArchitecture
    
    @main
    struct MyApp: App {
      var body: some Scene {
        WindowGroup {
          FeatureView(
            store: Store(
              initialState: Feature.State(),
              reducer: Feature()
            )
          )
        }
      }
    }
    ```

    - View를 다 그렸으니 이제 앱의 진입점으로 가서 store를 구성합시다.
    - 초기 state와 reducer를 구체화하여 앱을 시작합니다.

    <br/><br/>

### 장점

- 단순히 로직을 관측 가능한 객체나 다양한 UI 컴포넌트의 클로저에 흩뿌리는 것보다, 상태 변경을 적용하는 것에 일관된 태도를 가지도록 해줍니다.
- 사이드 이펙트를 간결하게 표현하는 방법도 제공합니다.
- 그리고 추가적인 작업 없이 이펙트가 포함된 로직을 바로 테스트할 수도 있습니다.

<br/>

## 테스트

```swift
@MainActor
func testFeature() async {
  let store = TestStore(
    initialState: Feature.State(),
    reducer: Feature()
  )
}
```

- 테스트를 하기 위해선 테스트에 특화된 기능을 좀 더 가진 `TestStore`를 생성해야 합니다.
- `Store`와 동일한 정보로 생성할 수 있지만 `TestStore`를 쓰는 이유는 action들이 전달될 때 기능이 어떻게 동작하는지 확인할 수 있도록 추가 작업을 합니다.

<br/>

```swift
// Test that tapping on the increment/decrement buttons changes the count
await store.send(.incrementButtonTapped) {
  $0.count = 1
}
await store.send(.decrementButtonTapped) {
  $0.count = 0
}
```

- test store가 생성되면 이를 사용해 사용자 플로우 전체를 넣을 수 있습니다.
- 우리가 예상한대로 state가 변경되었음을 증명하는 단계들을 넣을 수 있습니다.
- 예를 들어 증가 혹은 감소 버튼을 탭하는 사용자 플로우를 시뮬레이션할 수 있습니다.



```swift
await store.send(.numberFactButtonTapped)

await store.receive(.numberFactResponse(.success(???))) {
  $0.numberFactAlert = ???
}
```

- 그리고 만약 한 스텝(액션)에 의해 store내에 데이터를 공급하는 다른 이펙트가 발생된다면 반드시 assert를 돌려줘야 합니다.
- 예를 들어 fact 버튼을 누르는 것을 시뮬레이션하면, 숫자에 관한 fact를 응답받을 것이고 이는 alert를 띄울 것입니다.
- 근데 이 때, <u>어떤</u> fact가 다시 반환될지 어떻게 알 수 있을까??
- 현재까지의 reducer는 실제 세계에 도달하여 API서버를 찌르는 effect를 사용하고 있으며 이말은 우리가 그 동작에 대해 제어할 방법이 없음을 의미합니다.
- 우리는 이 테스트를 작성하기 위해 인터넷 연결과 API 서버의 가용성에 달려 있습니다.

<br/>

```swift
struct Feature: ReducerProtocol {
  let numberFact: (Int) async throws -> String
  …
}
```

- 디바이스에서 앱을 실행할 때 live dependency를 사용할 수 있지만, 테스트 할 땐 mocked dependency를 사용할 수 있도록 reducer에게 전달하는 것이 좋습니다.
- `Feature`에 프로퍼티를 추가하여 이를 수행할 수 있습니다. 

<br/>

```swift
case .numberFactButtonTapped:
  return .task { [count = state.count] in 
    await .numberFactResponse(TaskResult { try await self.numberFact(count) })
  }
```

- 프로퍼티 추가 후 위의 코드처럼`reduce`구현부에서 사용할 수 있습니다.

<br/>

```swift
@main
struct MyApp: App {
  var body: some Scene {
    FeatureView(
      store: Store(
        initialState: Feature.State(),
        reducer: Feature(
          numberFact: { number in
            let (data, _) = try await URLSession.shared
              .data(from: .init(string: "http://numbersapi.com/\(number)")!)
            return String(decoding: data, as: UTF8.self)
          }
        )
      )
    )
  }
}
```

- 그리고 앱의 시작점에서 실제 API서버와 상호작용이 가능한 의존성을 제공할 수 있습니다.

<br/>

```swift
@MainActor
func testFeature() async {
  let store = TestStore(
    initialState: Feature.State(),
    reducer: Feature(
      numberFact: { "\($0) is a good number Brent" }
    )
  )
}
```

- 그러나 테스트에서는 결정되어있고 예측 가능한 fact를 반환하는 mock dependency를 사용할 수 있습니다.

<br/>

```swift
await store.send(.numberFactButtonTapped)

await store.receive(.numberFactResponse(.success("0 is a good number Brent"))) {
  $0.numberFactAlert = "0 is a good number Brent"
}

await store.send(.factAlertDismissed) {
  $0.numberFactAlert = nil
}
```

- 앞서 진행한 약간의 선행 작업들을 통해 이젠 fact버튼을 탭하고 dependency에 의한 응답을 수신하며 alert를 트리거하고, alert를 없애는 것까지 시뮬레이션하여 테스트를 진행할 수 있게 되었습니다.

<br/><br/>

- 앱에서 `numberFact`에 대한 의존성 사용에 대해 인체공학적으로 개선시킬 수 있습니다.
- 시간이 지남에 따라 앱은 많은 기능을 가지도록 발전하며, 이 기능들 중 몇몇은 `numberFact`의 접근을 원할수도 있을텐데 그 때마다 명시적으로 모든 계층을 통해 의존성들을 전달하면 성가시고 귀찮을 수 있습니다.
- 종속성을 라이브러리에 "register"하여 앱의 모든 계층에서 즉시 사용할 수 있도록 돕는 프로세스가 있습니다.
  - 이 프로세스의 도입으로 Environment에 대한 보일러 플레이트 코드가 엄~청 줄어들었다(감사합니다!!!!).



```swift
struct NumberFactClient {
  var fetch: (Int) async throws -> String
}
```

- number fact기능을 새로운 타입으로 래핑하며 시작합니다.

<br/>

```swift
extension NumberFactClient: DependencyKey {
  static let liveValue = Self(
    fetch: { number in
      let (data, _) = try await URLSession.shared
        .data(from: .init(string: "http://numbersapi.com/\(number)")!)
      return String(decoding: data, as: UTF8.self)
    }
  )
}

extension DependencyValues {
  var numberFact: NumberFactClient {
    get { self[NumberFactClient.self] }
    set { self[NumberFactClient.self] = newValue }
  }
}
```

- NumberFactClient에 대해 `DependencyKey`프로토콜 채택을 통해 종속성 관리 시스템에 등록합니다.
- 해당 프로토콜은 시뮬레이터 혹은 디바이스에서 앱을 실행할 때 사용할 live value을 요구합니다.

<br/>

```swift
struct Feature: ReducerProtocol {
  struct State { … }
  enum Action { … }
  @Dependency(\.numberFact) var numberFact
  …
}
```

- 앱에서 `numberFact`에 대한 의존성 사용에 대해 인체공학적으로 개선시키기 위해, 새로운 타입을 생성하고  `DependencyKey`을 채택하고 live value를 초기화하는 과정을 밟았습니다.
- 이젠 어느 Feature에서든 좀 전에 선언한 `numberFact`의존성에 대해 어디서든지 즉시 사용할 수 있습니다.
- 그리고 이전과 동일하게 작동하면서도 Reducer를 구성할 때 dependency를 명시적으로 전달할 필요가 없습니다.
- 프리뷰, 시뮬레이터, 디바이스를 통해 앱을 실행할 때는 reducer에 live dependency가 제공되고, 테스트에서는 test dependency가 제공됩니다.

<br/>

```swift
@main
struct MyApp: App {
  var body: some Scene {
    FeatureView(
      store: Store(
        initialState: Feature.State(),
        reducer: Feature()
      )
    )
  }
}
```

- 이 뜻은 앱의 시작점에서 더이상 의존성을 구성할 필요가 없어졌음을 의미합니다.

```swift
let store = TestStore(
  initialState: Feature.State(),
  reducer: Feature()
)

store.dependencies.numberFact.fetch = { "\($0) is a good number Brent" }

…
```

- 그리고 구체적인 의존성 구현을 하지 않고도 test store를 구성할 수 있습니다.
- 테스트 목적에 맞게 의존성들을 재정의하여 사용하면 됩니다.

<br/>

<br/>

## 마무리

Composable Architecture에 대한 기본적인 설명은 여기까지 입니다. 이 외에 composition, modularity, adaptability, complex  effects 등 알아볼 것들이 많이 존재합니다. [예제 프로젝트들](https://github.com/pointfreeco/swift-composable-architecture/blob/main/Examples)을 통해 고급 사용법들을 확인할 수 있습니다.
  

<br/>

<br/>



## 참고

- [https://gist.github.com/pilgwon/ea05e2207ab68bdd1f49dff97b293b17#자주-묻는-질문](https://gist.github.com/pilgwon/ea05e2207ab68bdd1f49dff97b293b17#%EC%9E%90%EC%A3%BC-%EB%AC%BB%EB%8A%94-%EC%A7%88%EB%AC%B8)
- [https://github.com/pointfreeco/swift-composable-architecture](https://github.com/pointfreeco/swift-composable-architecture)
- [https://www.kodeco.com/24550178-getting-started-with-the-composable-architecture](https://www.kodeco.com/24550178-getting-started-with-the-composable-architecture)
