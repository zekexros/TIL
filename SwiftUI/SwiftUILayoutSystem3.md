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

<img width="260" alt="CleanShot 2023-01-23 at 01 04 02@2x" src="https://user-images.githubusercontent.com/42647277/213926025-7f1ffafc-bd48-49be-aa67-88f1e476592b.png">

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

<img width="260" alt="CleanShot 2023-01-23 at 01 04 35@2x" src="https://user-images.githubusercontent.com/42647277/213926053-cdd43b29-9ff4-4848-8c1c-dadb58bc7c3b.png">

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

<img width="260" alt="CleanShot 2023-01-23 at 01 07 03@2x" src="https://user-images.githubusercontent.com/42647277/213926174-7358cfbe-ecc8-4c46-94d8-7dde9999f3fb.png">

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

<img width="260" alt="CleanShot 2023-01-23 at 01 07 46@2x" src="https://user-images.githubusercontent.com/42647277/213926211-55d46448-6f71-4d8b-be49-161559ef421c.png">

- layout priorities는 SwiftUI 레이아웃 시스템에게 어떤 뷰가 가장 중요한지 표시하는 것을 가능하게 합니다.
- 각 뷰는 우선순위 0부터 시작하며 `layoutPriority()` modifier를 적용하여 순위를 조절할 수 있습니다(0보다 크면 레이아웃에 영향을 미치니 999 혹은 .infinity같은 값을 사용할 필요는 없습니다).
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

<img width="260" alt="CleanShot 2023-01-23 at 01 09 29@2x" src="https://user-images.githubusercontent.com/42647277/213926277-c9f3f08d-551b-4ce6-8b48-6cc45957108e.png">

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

<img width="260" alt="CleanShot 2023-01-23 at 01 10 19@2x" src="https://user-images.githubusercontent.com/42647277/213926315-e907b484-a92b-4e53-bad6-a2a5869307a3.png">

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

<img width="260" alt="CleanShot 2023-01-23 at 01 14 52@2x" src="https://user-images.githubusercontent.com/42647277/213926512-58d9b3cf-de40-4abb-85f9-7e7c1eee022e.png">

- placeholer와 description 모두 보기 좋아졌지만 `EventInfoBadge`의 텍스트는 여전히 잘렸습니다. 이 문제를 해결하기 위해 `EventInfoList`의 priority를 올려 레이아웃 시스템에게 다른 뷰들보다 높이를 우선시 하도록 구현합니다.
- SwiftUI의 layout priority 시스템은 뷰의 배치에 대해 명시적으로 순서를 지정할 수 있는 간단하고 강력한 도구입니다.
- 사용 가능한 공간에 맞게 뷰의 크기를 조정하는 방법으로서 뷰들 간의 충돌을 해결하는데 큰 도움이 됩니다.

<br/>

<br/>

## Fixed dimensions

- 그러나 layout priorities와 관련된 이슈가 하나 있습니다. 그것은 바로 '두더지 잡기' 게임 처럼 느껴질 수 있다는 것인데요, 즉 우리가 적용하는 모든 수정 사항에 대해 새로운 문제가 나타난다는 것입니다.
- 우리는 이미 위에서 문제가 나타난 것을 보았습니다(layout priority를 높이거나 낮출 때 발생).
- 따라서 layout의 priority를 조정하여 해결하는 것은 일회성으로는 좋은 방법이고 또 이게 유일한 방법은 아닙니다.
- 다른 방법은 바로 `fixedSize()` modifier입니다. 이것은 이름부터 보면 알 수 있듯이 뷰의 너비 혹은 높이(혹은 둘 다)에 대해 고정할 수 있습니다.

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
            EventInfoList().fixedSize(horizontal: false, vertical: true)
        }.padding()
    }
}
```

<img width="260" alt="CleanShot 2023-01-23 at 01 14 52@2x" src="https://user-images.githubusercontent.com/42647277/213926512-58d9b3cf-de40-4abb-85f9-7e7c1eee022e.png">

- 새로운 modifier를 사용하여 `EventInfoList`에 고정된 vertical 크기를 제공하여 압축되지 않도록 하여 이전 `layoutPriority`사용한 것과 동일한 결과를 얻을 수 있습니다.

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
            EventInfoList().fixedSize(horizontal: true, vertical: true)
        }.padding()
    }
}
```

