# Value & Refence Types

- Swift의 타입들은 둘 중 하나입니다. 
  - 인스턴스가 데이터의 고유한 복사본을 유지하는 값(value) 타입 - `struct`, `enum`, tuple
  - 인스턴스가 데이터의 단일 복사본을 공유하는 참조(reference) 타입 - `class`
- 이번 내용에서는 값과 참조타입 각각의 장점과 선택하는 방법에 대해 알아봅니다.

<br/>

## What's the difference?

- 값 타입의 가장 기본적인 특징은 <u>복사</u>입니다. 구체적으로 할당, 초기화, 인수로 전달될 때 복사가 일어납니다.
- 고유한 데이터의 사본(copy)을 이용하여 독립적인 인스턴스를 생성합니다.

```swift
// Value type example
struct S { var data: Int = -1 }
var a = S()
var b = a						// a is copied to b
a.data = 42						// Changes a, not b
println("\(a.data), \(b.data)")	// prints "42, -1"
```

<br/>

<br/>

- 반면 참조를 복사하면 암시적으로 공유된 인스턴스가 생성됩니다.
- 복사한 후 두개의 변수는 하나의 인스턴스를 참조하므로 두 번째 변수의 데이터를 변경하는 것은 첫 번째 변수에게도 영향을 미칩니다.

```swift
// Reference type example
class C { var data: Int = -1 }
var x = C()
var y = x						// x is copied to y
x.data = 42						// changes the instance referred to by x (and y)
println("\(x.data), \(y.data)")	// prints "42, 42"
```

<br/>

## The Role of Mutation in Safety

- 참조 타입이 아닌 값 타입을 선택하는 가장 큰 이유는 쉽게 추론할 수 있다는 점입니다.
- 만약 복사한 인스턴스가 항상 고유한 값이라면 앱의 어느 부분에서도 데이터를 몰래 변경하지 않는다고 확신할 수 있을 것입니다.
- 이것은 특히 멀티스레드 환경에서 서로 다른 스레드가 데이터를 다룰 때 굉장히 유용합니다.

<br/>

- 인스턴스가 writable data를 갖지 않을 때, 변형이 일어날 일이 없기에 value와 reference는 같은 방식으로 작동합니다.
  - write할 때 복사, 참조에 따라 주의깊게 사용하지만 그냥 값을 읽기만 한다면 뭔들 상관없다.
- 그러면 `class`가 불변인 경우를 갖는 것이 더 가치있다고 생각할 수 있습니다.(복잡한 객체 그래프를 갖는다거나 상속을 이용하거나 dynamic dispatch를 이용하여 다양한 클래스들을 이용하다거나 하는 등의 이점들로 인해 더 가치있을 수 있음)
- 이렇게 하면 value의 이점은 유지하면서 Cocoa의 `NSObject`객체들을 더 쉽게 사용할 수 있습니다.
- 오늘날에는 변경 불가능한 저장 프로퍼티들과 state를 변경할 수 있는 API의 노출을 피하면 Swift에서 변경 불가능한 클래스를 작성할 수 있습니다.
- 그리고 사실, `NSURL`과 같은 많은 Cocoa의 class들이 변경할 수 없는 class들로 설계되었습니다.
- 스위프트는 structs나 enums에 대해 불변성을 적용하는 메커니즘이 있지만, classes에는 없습니다. 예를 들어, 불변성을 가진 struct나 enum을 생성하면 스위프트는 어떠한 변경도 방지해줄 수 있지만 불변성을 가진 class를 생성하면 스위프트에서는 변경을 방지해줄 어떠한 것도 없습니다.
- 그러므로 불변한 class를 생성하면 계속해서 불변한 상태로 유지하고 변경하지 않도록 하는 것은 나와 내 팀에게 달려있습니다.





<br/>

## How to choose?

- Cocoa와 관련된 새로운 타입을 생성할 때는 많은 API들이 `NSObject`의 하위 클래스이므로 `class`를 사용해야만 합니다. 그게 아닌 다른 경우에는 아래와 같은 가이드 라인을 따르면 됩니다.

<br/>

