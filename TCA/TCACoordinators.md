# TCACoordinator

- TCACoordinator는 ComposableArchitecture를 이용하여 SwiftUI의 navigation에 대해 유연한 접근 방식을 제공합니다.
- TCACoordinator를 통해 복잡한 navigation와 presentation 플로우를 high-level의 코디네이터로 끌어올려 관리할 수 있습니다.
- 이 패턴을 사용하면 앱의 navigation 플로우 내에서 화면간 의존성이 없는 독립된 화면들을 구현할 수 있습니다.
- TCA가 제공하는 `.forEach`, `ifCaseLet`, `SwitchStore`와 [SwiftUI의 navigation을 다루는 새로운 접근](https://github.com/johnpatrickmorgan/TCACoordinators)(FlowStacks)이 결합되어있습니다.

<br/>

TCACoordinator를 통해 할 수 있는 것

- [ ] 앱 내에 깊이 중첩된 navigation에 대한 딥 링크를 지원합니다.
- [ ] 다른 navigation이라도 Screen을 쉽게 재사용할 수 있습니다.
- [ ] root screen으로 되돌아가거나 navigation내의 특정 screen으로 이동하기 쉽습니다.
- [ ] 이 모든 navigation 로직을 한 곳에서 관리합니다.
- [ ] 앱의 navigation을 재사용 가능한 여러개의 Coordinator로 나누고 다시 합칠 수 있습니다.
- [ ] 단일 시스템을 사용하여 push navigation과 modal presentation을 통합합니다.

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





<br/>

<br/>



## Dismiss될 때 Effect의 취소





## 복잡한 navigation 업데이트



## Child coordinator들로 구성



## Identifying Screens



## 유연성과 재사용성











## 출처

- https://github.com/johnpatrickmorgan/TCACoordinators