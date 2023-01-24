# SwiftUI Property Wrappers

<img width="1788" alt="decision_draft" src="https://user-images.githubusercontent.com/42647277/214206598-51b6e3b6-51ed-4d60-bdba-fca6241256a7.png">



## @State

- `@State` property wrapper는 `View`객체 안에서 주로 사용되고 `@State`에 대한 모든 변경사항에 대해 view가 응답할 수 있도록 합니다. 
- `@State`는 view가 가지고 있는 속성에 붙일 수 있습니다. 
- view는 `@State` 속성 자체를 초기화 합니다. 다른 객체로부터 `@State` 프로퍼티를 받지 않습니다.

```swift
struct StateExample: View {
	@State private var intValue = 0
            
	var body: some View {
		VStack {
			Text("intValue equals \(intValue)")
                    
			Button("Increment") {
					intValue += 1
			}
		}
	}
}
```

- 내부적으로 SwiftUI는 `@State` 프로퍼티의 값을 저장하고 view를 다시 렌더링하는 동안 해당 값을 유지합니다.
- 이것은 view에 의해 관리되고 SwiftUI가 refresh중에 view 인스턴스를 버리고 다시 만들어야 할 때 유지되어야 하는 state에 사용하기 적합합니다.
- `@State` 프로퍼티는 `private`로 선언해주는 것이 좋습니다. 외부 어떤 속성도 `@State` 프로퍼티를 수정해서는 안됩니다.
  - 의도한 방식으로만 사용하고 예상치 못한 state 수정으로 인한 잠재적인 버그를 방지하기 위함입니다.

- 아래와 같은 상황일 때 `@State`를 사용합니다.
  - view가 감싸고 싶은 인스턴스를 생성하고 소유할 떄
  - wrapped property에서 발생한 변화에 대해 응답하고 싶을 때
  - value type을 감쌀 때(struct or enum)
    - reference type(class)에도 사용할 수 있지만 인스턴스의 프로퍼티를 변경하는 것은 업데이트로 간주되지 않습니다. 변경한 프로퍼티가 `@Published`여도 마찬가지 입니다. 이 때에는 `@ObservedObject`, `@StateObject`, `@EnvironmentObject`를 살펴보는 것이 좋습니다.

<br/>

<br/>

## @Binding

- `@Binding` property wrapper는 다른 view로부터 전달받는 프로퍼티에 사용됩니다. 
- 바인딩 값을 받는 view는 바인딩된 프로퍼티를 읽고 외부 소스의 변경 사항에 응답할 수 있으며 프로퍼티에 대한 <u>쓰기</u> 권한이 있습니다.
- `@Binding`을 업데이트하면 `@Binding`을 제공한 view에서 해당 프로퍼티가 업데이트 된다는 의미입니다.

```swift
struct BindingView: View {
  @Binding var intValue: Int
  
  var body: some View {
    VStack {
      Button("Increment") {
        intValue += 1
      }
    }
  }
}
```

- 아래의 예시는 Binding을 받고 사용자가 버튼을 탭하여 값을 변경하는 view입니다.

```swift
struct StateView: View {
  @State private var intValue = 0
  
  var body: some View {
    VStack {
      Text("intValue equals \(intValue)")
      
      BindingView(intValue: $intValue)
    }
  }
}

struct BindingView: View {
  @Binding var intValue: Int
  
  var body: some View {
    Button("Increment") {
      intValue += 1
    }
  }
}
```

- `$intValue`를 보면 접두사로 $를 붙여 BindingView에  `@State`로 래핑된 intValue에 대한 바인딩을 전달합니다. 
- `@State`의 projected value는 `Binding<T>`으로 자식 뷰에게 전달할 수 있습니다. 그러므로 `@State`를 바인딩을 통해 간접적으로 수정할 수 있게됩니다.
- 내부적으로 SwiftUI는 뷰가 폐기될 때 `@Binding`을 유지하지 않습니다. `@Binding`을 항상 외부 소스로부터 전달받기 때문에 유지할 필요가 없기 때문이죠.
- 아래와 같은 상황일 때 `@Binding`를 사용합니다.
  - 부모 뷰가 소유하고 있는 프로퍼티에 대해 값을 읽고 쓰는 것이 필요할 때
  - wrapped property가 value type일 때(struct or enum)
    - reference type에도 사용할 순 있으나 일반적이진 않음
  - wrapped property를 소유하고 있지 않을 때(부모 뷰가 제공해줄 때)

