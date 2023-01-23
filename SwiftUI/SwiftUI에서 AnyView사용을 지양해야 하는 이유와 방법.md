# SwiftUI에서 AnyView사용을 지양해야 하는 이유와 방법

- SwiftUI는 type-erased view로서 `AnyView`를 제공합니다. 이로인해 특정 뷰를 구현할 때 `AnyView`를 사용하면 좀 더 편리하게 만들 수 있습니다. 하지만 애플은 `AnyView`의 사용을 추천하지 않습니다.
- 이유는 SwiftUI는 <u>structural identity</u>라고 불리는 메커니즘을 사용하는데 이는 view의 **타입**을 사용하여 식별하고 업데이트 시기를 결정하는 것을 말합니다. 
- View에 변경사항이 있어도 `AnyView`는 view의 타입을 지움으로 인해 타입 시스템 관점에서 항상 동일하게 보이기 때문에 뷰를 효율적으로 업데이트하는 기능이 크게 줄어듭니다.



## AnyView를 사용하지 않는 여러가지 방법들

### 1. ViewBuilder

```swift
private var nameView: some View {
    if isEditable {
        return TextField("Your name", text: $name)
    } else {
        return Text(name)
    }
}
```

- 새로운 뷰를 만드는 연산 프로퍼티 혹은 메소드를 정의했을 때 우리는 종종 `some View`라는 opaque type을 사용합니다. 따라서 구체 타입을 명시해줄 필요가 없었죠.
- 하지만 위 코드는 `Function declares an opaque return type, but the return statements in its body do not have matching underlying types`라는 컴파일 에러를 뱉습니다.

<br/>

<br/>

```swift
private var nameView: some View {
    if isEditable {
        return AnyView(TextField("Your name", text: $name))
    } else {
        return AnyView(Text(name))
    }
}
```

- 이러한 컴파일 에러를 해결하기 위해 `AnyView`를 사용하기도 합니다.

<br/>

<br/>

```swift
@ViewBuilder
private var nameView: some View {
    if isEditable {
        TextField("Your name", text: $name)
    } else {
        Text(name)
    }
}
```

-  하지만 좀 더 나은 방법은 `ViewBuilder`를 사용하는 것입니다. 그저 프로퍼티 혹은 메소드에 `@ViewBuilder` attribute를 추가하고 `return`구문을 제거하면 끝입니다.
- SwiftUI의 body와 같은 메커니즘이지만 한 가지 다른점은 명시적으로 `ViewBuilder`를 추가해주느냐 아니냐의 차이입니다.

<br/>

<br/>

### 2. Group

```swift
private var nameView: some View {
    Group {
        if isEditable {
            TextField("Your name", text: $name)
        } else {
            Text(name)
        }
    }
}
```

- 간단하게 `Group`을 사용하면 여러개의 뷰를 단일 뷰로 나타낼 수 있습니다.
- 이유는 `Group`은 `ViewBuilder`를 사용하기 때문입니다.



### 3. Generics

```swift
struct FlyoutView: View {

    let headerView: AnyView

    var body: some View {
        VStack {
            headerView
            Spacer()
            // other views
        }
    }
}
```

- 우리는 view의 타입을 모르는 채로 저장하고 싶을 때 `AnyView`를 종종 사용합니다.
- 저장 프로퍼티로 `some View`같은 opaque type은 사용할 수 없기 때문이죠.

<br/>

<br/>

```swift
struct FlyoutView<HeaderView> : View where HeaderView : View {

    let headerView: HeaderView

    var body: some View {
        VStack {
            headerView
            Spacer()
        }
    }
}

// When creating FlyoutView
FlyoutView(headerView: Text("Header view"))

```

- 그 때 `AnyView`의 사용 대신 generics를 이용하면 훨씬 퍼포먼스 좋고 아름다운 코드가 됩니다.
- 그리고 `FlyoutView`를 초기화할 때 인자로 들어올 뷰를  `AnyView`로 감싸주지 않아도 됩니다.

<br/>

```swift
struct VStack<Content> : View where Content : View {
}
```

- 이러한 방식은 이미 SwiftUI에서 굉장히 널리 제공되고 있습니다. 예를 들면 `VStack`이 그렇습니다.



<br/>

## 출처

- https://tanaschita.com/20210802-how-to-avoid-using-anyview-in-swiftui/

- https://www.swiftbysundell.com/articles/avoiding-anyview-in-swiftui/
