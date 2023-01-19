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

- 



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





<br/>

<br/>

## ZStacks and offset

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



<br/>

<br/>





## 출처

https://www.swiftbysundell.com/articles/swiftui-layout-system-guide-part-1/