# SwiftUI Layout System 3

- 이번 3장에서는 다양한 커스텀 옵션에 대해 알아보고 SwiftUI 레이아웃을 정의할 때 나타나는 일반적인 충돌을 해결하고 애매모호한 것을 없애는 방법을 살펴봅니다.



## Encountering conflicts

- 2장에서 만들었던 View들을 그대로 이어서 예제를 살펴봅시다.



```swift
struct ImagePlaceholder: View {
    var body: some View {
        ZStack {
            RoundedRectangle(cornerRadius: 10).stroke()
            Text("Image placeholder")
        }
    }
}
```

- `ZStack`안에 `Text`와 함께 `RoundedRectangle`을 이용하여 이미지를 위한 placeholder를 렌더링하는 새로운 뷰를 만들어줍니다.

<br/>

<br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            EventHeader()
            ImagePlaceholder()
            Text("This is a description")
            Spacer()
            EventInfoList()
        }.padding()
    }
}
```

- `ContentView`에 설명하는 텍스트와 함께 `ImagePlaceHolder` 인스턴스를 추가해 줍니다.
- 위 코드의 결과는 SwiftUI의 다양한 모양의 흥미로운 면을 보여줍니다.
- Spacer는 항상 가능한 많은 공간을 차지하므로 description은 굉장히 짧아지고 image placeholder는 스크린의 상당 부분을 차지하기 위해 스스로 늘어나게 됩니다.

<br/>

<br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            EventHeader()
            ImagePlaceholder()
            Text(makeDescription())
            Spacer()
            EventInfoList()
        }.padding()
    }
}

private extension ContentView {
    func makeDescription() -> String {
        String(repeating: "This is a description ", count: 50)
    }
}
```

- Description을 좀 더 길게 바꾸면 어떻게 될지 살펴봅시다.
- SwiftUI 레이아웃 시스템은 긴 텍스트를 자를 뿐 아니라 화면 하단의 `EventInfoBadge`의 텍스트도 자릅니다.
- 여전히 사용 가능한 공간의 상당 부분을  `ImagePlaceholder`에게 제공하는 모습입니다.
- 모든 것은 1부에서 살펴본 SwiftUI의 레이아웃 시스템으로 작동합니다. 각 뷰는 자신의 크기를 결정해야 하는 책임이 있으며 그 후에 각 부모가 자신들의 프레임 안에 어떻게 자식들을 배치해야 할지 결정합니다.
- 결과적으로 `ImagePlaceholder`와 descrption text는 `VStack`에 동시에 들어갈 수 있는 것보다 훨씬 더 큰 프레임을 요청합니다.
- 레이아웃 시스템은 먼저 각 뷰를 가능한 많이 압축하고(이 때문에 `EventInfoBadge`가 잘림) 사용 가능한 공간을 자식 뷰들에게 균등하게 분배하여 타협해야 합니다.
- 고마운 점은 SwiftUI는 위와 같은 종류의 레이아웃 충돌을 해결하는데 사용하는 여러 도구들을 제공합니다. 각 뷰를 수동으로 직접 그리거나 UIKit을 사용하는 방법은 필요 없습니다.

<br/>

<br/>

## Layout priorities

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            EventHeader()
            ImagePlaceholder()
            Text(makeDescription()).layoutPriority(1)
            Spacer()
            EventInfoList()
        }.padding()
    }
}
```

- layout priorities는 SwiftUI 레이아웃 시스템에게 어떤 뷰가 가장 중요한지 표시하는 것을 가능하게 합니다.
- 각 뷰는 우선순위 0부터 시작하며 `layoutPriority()` modifier를 적용하여 순위를 조절할 수 있습니다(0보다 크면 레이아웃에 영향을 미치니 999 혹은 .infinity같은 값을 사용할 필요는 없습니다).

<br/>

<br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            EventHeader()
            ImagePlaceholder().layoutPriority(-1)
            Text(makeDescription())
            Spacer()
            EventInfoList()
        }.padding()
    }
}
```

- 설명하는 뷰가 좀 더 길어진 모습으로 그 전보다 한결 나아졌습니다.
- 그러나 하단의 마지막  `EventInfoBadge`는 여전히 찌그러졌습니다. 그리고 `ImagePlaceHolder`는 높이가 더 작아졌습니다.

<br/>

<br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            EventHeader()
            ImagePlaceholder().layoutPriority(-1)
            Text(makeDescription())
            Spacer()
            EventInfoList()
        }.padding()
    }
}
```

- `EventInfoBadge`의 문제를 해결하는 방법중 하나는 위에서 한 방법을 거꾸로 하는 것입니다. image placeholder의 priority를 낮추는 것이죠.

<br/>

<br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            EventHeader()
            ImagePlaceholder()
                .layoutPriority(-1)
                .frame(minHeight: 100)
            Text(makeDescription())
            Spacer()
            EventInfoList()
        }.padding()
    }
}
```

- 우선순위를 낮추며 한결 더 나아졌지만 image placeholder는 여전히 가장 작은 높이로 줄어들어 보기 좋지 않습니다.
- 문제를 해결하기 위해 placeholder에게 `.frame()` modifier를 사용하여 최소한의 높이를 지정하겠습니다.

<br/>

<br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            EventHeader()
            ImagePlaceholder()
                .layoutPriority(-1)
                .frame(minHeight: 100)
            Text(makeDescription())
            Spacer()
            EventInfoList().layoutPriority(1)
        }.padding()
    }
}
```

- placeholer와 description 모두 보기 좋아졌지만 `EventInfoBadge`의 텍스트는 여전히 잘렸습니다. 이 문제를 해결하기 위해 `EventInfoList`의 priority를 올려 레이아웃 시스템에게 다른 뷰들보다 높이를 우선시 하도록 구현합니다.
- SwiftUI의 layout priority 시스템은 뷰의 배치에 대해 명시적으로 순서를 지정할 수 있는 간단하고 강력한 도구입니다.
- 사용 가능한 공간에 맞게 뷰의 크기를 조정하는 방법으로서 뷰들 간의 충돌을 해결하는데 큰 도움이 됩니다.

<br/>

<br/>

## Fixed dimensions

```swift
```



<br/>

<br/>

```swift
```



<br/>

<br/>

```swift
```

```swift
```

```swift
```




## 출처

- https://www.swiftbysundell.com/articles/swiftui-layout-system-guide-part-3/

