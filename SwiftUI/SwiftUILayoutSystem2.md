# SwiftUI Layout System 2

- 이번엔 SwiftUI의 좀 더 고급 기술을 알아봅시다.
- 동적 넓이에 맞춰 정렬하는 방법이나 완벽한 커스텀 레이아웃 구현을 위한 뷰의 주변 geometry를 읽는법을 살펴볼 예정입니다.

<br/>

## Handling dynamic content

- 앱 UI의 몇몇 부분은 상대적으로 정적이고 예상 가능할 수 있지만 대부분의 경우에는 매우 동적으로 표현해야 합니다.
- 컴파일 타임에는 각 계정에 대한 컨텐츠를 알 수 없을뿐만 아니라 앱이 실행되는 환경에 따라 문자열의 로컬라이징 혹은 기타 리소스에 따라 뷰가 유연하게 확장하는지 확인해야 합니다.
- SwiftUI는 매우 동적인 앱 환경에 맞춰 설계되었습니다. 그리하여 선언한 뷰들이 컨텐츠, 환경 및 기타 요인에 따라 자동으로 바뀝니다.
- 실제 코드로 알아보기 위해 `EventInfoBadge`라는 간단한 뷰를 생성합니다.

<br/>

```swift
struct EventInfoBadge: View {
    var iconName: String
    var text: String

    var body: some View {
        VStack {
            Image(systemName: iconName)
                .resizable()
                .frame(width: 25, height: 25)
            Text(text)
        }
    }
}
```

- 위 코드는 딱히 문제없이 괜찮아보입니다. 하지만 새로운 ContentView의 하단에 수평으로 3개의 뱃지를 나란히 렌더링한다면 뭔가 이상할 것입니다.

<br/><br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            EventHeader()
            Spacer()
            HStack {
                EventInfoBadge(
                    iconName: "video.circle.fill",
                    text: "Video call available"
                )
                EventInfoBadge(
                    iconName: "doc.text.fill",
                    text: "Files are attached"
                )
                EventInfoBadge(
                    iconName: "person.crop.circle.badge.plus",
                    text: "Invites allowed"
                )
            }
        }.padding()
    }
}
```

<img width="260" alt="CleanShot 2023-01-21 at 15 30 33@2x" src="https://user-images.githubusercontent.com/42647277/213846980-a4911f89-5be9-4768-a013-af43c184942f.png">

- 여기서 우리는 2가지 문제점이 있습니다. 
  - 하나는 아이콘들의 aspect ratio가 잘못되어 늘어진 것처럼 보입니다.
  - 다른 하나는 뱃지들이 서로 다른 문자열을 렌더링하므로 서로 다른 너비를 갖게됩니다(고르게 보이지 않음).

<br/>

<br/>

```swift
struct EventInfoBadge: View {
    var iconName: String
    var text: String

    var body: some View {
        VStack {
            Image(systemName: iconName)
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: 25, height: 25)
            Text(text)
        }
    }
}
```

<img width="260" alt="CleanShot 2023-01-21 at 15 32 07@2x" src="https://user-images.githubusercontent.com/42647277/213847036-8036b8e9-f304-41df-8ffd-13f211b09acc.png">

- `Image`에 `.aspectRatio()` modifier를 적용시켜 아이콘이 늘어나는 문제를 해결해 봅시다.
- resize할 때 content를 bounds에 맞추도록 작동합니다.

<br/>

- 다음으로 3개의 뱃지를 `ContentView`에 모두 같은 공간을 차지하도록 만드려면, 각 뱃지가 컨테이너 내에서 가능한 많은 공간을 차지하도록 해야 합니다.
- 부모 뷰(여기선 HStack)가 가장 긴 텍스트를 가진 자식 뷰에게 가장 많은 공간을 제공하는 대신 각 뷰에게 사용 가능한 공간을 균등하게 나눠 배치합니다.

<br/>

```swift
struct EventInfoBadge: View {
    var iconName: String
    var text: String