<br/>

<br/>

## @StateObject

> *Only available in iOS 14+, iPadOS 14+, watchOS 7+, macOS 10.16+ etc.*

- `@StateObject`는 `@State`와 비슷한 이유로 사용됩니다(`ObservableObject`에 적용하는 것은 제외). 
- `ObservableObject`는 항상 reference type(class)이고 `@Published` 프로퍼티들 중에 하나가 변경될 때마다 SwiftUI에게 알립니다.

```swift
class DataProvider: ObservableObject {
  @Published var currentValue = "a value"
}

struct DataOwnerView: View {
  @StateObject private var provider = DataProvider()
  
  var body: some View {
    Text("provider value: \(provider.currentValue)")
  }
}
```

- `DataOwnerView`는 `DataProvider` 인스턴스를 생성합니다. 이 뜻은 `DataOwnerView`가 `DataProvider`를 소유한다는 뜻이죠. `DataProvider.currentValue` 가 변경될 때마다 `DataOwnerView`가 다시 렌더링 됩니다.
- 내부적으로 SwiftUI가 새로운 렌더링을 위해 `DataOwnerView`를 버리고 다시 생성할 때마다 `DataProvider`인스턴스는 유지됩니다. 이 뜻은 `@StateObject`가 한번만 초기화됨을 의미합니다.
- SwiftUI는 `@StateObject`와 연결된 인스턴스를 별도로 설정하고 `@StateObject`를 가지고있는 view가 다시 초기화될 때마다 인스턴스를 재사용합니다. 이 말은 새로운 view 인스턴스가 재사용될 때 `@StateObject`로 표기된 프로퍼티의 새로운 인스턴스를 가져오지 않는다는 뜻입니다.
- SwiftUI에 의해 구조체가 다시 생성되는 경우에도 `@StateObject`로 표기된 프로퍼티는 뷰가 필요한 동안에는 초기에 할당된 `ObservedObject` 인스턴스를 유지합니다.
- 이것은 reference type(class)에 `ObservableObject`이 적용된다는 점을 제외하면 `@State`와 동일한 동작을 합니다.
- 아래와 같은 상황일 때 `@StateObject`를 사용합니다.
  - `ObservableObject`의 변경사항에 대해 응답하고 싶을 때
  - view에서 `ObservableObject`의 인스턴스를 생성하여 사용할 때



## @ObservedObject

<br/>

<br/>

## @ObservedObject

- `ObservedObject`는 view에서 생성되거나 소유되지 않은 `ObservableObject` 인스턴스를 래핑하는데 사용됩니다.
- `@StateObject`와 동일한 타입(reference type)의 객체에 적용되며 view가 자체 `@ObservedObject` 인스턴스를 생성하지 않는 점을 제외하면 유사합니다. 직접 생성하지 않고 다른 뷰에게 전달하는 방식을 채택합니다.

```swift
struct DataOwnerView: View {
  @StateObject private var provider = DataProvider()
  
  var body: some View {
    VStack {
      Text("provider value: \(provider.currentValue)")
      
      DataUserView(provider: provider)
    }
  }
}

struct DataUserView: View {
  @ObservedObject var provider: DataProvider
  
  var body: some View {
    // create body and use / modify `provider`
  }
}
```

- `DataOwnerView`는  `DataUserView`에게 `@StateObject`에 대한 참조를 전달합니다. 
- 내부적으로 SwiftUI는 새롭게 렌더링이 필요한 경우 버리고 다시 생성하는데 이 때 `@ObservedObject`를 유지하지 않습니다.
- 대신 SwiftUI는 부모 뷰가  `@ObservedObject`가 붙은 프로퍼티의 값으로 `ObservedObject`를 전달할 것을 알고 있습니다.
  - 이 때 부모 뷰가 프로퍼티를 소유할 경우 `@StateObject`가 되고, 소유하지 않을 경우 `@ObservedObject`가 됩니다.