<img width="260" alt="CleanShot 2023-01-23 at 01 18 26@2x" src="https://user-images.githubusercontent.com/42647277/213926681-142368ac-c6b3-4bf2-985f-777fecbec345.png">

- `fixedSize()`가 어떻게 작동하는지 더 자세히 살펴보기 위해 `EventInfoList`의 가로 크기에도 고정으로 변경하면 어떻게 되는지 살펴봅시다.
- `ContentView`의 크기가 화면 범위를 넘어서까지 늘어나는 모습으로 상당히 이상하게 나타납니다.
- 그 이유는 layout 시스템이 `EventInfoList`의 너비를 조정하는 것을 방지하고 있기 때문에 상위 `VStack`은 동일한 큰 너비를 차지하기 위해 자체적으로 확장되어야 합니다(스택은 항상 들어있는 자식 뷰들에 맞춰 resize하기 때문).
- 그래서 resize된 크기가 bounds를 넘어버릴지라도 하위 뷰들에게 좀 더 넓은 수평적인 공간을 제공합니다.

<br/>

<br/>

<br/>

## Custom alignment guides

- 마지막으로 살펴볼 정렬 방법은 padding 혹은 offset의 훌륭한 대안이 될 수 있습니다.

```swift
extension View {
    func addVerifiedBadge(_ isVerified: Bool) -> some View {
        ZStack(alignment: .topTrailing) {
            self

            if isVerified {
                Image(systemName: "checkmark.circle.fill")
                    .offset(x: 3, y: -3)
            }
        }
    }
}
```

<img width="79" alt="CleanShot 2023-01-23 at 01 21 05@2x" src="https://user-images.githubusercontent.com/42647277/213926795-236ec4e1-a56d-475d-a57f-54a10f7741be.png">

- 예제를 위해 시리즈 1편의 `verified badge`로 되돌아갑니다.
- View를 확장하여 `ZStack`과 `.offset()` modifier를 활용하여 구현하였습니다.

<br/>

<br/>

```swift
extension View {
    func addVerifiedBadge(_ isVerified: Bool) -> some View {
        ZStack(alignment: .topTrailing) {
            self

            if isVerified {
                Image(systemName: "checkmark.circle.fill")
                    .alignmentGuide(HorizontalAlignment.trailing) {
                        $0[HorizontalAlignment.center]
                    }
                    .alignmentGuide(VerticalAlignment.top) {
                        $0[VerticalAlignment.center]
                    }
            }
        }
    }
}
```

<img width="84" alt="CleanShot 2023-01-23 at 01 21 22@2x" src="https://user-images.githubusercontent.com/42647277/213926801-8f4911f6-b173-4f56-b2c2-12976ebe1244.png">

- 기존 확장의 문제점은 렌더링된 이미지의 사이즈를 고려하지 않고 고정적으로 3x3으로 하드코딩 된다는 점입니다.
- 이 문제를 해결하기 위해 `.offset()` modifier를 2개의 커스텀 가이드로 바꾸겠습니다.
- `.alignmentGuide()` modifier를 뷰에 적용하면 커스텀으로 계산된 클로저를 사용하여 수평 혹은 수직 정렬이 사용될 때 위치가 지정되는 방법을 조정할 수 있습니다.
- `ZStack`은 현재 `.topTrailing`정렬을 사용하므로 두 가이드에 따라 중심을 배치하여 뱃지 위치를 조정하겠습니다.
- 괜찮아보이지만 offset을 하드코딩한 것보다 좋아보이지는 않습니다.
- 기본적으로 뱃지 이미지를 달력 아이콘 중앙쪽으로 약간 더 이동시켜 호스트 뷰에 더 연결되어있는 느낌을 주고싶습니다.

<br/>