    var body: some View {
        VStack {
            Image(systemName: iconName)
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: 25, height: 25)
            Text(text)
                .frame(maxWidth: .infinity)
        }
    }
}
```

<img width="260" alt="CleanShot 2023-01-21 at 15 32 39@2x" src="https://user-images.githubusercontent.com/42647277/213847055-a1b67c3d-60cd-4ed8-aab2-2b7e8fbb1e74.png">

- 따라서 `EventInfoBadge`의 `Text`에게 최대한 늘어나는 너비를 제공합니다.
- 그러면 레이아웃 시스템이 여러 줄로 분할하기 전에 가로 축에서 최대한 크기를 조정합니다.

<br/>

<br/>

```swift
struct EventInfoBadge: View {
    var iconName: String
    var text: String

    var body: some View {
        VStack {
            Image(systemName: iconName)
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: 25, height: 25)
            Text(text)
                .frame(maxWidth: .infinity)
                .multilineTextAlignment(.center)
        }
        .padding(.vertical, 10)
        .padding(.horizontal, 5)
        .background(Color.secondary)
        .cornerRadius(10)
    }
}
```

<img width="260" alt="CleanShot 2023-01-21 at 15 33 29@2x" src="https://user-images.githubusercontent.com/42647277/213847079-3acdc040-6d1b-453d-8ad5-2c1ab39f9700.png">

- 이제 텍스트를 가운데 정렬하고 여백을 추가하고 백그라운드 색상 지정하고 둥근 모서리로 만들면서  `EventInfoBadge`의 구현을 마무리합니다.

<br/>

<br/>

```swift
struct EventInfoList: View {
    var body: some View {
        HStack {
            EventInfoBadge(
                iconName: "video.circle.fill",
                text: "Video call available"
            )
            EventInfoBadge(
                iconName: "doc.text.fill",
                text: "Files are attached"
            )
            EventInfoBadge(
                iconName: "person.crop.circle.badge.plus",
                text: "Invites enabled"
            )
        }
    }
}
```

- 마지막으로 뱃지들을 포함하는 새로운 뷰를 하나의 UI Component로서 제공할 수 있도록 구현해 줍니다.

<br/>

<br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            EventHeader()
            Spacer()
            EventInfoList()
        }.padding()
    }
}
```

- 그러면 ContentView에는 간단하게 `EventInfoList`의 인스턴스를 초기화하여 사용하면 됩니다.

<br/>





## Geometry, preferences, and layout dependencies

- 하지만 아직 `EventInfoBadge`에는 한 가지 이슈가 남아있습니다.
- 텍스트 길이에 따라 동적으로 width를 다뤘지만 여전히 뱃지는 서로 다른 높이를 가지고 있고 이를 해결해야 합니다.
- 예를 들어 텍스트의 길이가 좀 더 길어질 때의 상황을 가정해보겠습니다.

<br/>

<br/>

```swift
struct EventInfoList: View {
    var body: some View {
        HStack {
            EventInfoBadge(
                iconName: "video.circle.fill",
                text: "Video call available"
            )
            EventInfoBadge(
                iconName: "doc.text.fill",
                text: "Files are attached"
            )
            EventInfoBadge(
                iconName: "person.crop.circle.badge.plus",
                text: "Invites enabled, 5 people maximum"
            )
        }
    }
}
```

<img width="260" alt="CleanShot 2023-01-21 at 15 35 25@2x" src="https://user-images.githubusercontent.com/42647277/213847153-dae2bb12-c8c7-4f16-abe6-5f8c43c22c28.png">

- 위 코드만으로도 나쁘진 않지만 높이가 일정하다면 훨씬 보기좋은 UI가 될 것입니다.
- `EventInfoList`의 자식중에서 최대 높이를 알리는 방법을 찾아야 합니다.
- 그러면 해당 자식 뷰가 나머지 자식 뷰들이 동일한 높이를 가지도록 조정할 수 있습니다.

<br/>

<br/>

- 위 View는 재사용 할 예정이므로 `HeightSyncedRow`라는 이름의 새로운 뷰에서 확인해보겠습니다.

```swift
struct HeightSyncedRow<Content: View>: View {
    private let content: Content
    @State private var childHeight: CGFloat?

    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        HStack {
            content.frame(height: childHeight)
        }
    }
}
```

- `@ViewBuilder`라는 function builder를 사용하여 SwiftUI에 내장된 container와 stack이 사용하는 것과 동일한 DSL 유사 구문으로 새 뷰를 사용할 수 있도록 하겠습니다.
- 그리고 childHeight를 할당합니다. 

<br/>

<br/>

