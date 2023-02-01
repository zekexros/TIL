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

## 출처

- https://developer.apple.com/swift/blog/?id=10