- 아래와 같은 상황일 때 `@ObservedObject`를 사용합니다.
  - `ObservedObject`의 변경사항에 대해 응답하고 싶을 때
  - view가 `ObservedObject`의 인스턴스를 생성하지 않을 때(생성한다면 `@StateObject`를 사용하세요)

<br/>

<br/>

## @EnvironmentObject

- 앱 내에 다양한 곳에서 객체가 필요하지만 각 뷰에 일일히 전달하면서 사용하고 싶지 않을 수도 있습니다.
- 이 때 App 혹은 Scene에서 하위 뷰들에게 dependency를 사용할 수 있는 프로퍼티 래퍼가 있습니다.

```swift
// child
struct EnvironmentUsingView: View {
  @EnvironmentObject var dependency: DataProvider
  
  var body: some View {
    Text(dependency.currentValue)
  }
}

// parent
struct MyApp: App {
  @StateObject var dataProvider = DataProvider()
  
  var body: some Scene {
    WindowGroup {
      EnvironmentUsingView()
        .environmentObject(dataProvider)
    }
  }
}
```

- `@EnvironmentObject`가 표시된 프로퍼티는 반드시 `ObservableObject`를 채택해야 합니다.
- `@EnvironmentObject`를 사용하는 객체의 상위 객체에 의해 구성됩니다.
- App 구조체에서 environment object를 주입하여 우리가 생성하는 모든 view에서 사용할 수 있도록 구현이 가능합니다.
- `@EnvironmentObject`는 `@ObservedObject`와 기능이 동일합니다. `@EnvironmentObject`에 속한 프로퍼티들 중 하나가 변경된다면 view는 이에 맞춰 렌더링합니다.
- 가장 큰 차이점은 `@EnvironmentObject`의 프로퍼티들은 `@ObservedObject`보다 훨씬 더 큰 규모로 제공된다는 것입니다. 위 예제에서 볼 수 있듯 앱 전체에서 주입하는 것처럼 말이죠.
- 아래와 같은 상황일 때 `@EnvironmentObject`를 사용합니다.
  - `@ObservableObject`에 대해 `@ObservedObject`를 사용하면 뷰에 도달하기 전까지 계속해서 초기화할 때 전달해야 합니다. 객체를 한번에 주입하고 싶을 때 사용하세요.

<br/>

<br/>

## @Environment

- `@Environment`는 `@EnvironmentObject`와 한가지만 빼고 똑같습니다. 차이점은 view의 environment로부터 값을 읽는데 사용된다는 점입니다.
- environment 안에있는 값이 바뀌면 view는 업데이트 됩니다.
- 이 property wrapper를 사용하여 environment의 속성을 변경할 수 없습니다.
- view에 `@Environment`속성을 설정하기 위해서는 `.environment` modifier를 사용해야 합니다.
- `@Environment`와 `@EnvironmentObject`의 관계는 `@State`와 `@StateObject`와의 관계를 생각하시면 됩니다.

```swift
struct MyView: App {
  @Environment(\.colorScheme) var colorScheme: ColorScheme
  
  var body: some View {
    Text("The color scheme is \(colorScheme == .dark ? "dark" : "light")")
  }
}
```

- 위 코드처럼 environment의 값을 읽을 수 있습니다.

<br/>

<br/>

```swift
ContentView()
  .environment(\.managedObjectContext, Persistence.shared.viewContext)
```

- 위와 같이 사용하면 view의 environment에 값을 할당할 수 있습니다.

<br/>

<br/>

```swift
// The type we want to use for the custom environment value
enum AppStyle {
  case classic, modern
}

// Our environment key
private struct AppStyleKey: EnvironmentKey {
  static let defaultValue = AppStyle.modern
}

// Register the key on SwiftUI's EnvironmentValues
extension EnvironmentValues {
  var appStyle: AppStyle {
    get { self[AppStyleKey.self] }
    set { self[AppStyleKey.self] = newValue }
  }
}

// Example usage
@main
struct PropertyWrappersApp: App {
  var body: some Scene {
    WindowGroup {
      ContentView()
        .environment(\.appStyle, .classic)
    }
  }
}

```