- Value 타입을 사용할 때
  - 인스턴스 데이터를 `==`로 비교하는 것이 적합할 때
  - 독립적인 state를 갖도록 복사하고 싶을 때
  - 데이터가 여러 스레드에서 사용될 때

<br/>

- Reference 타입을 사용할 때
  - 인스턴스의 identity를 `===`로 비교하는 것이 적합할 때
  - 공유 및 변경 가능한 state를 생성하고자 할 때

<br/>

<br/>

- 스위프트의  `Array`, `String`, `Dictionary`는 value 타입입니다.
- 이들은 C언어의 간단한 `int`처럼 동작하며 해당 데이터의 고유한 인스턴스 역할을 합니다.
- 뒤에서 다른 코드가 해당 데이터를 수정하지 못하도록 명시적으로 복사본 만들기 같은 특별한 작업을 수행할 필요가 없습니다.
  - 이 말은 메소드에 인자로 전달하거나 코드의 다른 부분에서 사용하기 위해 인위적으로 `let a = b`처럼 새로 만들 필요가 없다는 뜻입니다. 왜냐하면 스위프트에 의해 자동적으로 복사되어 전달되기 때문입니다.
- 중요한 것은, 동기화 없이 스레드 간에 값의 복사본을 안전하게 전달할 수 있다는 것입니다.



<br/>

<br/>

