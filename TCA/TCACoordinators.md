# TCACoordinator

- TCACoordinator는 ComposableArchitecture를 이용하여 SwiftUI의 navigation에 대해 유연한 접근 방식을 제공합니다.
- TCACoordinator를 통해 복잡한 navigation와 presentation 플로우를 high-level의 코디네이터로 끌어올려 관리할 수 있습니다.
- 이 패턴을 사용하면 앱의 navigation 플로우 내에서 화면간 의존성이 없는 독립된 화면들을 구현할 수 있습니다.
- TCA가 제공하는 `.forEach`, `ifCaseLet`, `SwitchStore`와 [SwiftUI의 navigation을 다루는 새로운 접근](https://github.com/johnpatrickmorgan/TCACoordinators)(FlowStacks)이 결합되어있습니다.

<br/>

TCACoordinator를 통해 할 수 있는 것

- [x] 앱 내에 깊이 중첩된 navigation에 대한 딥 링크를 지원합니다.
- [x] 다른 navigation이라도 Screen을 쉽게 재사용할 수 있습니다.
- [x] root screen으로 되돌아가거나 navigation내의 특정 screen으로 이동하기 쉽습니다.
- [x] 이 모든 navigation 로직을 한 곳에서 관리합니다.
- [x] 앱의 navigation을 재사용 가능한 여러개의 Coordinator로 나누고 다시 합칠 수 있습니다.
- [x] 단일 시스템을 사용하여 push navigation과 modal presentation을 통합합니다.

<br/>

이 라이브러리는 screen이 담긴 배열을 중첩된 `NavigationLink`와 presentation 호출로 변환하여 작동하므로..

- UIKit에 의존하지 않습니다.
- Screen의 타입을 지우기 위해 AnyView를 사용하지 않습니다.
- 처음부터 NavigationView를 다시 만드려고 시도하지 않습니다.

<br/><br/>



## 사용법

### 1. Screen Reducer 만들기

```swift
struct Screen: ReducerProtocol {
  enum State: Equatable {
    case home(Home.State)
    case numbersList(NumbersList.State)
    case numberDetail(NumberDetail.State)
  }
  enum Action {
    case home(Home.Action)
    case numbersList(NumbersList.Action)
    case numberDetail(NumberDetail.Action)
  }
  
  var body: some ReducerProtocol<State, Action> {
    Scope(state: /State.home, action: /Action.home) {
      Home()
    }
    Scope(state: /State.numbersList, action: /Action.numbersList) {
      NumbersList()
    }
    Scope(state: /State.numberDetail, action: /Action.numberDetail) {
      NumberDetail()
    }
  }
}
```

- 우선, Navigation 플로우의 일부가 되는 Screen들을 포함하여 모든 Screen들을 식별합니다.
- 목표는 reducer들을 단일 reducer로 결합하는 것입니다. 단일 reducer는 화면의 동작들을 구동시킬 수 있습니다.
- state와 action타입들은 개별 screen의 state와 action타입들의 집합이며, reducer는 각 screen의 reducer들을 하나로 결합합니다.

<br/><br/>

### 2. Coordinator reducer 만들기

```swift
struct Coordinator: ReducerProtocol {
  struct State: Equatable, IndexedRouterState {
    var routes: [Route<Screen.State>]
  }
  ...
}
```

- Coordinator는 navigation 플로우 안에서 여러 screen들을 관리합니다.
- Coordinator의 state는 navigation stack을 나타내는  `Route<Screen.State>`배열이 포함되야 합니다. 즉, 이 배열에 새로운 화면의 state를 추가하면 해당 화면이 push되거나 presented되도록 트리거됩니다.
- `Route`는 enum으로 case들은 screen의 state를 캡쳐하고 어떻게 보여지는지 나타낸다. 예를 들어,  `case push(Screen.State)` 코드 처럼.

<br/><br/>

```swift
struct Coordinator: ReducerProtocol {
  ...
  enum Action: IndexedRouterAction {
    case routeAction(Int, action: Screen.Action)
    case updateRoutes([Route<Screen.State>])
  }
  ...
}
```

- Coordinator의 action은 2개의 특별한 case들이 포함되어야 합니다.
- 첫 번째는 routes 배열의 올바른 화면으로 screen의 action들을 전송할 수 있도록 index를 포함합니다.
- 두 번째는 routes 배열이 자동적으로 업데이트되도록 합니다. 예를 들어 유저가 `Back`을 탭할 때가 있겠네요.

<br/><br/>

```swift
struct Coordinator: ReducerProtocol {
  ...
  var body: some ReducerProtocol<State, Action> {
    return Reduce<State, Action> { state, action in
      switch action {
      case .routeAction(_, .home(.startTapped)):
        state.routes.presentSheet(.numbersList(.init(numbers: Array(0 ..< 4))), embedInNavigationView: true)
        
      case .routeAction(_, .numbersList(.numberSelected(let number))):
        state.routes.push(.numberDetail(.init(number: number)))
        
      case .routeAction(_, .numberDetail(.showDouble(let number))):
        state.routes.presentSheet(.numberDetail(.init(number: number * 2)))
        
      case .routeAction(_, .numberDetail(.goBackTapped)):
        state.routes.goBack()
        
      case .routeAction(_, .numberDetail(.goBackToNumbersList)):
        return .routeWithDelaysIfUnsupported(state.routes) {
          $0.goBackTo(/Screen.State.numbersList)
        }
        
      case .routeAction(_, .numberDetail(.goBackToRootTapped)):
        return .routeWithDelaysIfUnsupported(state.routes) {
          $0.goBackToRoot()
        }
        
      default:
        break
      }
      return .none
    }.forEachRoute {
      Screen()
    }
  }
}
```

- Coordinator reducer는 screen들에 대한 presenting 혹은 dismissing 같은 로직들을 정의합니다. 그리고 `forEachRoute`를 사용하여 `routes`배열의 각 screen에 `Screen` reducer를 추가로 적용합니다.

<br/><br/>

### 3. Coordinator View 만들기

```swift
struct CoordinatorView: View {
  let store: StoreOf<Coordinator>

  var body: some View {
    TCARouter(store) { screen in
      SwitchStore(screen) {
        CaseLet(
          state: /Screen.State.home,
          action: Screen.Action.home,
          then: HomeView.init
        )
        CaseLet(
          state: /Screen.State.numbersList,
          action: Screen.Action.numbersList,
          then: NumbersListView.init
        )
        CaseLet(
          state: /Screen.State.numberDetail,
          action: Screen.Action.numberDetail,
          then: NumberDetailView.init
        )
      }
    }
  }
}
```

- 이제 `Coordinator View`를 생성할 수 있습니다. `Coordinator View`는 routes 배열을 보이지 않는 NavigationLinks와 presentation들을 가진 screen들이 중첩된 리스트로 변환해주는 `TCARouter`를 사용할 것입니다. 그리하여 routes 배열의 변경 사항에 적절하게 반응하는 바인딩으로 구성됩니다.
- `TCARouter`는 navigation 플로우의 모든 screen에 대한 view를 생성할 수 있는 클로저를 갖습니다.
- `SwitchStore`는 위 사항을 달성하는 가장 자연스러운 방법이며, 각 화면에 대해 `CaseLet`을 같이 사용합니다.

<br/> <br/>

## 편리한 메소드들

- routes 배열은 `append`와 같은 일반 배열 메소드들을 사용하여 관리할 수 있습니다. 그러나 일반적인 변환에 대해 사용할 수 있는 몇가지 편리한 메소드들 또한 준비되어 있습니다.
- presented된 화면에서 screen들을 push하고싶다면 `embedInNavigationView: true`를 사용하세요.

| Method       | Effect                                           |
| ------------ | ------------------------------------------------ |
| push         | Pushes a new screen onto the stack.              |
| presentSheet | Presents a new screen as a sheet.†               |
| presentCover | Presents a new screen as a full-screen cover.†   |
| goBack       | Goes back one screen in the stack.               |
| goBackToRoot | Goes back to the very first screen in the stack. |
| goBackTo     | Goes back to a specific screen in the stack.     |
| pop          | Pops the current screen if it was pushed.        |
| dismiss      | Dismisses the most recently presented screen.    |

<br/><br/>

## 자동으로 업데이트되는 Routes 배열

- 유저가 back버튼을 누르면 routes 배열은 새로운 navigation state를 반영하도록 자동적으로 업데이트 됩니다.
- 화면 가장자리 스와이프 혹은 back버튼을 오래 누르기를 통해 화면을 뒤로 이동하면 routes 배열이 자동으로 업데이트 됩니다. 마치 sheet를 스와이프하여 닫으면 자동으로 routes 배열이 업데이트 되는 것처럼 말이죠.

<br/>

<br/>



## Dismiss될 때 Effect의 취소

- 일반적으로 특정 화면에서 시작된 effect는 해당 화면이 pop되거나 dismiss되면 자동적으로 취소됩니다. 이건 많은 보일러플레이트 코드가 필요하지만 추가 작업 없이 이 라이브러리 내에서 완전히 처리됩니다. 
- 자동적으로 effect들이 cancel되는것을 거부하려면 `forEachRoute`에 인자로 `cancellationId: nil`를 전달하세요.

<br/>

<br/>

## 복잡한 navigation 업데이트

- SwiftUI에서는 한번의 업데이트 안에 2개 이상의 화면을 push, present, dismiss를 할 수 없습니다.(업데이트 한번에 화면전환 1개만 가능하단 소리)

- 이로인해 navigation상태를 큰 단위로 업데이트하기가 까다롭습니다. 예를 들어, 

  - navigation계층의 깊숙한 곳으로 딥링킹을 할 때
  - 여러 가지 화면을 지나 다시 root 화면으로 되돌아올 때
  - 임의의 navigation state를 되돌릴 때(이 상황은 어떤 말인지 잘 이해가 안가네요)

- 이 라이브러리는 해결방법을 제공합니다. 

  - 지원되지 않는 큰 단위의 업데이트를 SwiftUI가 지원하는 작은 업데이트 단위로 나누고
  - 필요하다면 delay도 사용할 수 있도록 지원하고
  - coordinator reducer로부터 반환되는 effect를 사용할 수 있도록 만들어졌습니다.

- 사용법은 간단합니다. route mutation을 `Effect.routeWithDelaysIfUnsupported`로 감싸주면 됩니다.

  ```swift
  return Effect.routeWithDelaysIfUnsupported(state.routes) {
    $0.goBackToRoot()
  }
  ```

  ```swift
  return Effect.routeWithDelaysIfUnsupported(state.routes) {
    $0.push(...)
    $0.push(...)
    $0.presentSheet(...)
  }
  ```

<br/> <br/>

## Child coordinator들로 구성

- Coordinator는 Composable Architecture에서 사용하는 UI단위들과 똑같습니다. `View`와 `State`와 `Action`타입을 가진 `Reducer`로 구성된 것이 똑같죠.

- 이 말은 Coordinator들은 SwiftUI와 TCA가 허용하는 일반적인 방법들을 사용하여 compose 할 수 있습니다.

- Coordinator를 present하고, `TabView`에도 추가하고, 자식 coordinator를 routes 배열에 추가하여 부모 coordinator에서 push 혹은 present도 할 수 있습니다.

- 그렇게 하면 자식 coordinator는 부모 routes배열의 마지막에 존재하는 것이 가장 좋습니다. 왜냐하면 자식 coordinator는 dismiss 되기 전까지 새로운 화면을 push하고 present 할 책임이 있기 때문입니다.

- 그렇게 하지 않으면 **<span style="color:orange">자식이 화면을 push하고 있을 때 부모가 화면을 push하려고 시도하여 충돌이 발생할 수 있습니다</span>**

  

<br/>

<br/>

## Identifying Screens

- Coordinator의 state는 `IndexedRouterState`를 채택하고 있고 action은 `IndexedRouterAction`을 채택하고 있습니다.

- 이 뜻은 screen들은 route 배열 안에서 index로 식별되었다는 것을 의미합니다.

- 이것은 index가 navigation 업데이트들에 대해 안정적이므로 안전합니다. 예를 들어 push와 pop은 이미 존재하는 화면들의 index에 대해 영향을 주지 않습니다.

- 그러나 만약 `Identifiable` 화면들을 사용하려면 우리는 화면들을 `IdentifiedArray`로 관리해야 합니다.

- 그리고 state를 `IdentifiedRouterState`, action을 `IdentifiedRouterAction`을 채택하도록 해야 위의 이점들을 누릴 수 있습니다.

- 어떠한 프로토콜을 채택하지 않는 것을 원할 경우, 이를 테면 property와 case들의 이름을 다르게 지정하려는 경우 API의 명시적 버전을 사용할 수 있습니다.

  

<br/>

<br/>

## 유연성과 재사용성

- 화면들의 플로우를 바꿔야 하는 경우에는 한 곳에서 쉽게 변경할 수 있습니다.
- Navigation 플로우 안에서 screen의 view들과 reducer들은 더이상 다른 screen들을 알 필요가 없습니다.
- 단순히 action을 보내 코디네이터가 새로운 view를 push할지 또는 present할지 결정하도록 할 수 있습니다.
- **<span style="color:orange">이를 통해 다양한 화면에서 쉽게 재사용할 수 있으며 화면의 책임과 navigation의 책임을 분리할 수 있습니다.</span>**



<br/>



## 출처

- https://github.com/johnpatrickmorgan/TCACoordinators