- view의 environment에 다양한 커스텀 프로퍼티를 추가할 수 있습니다.
- 아래와 같은 상황일 때 `@Environment`를 사용합니다.
  - SwiftUI environment에 key를 사용하여 값을 주입하고 싶을 때
  - 주입하는 프로퍼티가 `@EnvironmentObject`처럼 동작할 필요가 없을 때
  - `.environment` modifier를 받은 뷰의 모든 하위 뷰에서 사용할 수 있어야 할 때

<br/>

<br/>

## @FetchRequest

- Fetch request는 SwiftUI의 persistence와 관련된 프로퍼티 래퍼 중 하나입니다.
- 이것을 통해 Core Data에 있는 데이터를 검색할 수 있습니다.

```swift
struct ContentView: View {
  @FetchRequest(fetchRequest: MyModel.fetchRequest())
  var items: MyModel
  
  var body: some View {
    List(items) { item in
      Text(item.title)
    }
  }
}
```

- `@FetchRequest`에 의해 로드된 데이터가 업데이트될 때마다 뷰도 같이 업데이트됩니다.
- `@FetchRequest`에 대한 다양한 초기화 함수가 있지만 여기서는 다루지 않을 것입니다. 이 [사이트](https://www.youtube.com/watch?v=P8rqjs_CNsk)를 참고하세요.
- 아래와 같은 상황일 때 `@FetchRequest`를 사용합니다.
  - Core Data에 저장되어있는 데이터를 view로 빠르게 불러오고 싶을 때



<br/>

<br/>

## @AppStorage

- `@AppStorage`는 UserDefaults와 관련된 것으로 앱 전체를 아우르는 프로퍼티 래퍼입니다. 즉, 단순한 키-값 쌍을 저장하기 굉장히 좋은 프로퍼티 래퍼입니다.
- `UserDefaults`에 저장된 데이터가 변경되면 뷰도 변경됩니다.
- 프로퍼티에 값을 할당하는 것을 통해 `@AppStorage`의 값을 변경할 수 있습니다.

```swift
struct ContentView: View {
  @AppStorage("lastTap") var lastTap: Double?
  
  var dateString: String {
    if let timestamp = lastTap {
      return Date(timeIntervalSince1970: timestamp).formatted()
    } else {
      return "Never"
    }
  }
  
  var body: some View {
    Text("Button was last clicked on \(dateString)")
    
    Button("Click me") {
      lastTap = Date().timeIntervalSince1970
    }
  }
}
```

- 유의해야 할 점은 `@AppStorage`는 앱의 전체 데이터 모델을 보유하기 위한 것이 아니므로 작고 간단한 데이터를 저장하는 용도로만 사용해야 합니다.
- 아래와 같은 상황일 때 `@AppStorage`를 사용합니다.
  - 간단한 유저의 환경설정을 저장할 때
  - 유저가 마지막으로 앱을 켰을 때 추적할 간단한 데이터를 저장할 때
  - 앱을 다시 시작해야 하는 매우 간단한 state를 저장해야 할 때

<br/>

<br/>

## @SceneStorage

- `@SceneStorage`는 `@AppStorage`와 한 가지만 빼고 같습니다. 그것은 현재 뷰가 속한 Scene에 데이터를 저장한다는 것입니다.
- iOS의 앱은 일반적으로 1개의 scene을 사용하지만 mac이나 iPad의 앱은 여러개의 scene을 사용합니다.
- Scene의 해체되었다 다시 복구되면 scene storage를 통해 다시 사용할 수 있습니다. 만약 scene이 없어지면 scene과 관련된 모든 데이터도 없어집니다.
- 앱에 중요하지 않은 상태 관련 데이터만 scene storage에 저장해야 합니다.

```swift
struct ContentView: View {
  @SceneStorage("lastTap") var lastTap: Double?
  
  var dateString: String {
    if let timestamp = lastTap {
      return Date(timeIntervalSince1970: timestamp).formatted()
    } else {
      return "Never"
    }
  }
  
  var body: some View {
    Text("Button was last clicked on \(dateString)")
    
    Button("Click me") {
      lastTap = Date().timeIntervalSince1970
    }
  }
}

```

- 아래와 같은 상황일 때 `@AppStorage`를 사용합니다.
  - 현재 scene과 관련된 간단한 state만 저장할 때
  - 저장하려는 데이터가 민감하거나 필수적인 것이 아닐 때

<br/>

## 출처

- https://swiftuipropertywrappers.com/#state