<br/>

```swift
extension View {
    func addVerifiedBadge(_ isVerified: Bool) -> some View {
        ZStack(alignment: .topTrailing) {
            self

            if isVerified {
                Image(systemName: "checkmark.circle.fill")
                    .alignmentGuide(HorizontalAlignment.trailing) {
                        $0.width * 0.8
                    }
                    .alignmentGuide(VerticalAlignment.top) {
                        // Here we first align our view's bottom edge
                        // according to its host view's top edge,
                        // and we then subtract 80% of its height.
                        $0[.bottom] - $0.height * 0.8
                    }
            }
        }
    }
}
```

<img width="78" alt="CleanShot 2023-01-23 at 01 22 01@2x" src="https://user-images.githubusercontent.com/42647277/213926832-e55fcd87-d733-4fa8-a5d9-047ec502fc06.png">

- 고정된 offset값을 사용하지 않는 방식으로 구현하려면 뱃지 이미지의 중심을 사용하는 대신 뱃지 이미지의 너비와 높이를 비율과 함께 사용하여 정렬합니다.
- 각 클로저에 전달되는 `ViewDimensions` 컨텍스트에는 뷰의 너비와 높이도 포함되어 있기 때문에 편리하게 구현 가능합니다.
- 유의할 점은 이전 오프셋 기반 접근 방식과는 달리 새로운 방식은 프레임 계산할 때 뱃지가 포함됩니다. 이 경우에는 큰 차이가 없습니다. 그리고 뱃지에게 낮은 layout priority를 부여하여 피할 수 있습니다.

<br/>

<br/>

```swift
extension View {
    func alignAsBadge(withRatio ratio: CGFloat = 0.8,
                      alignment: Alignment = .topTrailing) -> some View {
        alignmentGuide(alignment.horizontal) {
            $0.width * ratio
        }
        .alignmentGuide(alignment.vertical) {
            $0[.bottom] - $0.height * ratio
        }
    }
}
```

- 새로 배운 custom alignment방식은 매우 강력하지만 구문이 굉장히 깁니다. 따라서 위의 modifier를 새로운 확장자로 빼서 모든 view에 적용하도록 구현해보겠습니다.

<br/>

<br/>

```swift
extension View {
    func addVerifiedBadge(_ isVerified: Bool) -> some View {
        ZStack(alignment: .topTrailing) {
            self

            if isVerified {
                Image(systemName: "checkmark.circle.fill")
                    .alignAsBadge()
            }
        }
    }
}
```

<img width="78" alt="CleanShot 2023-01-23 at 01 22 01@2x" src="https://user-images.githubusercontent.com/42647277/213926832-e55fcd87-d733-4fa8-a5d9-047ec502fc06.png">

- 새로 확장된 custom alignment을 이용하여 verified badge를 위와 같이 구현할 수 있게 되었습니다.
- `.alignmentGuide()` modifier를 통해 가로 혹은 세로 정렬이 주어진 경우 뷰가 정렬되는 방식을 재정의 및 조정할 수 있습니다.

<br/>

<br/>

- 이로써 시리즈를 마무리하고 이번 시리즈들을 통해 SwiftUI의 layout system에 대해 새로운 통찰을 얻고 커스텀을 도와주는 다양한 API와 툴들을 알게되었습니다.
- 이 밖에도 더 많은 스유의 레이아웃 시스템들이 있는데 이후 하나씩 다루는 글들을 추가하겠습니다.

<br/>

<br/>

## 요약

- layout priorities는 각 뷰의 우선순위 사이즈를 조정하는 좋은 방법입니다.
- fixed frame size(고정 사이즈)는 가로나 세로의 길이에 대해 재조정하는 것을 방지하는 좋은 방법입니다.
- custom alignment guide를 사용하면 뷰가 배치되는 방식을 조정할 수 있습니다. 이는 한 뷰를 다른 뷰와 관련하여 배치할 때 유용합니다. 





<br/>




## 출처

- https://www.swiftbysundell.com/articles/swiftui-layout-system-guide-part-3/