|                       Memory Segments                        |                     Multi-thread diagram                     |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![memory-segments](https://user-images.githubusercontent.com/42647277/216049527-885922a3-4887-40d9-82a0-cb9bb3007779.svg) | ![image](https://user-images.githubusercontent.com/42647277/216049424-a39bcc39-3a93-465d-8da1-cd1de5262081.gif) |

## Memory Segments

- 메모리는 단지 긴 목록을 가진 바이트(bytes)입니다. 바이트는 순서대로 정렬되어있고, 모든 바이트는 고유한 주소를 가집니다.
- 개별 주소의 범위를 address space라고 합니다.
- iOS 앱의 address space는 논리적으로 text, data, stack, heap 이 네 가지 세그먼트로 구성됩니다.

<br/>

### Text Segment

- Text segment는 앱의 실행 가능한 코드를 구성하는 명령어가 포함되어 있습니다.
- 컴파일러에 의해 Swift를 기계어로 변환하며 생성됩니다.
- 이 세그먼트는 읽기 전용이며 일정한 공간을 차지합니다.

<br/>

### Data Segment

- Swift의 static variables, constants 및 type의 metadata를 저장합니다.
- 프로그램이 시작될 때 초기 값이 필요한 모든 전역 데이터는 여기에 존재합니다.

```swift
class SomeClass {
    static let sharedInstance = SomeClass()
    static var count = 0
}
```

<br/>

### Stack Segment

- 스택은 메서드의 파라미터나 지역 변수와 같은 임시 데이터들을 저장합니다.
- 메서드를 호출할 때마다 스택에 새로운 메모리가 할당됩니다.
- 이 메모리는 메서드가 종료될 때 해제됩니다.
- 일부 예외를 제외하고는 모든 Swift의 value타입들이 여기에 저장됩니다.(일부는 아래에서 설명할 예정)

<br/>

### Heap Segment

- 힙은 수명을 갖는 객체를 저장합니다. 이 객체들은 모두 스위프트의 reference타입이고 value타입 중 몇몇도 여기에 저장됩니다.(아래에서 설명할 예정)
- 힙과 스택은 서로를 향해 증가합니다.



### Value type이지만 heap에 저장되는 경우(with chatGPT)

- In Swift, value types (such as structures and enumerations) are usually stored on the stack, while reference types (such as classes) are stored on the heap. However, there are some cases where value types may be stored on the heap if they need to have a longer lifetime or if they need to be passed as a reference.

- For example, if a value type is stored as a property of a reference type or is passed as an inout parameter to a function, it will be stored on the heap. This allows the value type to persist beyond the lifetime of the function call and enables it to be modified by reference.

- Here's an example to illustrate this concept:

```swift
class SomeClass {
    var someValueType: SomeValueType
    
    init(someValueType: SomeValueType) {
        self.someValueType = someValueType
    }
}

struct SomeValueType {
    var value: Int
}

var valueType = SomeValueType(value: 42)
var referenceType = SomeClass(someValueType: valueType)
valueType.value = 10
print(referenceType.someValueType.value)  // prints 10
```

- In this example, `valueType` is a value type stored on the stack, but when it is assigned as a property of `referenceType`, a reference type, it is stored on the heap. This allows the value of `valueType` to be updated and observed through the `referenceType` instance.



## Cost of Heap vs Stack Allocation

- 스택 메모리 세그먼트는 우리가 아는 자료 구조인 스택 처럼 작동합니다. 스택 상단에 푸쉬하거나 팝하는 것을 말합니다.
- 스택 상단의 하나의 포인터는 푸쉬나 팝 작업을 하기에 충분합니다.
- 따라서 공간을 만들기 위해 스택 포인터를 줄이는 것만으로 필요한 메모리를 할당할 수 있습니다.
- 메서드가 끝나면 스택 포인터를 메서드 호출하기 전의 위치로 증가시킵니다.

<br/>

- 힙 할당에는 더 많은 관련이 있습니다. 적절한 크기의 빈 메모리 블록을 찾기 위해 힙 데이터 구조를 검색해야 합니다.
- 또한 여러 스레드가 동시에 메모리를 할당할 수 있으므로 힙을 동기화 해야합니다.
- 힙에서 메모리 할당을 해제하려면 해당 메모리를 다시 적절한 위치에 다시 삽입해야 합니다.
  - 스위프트는 ARC에 의해 RC를 계산하여 더 이상 필요하지 않을 때 자동으로 메모리의 할당을 해제합니다. 그리고 메모리를 다시 힙에 삽입하여 향후 할당에 재사용할 수 있도록 합니다.

<br/>

- 값과 참조 타입은 일반적으로 각각 스택과 힙에 할당되지만 이러한 규칙에는 고려해야 할 예외가 있습니다.

<br/>

<br/>

## Stack Promotion of Swift Reference Types

- Swift의 참조 타입이 힙에 저장되면 스택에 직접 저장되는 값 타입과 달리 참조를 저장하기 위해 추가 메모리 오버헤드가 필요합니다.
- 성능을 향상시키기 위해 Swift 컴파일러는 크기가 고정되어 있거나 수명을 예측할 수 있을 때 참조 타입을 스택에 할당하도록 선택할 수 있습니다. 이를 "Stack promotion"이라고 합니다.
- 참조 타입의 stack promotion이 발생하면 힙에 저장되지 않고 값 타입처럼 스택에 직접 저장됩니다.
- 스택의 데이터에 접근하는 것이 힙의 데이터에 접근하는 것보다 빠르기 때문에 성능이 향상됩니다.

```swift
class Point {
    var x: Int
    var y: Int
    
    init(x: Int, y: Int) {
        self.x = x
        self.y = y
    }
}

func someFunction() {
    var point = Point(x: 42, y: 43)
    // ...
}
```

- 만약 `Point`가 메서드의 지역변수로 사용된다면 스위프트의 컴파일러는 stack promotion을 진행합니다.
- 이유는 사이즈가 고정되어있고 생명주기가 뻔히 예상되기 때문입니다.
- 이 결과는 메모리에 대한 빠른 접근과 퍼포먼스 향상으로 이어집니다.
- 예시를 보시다시피 reference type인 `Point`는 stack promotion이 이뤄졌습니다. `someFunction`메서드 안에서 지역 변수로 사용되었기 때문에 수명을 예측할 수 있고 x와 y의 크기가 고정되어 있기 때문이죠.
  - `Int`는 값이 무엇이 되든 고정된 사이즈를 갖습니다. 구동되는 환경에 따라 다른 사이즈를 갖는데, 32-bit에서는 4bytes, 64-bit에서는 8bytes를 갖습니다.



<br/>

## 출처

- https://developer.apple.com/swift/blog/?id=10
- https://www.vadimbulavin.com/value-types-and-reference-types-in-swift/
- https://www.tirupatibalan.com/2019/04/01/process-threads-queues-in-ios.html