```swift
struct EventInfoList: View {
    var body: some View {
        HeightSyncedRow {
            EventInfoBadge(
                iconName: "video.circle.fill",
                text: "Video call available"
            )
            EventInfoBadge(
                iconName: "doc.text.fill",
                text: "Files are attached"
            )
            EventInfoBadge(
                iconName: "person.crop.circle.badge.plus",
                text: "Invites enabled, 5 people maximum"
            )
        }
    }
}
```

- `@ViewBuilder`를 사용하는 것의 좋은 점은 `EventInfoList`로 돌아가서 추가 변경 없이  `HStack`을 새로운 `HeightSyncedRow`로 교체할 수 있다는 것입니다.

<br/>

<br/>

```swift
private struct HeightPreferenceKey: PreferenceKey {
    static let defaultValue: CGFloat = 0

    static func reduce(value: inout CGFloat,
                       nextValue: () -> CGFloat) {
        value = nextValue()
    }
}
```

- `HeightSyncedRow`가 각 자식 뷰에게 할당할 childHeight를 계산해 보겠습니다.
- 이를 위해선 SwiftUI의 preferences system을 사용하여 뷰 계층 구조를 따라 자식 뷰가 높이를 위쪽으로 보고하도록 할 것입니다.
- 주어진 값을 자식 뷰안의 preference key와 연결할 수 있으며 나중엔 부모 뷰에서 읽을 수도 있습니다.
- 그러기 위해선 default 값과 reduce()를 포함하는  `PreferenceKey`를 구현합니다.

<br/>

<br/>

```swift
extension View {
    func syncingHeightIfLarger(than height: Binding<CGFloat?>) -> some View {
        background(GeometryReader { proxy in
            // We have to attach our preference assignment to
            // some form of view, so we just use a clear color
            // here to make that view completely transparent:
            Color.clear.preference(
                key: HeightPreferenceKey.self,
                value: proxy.size.height
            )
        })
        .onPreferenceChange(HeightPreferenceKey.self) {
            height.wrappedValue = max(height.wrappedValue ?? 0, $0)
        }
    }
}
```

- 그리고 스유의 `GeometryReader`타입을 사용합니다. 이것은 현재 뷰의 컨테이너 크기를 읽을 수 있게 해주는 뷰입니다.
- 주어진 뷰의 `background`로 `GeometryReader`를 embed함으로써 뷰의 레이아웃에 어떠한 영향도 주지 않고 값을 읽을 수 있습니다.(백그라운드 뷰는 항상 연결된 뷰와 동일한 frame을 차지합니다)
- 마지막으로 View를 따르는 모든 뷰의 높이를 `Binding` property wrapper에 동기화할 수 있도록 view를 확장합니다.

<br/>

<br/>

```swift
struct HeightSyncedRow<Content: View>: View {
    private let content: Content
    @State private var childHeight: CGFloat?

    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        HStack {
            content.syncingHeightIfLarger(than: $childHeight)
                   .frame(height: childHeight)
        }
    }
}
```

- 이제 `HeightSyncedRow`로 되돌아가 새로 만든 `syncingHeightIfLarger` modifier를 적용하여 각각의 자식들이 같은 높이를 채택하도록 합니다.

<br/>

<br/>



```swift
struct HeightSyncedRow<Content: View>: View {
    private let content: Content
    @State private var childHeight: CGFloat?

    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        HStack {
            content.syncingHeightIfLarger(than: $childHeight)
                   .frame(height: childHeight)
                   .background(Color.red)
        }
    }
}
```

<img width="260" alt="CleanShot 2023-01-21 at 15 37 38@2x" src="https://user-images.githubusercontent.com/42647277/213847228-75ac13e5-946d-4c0d-bfa7-337dde129e41.png">

- 그러나 `ContentView`를 다시 렌더링해도 모든 뱃지의 높이가 동일한지 알 수 없습니다.
- 각 뱃지의 background color을 지정한 후 `.frame()`을 적용하기 때문입니다.
- 이 문제를 설명하기 위해 다시 background를 red로 하여 알아봅시다.

<br/>

<br/>

```swift
struct HeightSyncedRow<Background: View, Content: View>: View {
    private let background: Background
    private let content: Content
    @State private var childHeight: CGFloat?

    init(background: Background,
         @ViewBuilder content: () -> Content) {
        self.background = background
        self.content = content()
    }

    var body: some View {
        HStack {
            content.syncingHeightIfLarger(than: $childHeight)
                   .frame(height: childHeight)
                   .background(background)
        }
    }
}
```

