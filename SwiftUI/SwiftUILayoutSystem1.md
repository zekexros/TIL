# SwiftUI Layout System 1

- SwiftUI는 <u>오토레이아웃의 적응성</u>과 <u>명시적으로 프레임을 수동 계산하는 것</u>을 결합한 새로운 레이아웃 시스템을 갖고있습니다. 
- 처음보기엔 단순해보이지만, 다양한 구성 요소들을 정교한 레이아웃에 결합하기 시작하면 엄청난 유연성과 성능을 제공하는 시스템입니다.
- 예제를 통해 SwiftUI 레이아웃 시스템의 기본 규칙에 대해 알아보고 각 규칙은 또 어떻게 관련되어 있는지 살펴보겠습니다.

<br/>

## Setting a view's frame

- 애플이 기본적으로 제공하는 SFSymbol 이미지를 이용하여 간단한 캘린더 이미지를 나타내는 ContenView와 함께 시작합니다.

```swift
struct ContentView: View {
    var body: some View {
        Image(systemName: "calendar")
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 11 54 45@2x" src="https://user-images.githubusercontent.com/42647277/213608161-6d2cb49d-5236-4b50-b84d-29072a60c72b.png">

- 기본적으로 SwiftUI는 각 뷰가 렌더링된 컨테이너를 기준으로 자체 크기를 선택한 다음 부모 뷰 중앙에 배치됩니다.
- 따라서 위의 코드는 스크린의 중앙에 작은 아이콘으로 렌더링 됩니다. **<span style="color:orange">UIKit처럼 왼쪽 상단에 배치되는 것이 아닙니다.</span>**

<br/>

- 그리고 아이콘의 크기를 50x50 points로 늘려봅시다. 우선 `.frame()` modifier를 이용하여 늘려볼까요.

```swift
struct ContentView: View {
    var body: some View {
        Image(systemName: "calendar")
            .frame(width: 50, height: 50)
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 11 54 45@2x" src="https://user-images.githubusercontent.com/42647277/213608161-6d2cb49d-5236-4b50-b84d-29072a60c72b.png">

- view의 크기가 50x50 points로 커지지만 아이콘의 크기는 달라지지 않았습니다.
- 이유를 찾기 위해 백그라운드에 색을 지정하여 frame을 살펴봅시다.

<br/>

```swift
struct ContentView: View {
    var body: some View {
        Image(systemName: "calendar")
            .frame(width: 50, height: 50)
            .background(Color.red)
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 00 41@2x" src="https://user-images.githubusercontent.com/42647277/213608801-9cf817a2-ea36-4d29-95f0-368c286d2d9e.png">

- 색을 지정해보니 view의 크기는 정상적으로 50x50 points로 늘어나있습니다. 단지 아이콘이 `.frame()` modifier의 영향을 받지 않은 것 같군요.
- view에 modifier를 적용시킬 때 view가 바뀌지 않고 새롭고 투명한 view로 캡슐화하는 경우가 종종 있습니다.
- 따라서 위의 `background()`를 호출할 때는 이미지 자체가 아닌 이미지를 감싸는 새로운 view에 modifier가 적용됩니다.

<br/>

```swift
struct ContentView: View {
    var body: some View {
        Image(systemName: "calendar")
            .resizable()
            .frame(width: 50, height: 50)
            .background(Color.red)
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 01 34@2x" src="https://user-images.githubusercontent.com/42647277/213608898-f746efc9-1038-439c-82d4-cf32a17b249d.png">

- 레이아웃 관점에서 이미지 자체는 여전히 달라지지 않습니다.
- **<span style="color:orange">SwiftUI의 view는 자기 자신의 사이즈를 결정할 책임</span>**이 있으므로 우리는 이미지에게 가용할 수 있는 공간을 모두 사용하도록 resize하라고 말해야 합니다.
- 이는 간단하게 `.resizeable()` modifier를 적용시키면 됩니다.

<br/>

<br/>



## Applying padding

- SwiftUI에서 padding이 어떻게 작동하는지 살펴봅시다.
- 다른 레이아웃 시스템 이를테면 CSS는 padding을 통해 frame안에서 view의 내용을 offset이 가능합니다. 
- SwiftUI에서 padding modifier를 적용하는 위치에 따라 다른 결과를 도출합니다.

<br/>

```swift
struct ContentView: View {
    var body: some View {
        Image(systemName: "calendar")
            .resizable()
            .frame(width: 50, height: 50)
            .background(Color.red)
            .padding()
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 01 34@2x" src="https://user-images.githubusercontent.com/42647277/213608898-f746efc9-1038-439c-82d4-cf32a17b249d.png">

- `.padding()`modifier적용으로 달력 아이콘 바깥쪽에 패딩이 생길거라는 예상과는 달리 바뀐게 없습니다.
- 잘보면 이 동작은 위에 `.frame()`을 적용했을 때와 같은 동작입니다. `.padding()`은 이전 view와 modifier들을 바꾸는게 아니라 단지 이전 결과에 공백을 추가할 뿐입니다.

<br/>

```swift
struct ContentView: View {
    var body: some View {
        Image(systemName: "calendar")
            .resizable()
            .frame(width: 50, height: 50)
            .background(Color.red)
            .padding()
            .background(Color.blue)
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 02 45@2x" src="https://user-images.githubusercontent.com/42647277/213609002-bf69ddf1-e9e7-408a-bbe1-f12f03437e54.png">

- 여기에 다시 `.background()` modifier를 추가하면 어떻게 동작하는지 쉽게 파악이 될 것입니다.

<br/>

```swift
struct ContentView: View {
    var body: some View {
        Image(systemName: "calendar")
            .resizable()
            .frame(width: 50, height: 50)
            .padding()
            .background(Color.red)
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 04 07@2x" src="https://user-images.githubusercontent.com/42647277/213609145-14b24b77-20b4-4c72-aba4-4fa606d22489.png">

- 따라서 view의 background를 고려한 inner padding을 추가하려면 background를 추가하기 전에 padding을 적용해야 합니다.

<br/>

```swift
struct ContentView: View {
    var body: some View {
        Image(systemName: "calendar")
            .resizable()
            .padding()
            .frame(width: 50, height: 50)
            .background(Color.red)
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 04 54@2x" src="https://user-images.githubusercontent.com/42647277/213609210-ec22bdaf-0774-4cea-aa1b-c16b68b6565b.png">

- 만약 `.frame()` modifier를 적용하기 전에 `.padding()` modifier를 적용한다면 아이콘의 이미지는 축소될 것입니다.
- 이유는 padding이 <u>고정된 50x50 컨테이너</u> 내에 적용될 것이기 때문입니다.

<br/>

<br/>

## Stacks and spacers

- SwiftUI의 다양한 stack와 spacer들은 처음 볼 땐 단순하고 제한적으로 보이지만 실제로는 거의 무한한 레이아웃 조합을 표현하는데 사용될 수 있습니다.
- 살펴보기 위해 위에서 만든 View를 vertical stack으로 감싸보겠습니다. (위에서 만든 view를 CalendarView로 가정)

<br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            CalendarView()
        }
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 06 47@2x" src="https://user-images.githubusercontent.com/42647277/213609388-667e1098-494c-4dc6-9899-6a6f5cbf3ad4.png">

- 흥미로운 점은 위의 `VStack`은 레이아웃에 어떤 영향도 주지 않습니다.
- 대신 스택안에 있는 자식 뷰들의 전체 크기에 따라 resize될 뿐이죠.

<br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            CalendarView()
            Spacer()
        }
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 07 03@2x" src="https://user-images.githubusercontent.com/42647277/213609417-e63a1cea-1e8a-4fca-b5b3-56e974d8cc39.png">

- `CalendarView`를 이동시키기 위해 스택에 `Spacer`를 추가해야만 합니다.
- `HStack` 혹은 `VStack`안에 `Spacer`를 배치하면 항상 가용할 수 있는 최대의 공간을 차지합니다. 여기선 `CalendarView`가 화면의 위쪽으로 밀리겠네요. 

<br/>

```swift
struct ContentView: View {
    var body: some View {
        HStack {
            VStack {
                CalendarView()
                Spacer()
            }
            Spacer()
        }.padding()
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 07 20@2x" src="https://user-images.githubusercontent.com/42647277/213609444-2a5c2e9c-1f8f-47b9-bfd5-a385bdc94d14.png">

- 스택의 아주 좋은 점은 어떠한 수동적인 프레임 계산없이 복잡한 레이아웃을 표현하기 위해 스택을 **<span style="color:orange">중첩</span>**할 수 있다는 것입니다.
- 위의 코드는 `VStack`과 `HStack`을 중첩시켜 화면의 top-leading쪽으로 배치시킨 모습입니다(약간의 Spacer의 도움과 함께).

<br/>

```swift
struct ContentView: View {
    var body: some View {
        HStack {
            VStack {
                CalendarView()
                Spacer()
            }
            Text("Event title").font(.title)
            Spacer()
        }.padding()
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 13 58@2x" src="https://user-images.githubusercontent.com/42647277/213610104-1a30ec66-09bf-427c-a8ef-a2b3b83cf7a7.png">

- 다음으로 뷰에 텍스트를 추가해 보겠습니다.
- 위의 코드를 보면 `Text`는 `CalendarView` 바로 옆에 렌더링 될 것으로 예상할 수 있습니다. 수평축에서는 사실이지만 수직축에서는 전체 높이를 기준으로 중앙에 배치됩니다.
- 그 이유는 `Spacer`가 `CalendarView`를 포함하고있는 `VStack`에만 영향을 주기 때문입니다.

<br/>

```swift
struct ContentView: View {
    var body: some View {
        HStack(alignment: .top) {
            VStack {
                CalendarView()
                Spacer()
            }
            Text("Event title").font(.title)
            Spacer()
        }.padding()
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 07 52@2x" src="https://user-images.githubusercontent.com/42647277/213609508-41299a61-ed73-4000-b18d-4f8876592455.png">

- `Text`도 `CalendarView`와 같은 레이아웃을 가지려면 `VStack`으로 감싸야 하거나 `HStack`의 align을 조정하면 됩니다.

<br/>

```swift
struct ContentView: View {
    var body: some View {
        HStack(alignment: .top) {
            VStack {
                CalendarView()
                Spacer()
            }
            VStack(alignment: .leading) {
                Text("Event title").font(.title)
                Text("Location")
            }
            Spacer()
        }.padding()
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 08 15@2x" src="https://user-images.githubusercontent.com/42647277/213609542-90d1d25a-1a6a-4b61-948c-0259fad870fe.png">

- Event title 아래에 location 텍스트를 렌더링하기 위해 `VStack`이 배치 방식을 조정할 수도 있습니다.

<br/>

```swift
struct EventHeader: View {
    var body: some View {
        HStack(spacing: 15) {
            CalendarView()
            VStack(alignment: .leading) {
                Text("Event title").font(.title)
                Text("Location")
            }
            Spacer()
        }
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 08 45@2x" src="https://user-images.githubusercontent.com/42647277/213609594-59175406-357f-46db-a807-3abc57c0f7ed.png">

- 여태 만든 View를 `EventHeader`로 추출했습니다. 그리고 `HStack`의 자식 뷰들간의 spacing을 15로 지정하였습니다.

<br/>

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            EventHeader()
            Spacer()
        }.padding()
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 09 17@2x" src="https://user-images.githubusercontent.com/42647277/213609646-ea0fb43d-1a10-40be-a7a9-49f1e0186988.png">

- 상위 뷰인 `ContentView`로 되돌아가 `EventHeader`를 spacer와 함께 하나의 `VStack`에 포함시켜 구성할 수 있습니다.
- 가능하다면 계속해서 작은 뷰들로 추출하는 원칙을 따를겁니다. 이러한 방식으로 작업하면 많은 아키텍처 디자인 작업을 미리 수행할 필요 없이 UI를 작은 단위로 자연스럽게 분리할 수 있습니다.



<br/>

<br/>

<br/>

## ZStacks and offset

- view들을 뒤에서 앞으로 순서로 깊이를 가지도록 쌓을 수 있는 `ZStack`에 대해 알아봅시다.

<br/>

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

- 이전에 만든 캘린더 뷰 위에 작은 뱃지를 표시하기 위해 코드를 작성할 것입니다.
- 좀 더 제네릭하게 구현하기 위해 `ZStack`내에서 모든 뷰를 감쌀 수 있는 API로 확장하였습니다.
- `ZStack`이 alignment에 대해 완전한 2차원 제어를 제공하는 방법을 잘 살펴보세요. 아이콘의 위치를 부모 뷰의 top-trailing 코너에 배치할 수 있습니다.
- 그리고 `.offset()`modifier를 적용하여 부모 뷰의 bounds밖으로 약간 이동합니다.

<br/>

```swift
struct CalendarView: View {
    var eventIsVerified = true

    var body: some View {
        Image(systemName: "calendar")
            .resizable()
            .frame(width: 50, height: 50)
            .padding()
            .background(Color.red)
            .cornerRadius(10)
            .foregroundColor(.white)
            .addVerifiedBadge(eventIsVerified)
    }
}
```

<img width="260" alt="CleanShot 2023-01-20 at 12 17 03@2x" src="https://user-images.githubusercontent.com/42647277/213610417-2f1b57dd-622d-42bc-9a19-723b0680d38e.png">

- 이제 eventIsVerified라는 프로퍼티를 통해 `CalendarView`의 배지를 조건적으로 구현할 수 있게 되었습니다.
- `.offset()`과 `ZStack`을 함께 사용하는 것은 view 자체의 레이아웃에 전혀 영향을 주지 않고 다양한 종류의 오버레이를 추가하는 좋은 방법이 될 수 있습니다.
- 이 기술을 통해 로딩 인디케이터, 인앱 알림 등 기존 뷰 위에 다양한 뷰들을 렌더링하도록 구현할 수 있습니다.

<br/>

<br/>

## 요약

- SwiftUI의 core layout engine은 각 자식 뷰에게 부모의 bound를 기반으로 자체 크기를 결정하도록 요청한 다음 각 부모에게 bound안에 자식을 배치하도록 요청합니다.
- view modifier의 호출 순서에 따라 완전히 다른 레이아웃 결과를 얻을 수 있음에 유의해야 합니다.
- `.frame()`과 `.padding()`을 사용하면 뷰의 크기와 내부 마진을 조정할 수 있습니다.
- `HStack`, `VStack`, `ZStack`을 이용하여 뷰들을 수평, 수직, 깊이있게 쌓을 수 있습니다.
- `offset()`은 주변 환경에 영향을 주지 않고 뷰를 이동할 수 있으므로 오버레이를 구현할 때 매우 유용합니다.



## 출처

https://www.swiftbysundell.com/articles/swiftui-layout-system-guide-part-1/
