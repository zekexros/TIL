# ReducerProtocol의 body property

- ComposableArchitecture의 공식문서에 따르면 ReducerProtocol을 따르는 기능을 구현하는데 이 때 reducer를 구현하는 2가지 방법이 있다고 합니다.
  - 첫 번째는 `reduce(into:action:)`을 구현하여 `Action`이 시스템에 들어올 때마다 앱의 `State`에 직접 접근하여 변경이 가능하고, 외부 세계와 소통하는 `EffectTask`를 반환할 수 있고, 추가 `Action`들을 다시 시스템에 입력할 수 있습니다
  - 두 번째는 하나 이상의 reducer를 함께 결합할 수 있는  `body` property를 구현할 수 있습니다.

- 두 가지 중 최대 1개만 구현해야 합니다. 두 가지 모두 구현할 경우 `reducer(into:action:)`만 호출됩니다.

- 다른 reducer로부터 reducer의 body에 합치고 비즈니스 로직을 추가할 경우 `Reduce`를 이용하여 로직을 body에 추가합니다.

  ```swift
  var body: some ReducerProtocol<State, Action> {
    Reduce { state, action in
      // extra logic
    }
    Activity()
    Profile()
    Settings()
  }
  ```

- 혹은 새로운 로직을 Reduce로 감싸진 메소드로 보내면 됩니다.

  ```swift
  var body: some ReducerProtocol<State, Action> {
    Reduce(self.core)
    Activity()
    Profile()
    Settings()
  }
  
  func core(state: inout State, action: Action) -> EffectTask<Action> {
    // extra logic
  }
  ```

- 기존 reducer를 변환하는 커스텀 reducer operator를 구현하려는 경우, body가 아닌 `reduce(into:action:)`메소드를 호출하세요. 예를 들어, 아래의 코드는 reducer로 보내는 액션의 로그를 찍어내는 operator입니다.

  ```swift
  extension ReducerProtocol {
    func logActions() -> some ReducerProtocol<State, Action> {
      Reduce { state, action in
        print("Received action: \(action)")
        return self.reduce(into: &state, action: action)
      }
    }
  }
  ```

  



## 출처

- https://pointfreeco.github.io/swift-composable-architecture/main/documentation/composablearchitecture/reducerprotocol