- 문제 해결을 위해 background를 `EventInfoBadge`에서 `HeightSyncedRow`로 이동시킵니다.
- 그렇게 하면 각 뷰에 frame을 지정한 후 background를 지정할 수 있게 됩니다. 그러면 모든 background view에 올바른 크기가 제공됩니다.
- `HeightSyncedRow`를 재사용성이 높은 컴포넌트로 사용하려면 `Background`를 주입할 수 있도록 초기화 구문에 background를 추가합니다.


<br/>

<br/>

```swift
struct EventInfoList: View {
    var body: some View {
        HeightSyncedRow(background: Color.secondary.cornerRadius(10)) {
            EventInfoBadge(
                iconName: "video.circle.fill",
                text: "Video call available"
            )
            EventInfoBadge(
                iconName: "doc.text.fill",
                text: "Files are attached"
            )
            EventInfoBadge(
                iconName: "person.crop.circle.badge.plus",
                text: "Invites enabled, 5 people maximum"
            )
        }
    }
}
```

<img width="260" alt="CleanShot 2023-01-21 at 15 38 47@2x" src="https://user-images.githubusercontent.com/42647277/213847261-392553b3-53b4-4fe0-9fa7-828eb1e2f63d.png">

- `EventInfoList`로 되돌아간 후 `HeightSyncedRow`를 생성할 때 background view를 전달하도록 수정합니다.

<br/>

<br/>

```swift
struct EventInfoBadge: View {
    var iconName: String
    var text: String

    var body: some View {
        VStack {
            Image(systemName: iconName)
                .resizable()
                .aspectRatio(contentMode: .fit)
                .frame(width: 25, height: 25)
            Text(text)
                .frame(maxWidth: .infinity)
                .multilineTextAlignment(.center)
        }
        .padding(.vertical, 10)
        .padding(.horizontal, 5)
    }
}
```

<img width="260" alt="CleanShot 2023-01-21 at 15 39 39@2x" src="https://user-images.githubusercontent.com/42647277/213847293-23fb49e0-42ee-4591-bb07-22a1b84e7f71.png">

- 이제 `EventInfoBadge`에 남아있던 background를 제거하면 구현은 끝입니다.

<br/>

- 이번 연습에서 본질적으로 살펴보아야 할 점은 바로 layout dependencies입니다. 구체적으로 한 view의 layout이 다른 view에게 의존하는 경우입니다.
- 위에서 살펴본 것은 자식 뷰들 중 가장 큰 높이를 가진 뷰를 알기 전까지는 각 `EventInfoBadge`의 높이를 결정할 수 없었습니다.
- view들 간의 강한 결합성을 지양하기에 layout dependencies는 가급적 피해야 하지만 때로는 하위 view와 상위 view 사이에 커뮤니케이션을 위한 체인을 설정해야 할 필요도 있습니다.
- `HeightSyncedRow`를 구현한 것처럼 generic하게 추상화를 통해 이를 수행할 수 있는 경우 여전히 모듈화하고 변경하기 쉬운 방식으로 layout dependencies를 관리하는 방법을 찾을 수 있습니다.

<br/>

## 요약

- `.aspectRatio()` modifier는 뷰의 크기가 resize될 때 뷰의 content를 어떻게 조정할지 정할 수 있습니다. 특히 이미지를 다룰 때 사용합니다.
- `.frame(maxWidth: .infinity)`는 뷰가 가능한 많은 공간을 차지하도록 지시할 수 있습니다. 그리고 부모 뷰가 사용 가능한 공간에 대해서 자식 뷰들에게 균등하게 나누도록 '강제'하는데 사용할 수 있습니다.
- `GeometryReader`는 주변 geometry를 읽고 그에 따라 커스텀 layout을 빌드할 수 있게 해주는 특별한 view입니다.
- 스유의 Preferences system을 이용하여 자식 뷰의 계산된 사이즈를 부모 뷰에게 알리는 예시처럼 뷰 계층 구조를 통해 위로 알릴 수 있습니다.



## 출처

- https://www.swiftbysundell.com/articles/swiftui-layout-system-guide-part-2/



