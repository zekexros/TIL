# Capture List



- 일반적으로, 클로저는 외부에 있는 상수나 변수의 값들을 내부에서 사용하면 <u>강한 참조</u>로 캡쳐를 합니다(aka strong capturing).
- 이 때, 명시적으로 <u>캡쳐 리스트</u>를 사용한다면 클로저 안으로만 제한되는 캡쳐를 할 수 있습니다.
- 주로 강한 순환 참조를 피하기 위해 캡쳐 리스트를 사용합니다.



<br/>



```swift
var a = 0
var b = 0
let closure = { [a] in
    print(a, b)
}

a = 10
b = 10
closure()
// Prints "0 10"
```

- 클로저가 생성될 때 캡쳐 리스트의 항목들이 초기화 됩니다.
- 위의 a는 클로저의 캡쳐 리스트를 사용하였습니다. 클로저 내부로 제한된 캡쳐로, a에 대해 클로저 내부에서 변경을 하여도 외부에 영향이 없고 반대로 외부에서 변경을 하여도 클로저 내부에는 영향이 없습니다.
- 하지만 b는 캡쳐되지 않았으므로 외부 혹은 내부에서 값을 변경하면 항상 영향을 받습니다.



## Reference Semantics

- 하지만 캡쳐하려는 변수의 타입이 reference타입(예를 들어 class)들이라면 얘기는 달라집니다.



<br/>



```swift
class SimpleClass {
    var value: Int = 0
}
var x = SimpleClass()
var y = SimpleClass()
let closure = { [x] in
    print(x.value, y.value)
}

x.value = 10
y.value = 10
closure()
// Prints "10 10"
```

- x에 대해 외부에선 변수로, 클로저 내부에선 상수로 있습니다. 그러나 x는 SimpleClass 타입으로 class로 만들어졌습니다. reference semantics이라는 뜻이죠. 두 값 모두 같은 객체를 참조합니다.
- 따라서 캡쳐 리스트를 사용해도 외부에서 값을 변경하면 내부에도 영향을 끼칩니다. 그 반대도 마찬가지구요.



<br/>

```swift
myFunction { print(self.title) }                    // implicit strong capture
myFunction { [self] in print(self.title) }          // explicit strong capture
myFunction { [weak self] in print(self!.title) }    // weak capture
myFunction { [unowned self] in print(self.title) }  // unowned capture
```

- 값이 타입이 class인 경우 캡쳐 리스트 안에 weak 혹은 unowned 키워드를 사용할 수 있습니다.
- weak 혹은 unowned 키워드를 통해 값을 reference count를 늘리지 않게 참조하도록 캡쳐할 수 있습니다.
- struct 타입은 weak 혹은 unowned 키워드와 같이 캡쳐 할 수 없습니다. 쓰면 아래와 같은 컴파일 에러를 보게됩니다.

<img width="665" alt="CleanShot 2023-01-06 at 14 43 54@2x" src="https://user-images.githubusercontent.com/42647277/210942552-04e2638e-7e47-4b84-800f-f5f845f86f82.png">



<br/>



```swift
// Weak capture of "self.parent" as "parent"
myFunction { [weak parent = self.parent] in print(parent!.title) }
```

- 캡쳐 리스트를 통해 클로저 내부에서만 한정적으로 사용되는 변수의 네이밍도 가능합니다.

<br/>



## Weak capturing

- <u>Strong capturing</u>를 피하기 위한 가장 일반적인 방법은 <u>weak capturing</u>입니다.
- 약하게 캡쳐된 값이 만약 할당에서 해제되면 nil로 지정됩니다.
- 그렇기에 약하게 캡쳐된 값은 항상 <u>옵셔널</u> 타입입니다.



<br/>



## Unowned capturing

- weak을 대안하는 다른 방법은 unowned입니다. 옵셔널이 언래핑 된 것처럼 작동합니다.
- unowned로 참조하는 객체는 항상 메모리에 할당되어있다고 가정하기에 옵셔널 타입도 아니고 당연히 nil이 할당되지도 않습니다.
- 사용하기 편한만큼 unowned키워드를 사용할 때에는 주의해야 합니다. 만약 클로저가 실행될 때 unowned로 캡쳐한 값이 할당에서 해제된 상태라면, 런타임 에러가 발생하며 앱이 죽게됩니다.



<br/>



## 갖고 놀다가 알게된 사실

1. Class타입의 객체 안에 Class타입의 프로퍼티를 캡쳐하면 클로저 내부에서 변경할 수 있을까 없을까? 답은 변경할 수 없다.<img width="889" alt="CleanShot 2023-01-06 at 15 41 13@2x" src="https://user-images.githubusercontent.com/42647277/211140779-2269107a-e609-4101-bffe-a905339093da.png">

   

2. Struct타입의 객체 안에 Class타입의 변수를 캡쳐하면 클로저 내부에서 변경할 수 있을까 없을까? 답은 변경할 수 없다.<img width="913" alt="CleanShot 2023-01-06 at 15 45 31@2x" src="https://user-images.githubusercontent.com/42647277/210945725-4efba419-e504-4917-bf84-e4024f092ac6.png">

3. 객체의 내부 변수를 캡쳐하려고 하면 항상 이름을 지정해줘야 한다.<img width="884" alt="CleanShot 2023-01-06 at 15 36 25@2x" src="https://user-images.githubusercontent.com/42647277/210944484-14dd8582-ae6b-42b9-ab99-d110523d7531.png">



- 1,2 번으로 미뤄볼 때, 객체의 내부에 존재하는 변수 혹은 상수를 캡쳐하면 value, reference와 상관없이 아예 값 변경이 불가능하다.

- 그리고 내부 객체 내부의 변수를 캡쳐하려면 항상 이름을 지정해주자.

- 캡쳐 리스트 사용하는 목적이 대상이 value타입이냐 reference타입이냐에 따라 다르다.

  - value타입은 상수로 캡쳐하는 것으로 주로 클로저 외부로부터 영향을 전혀 받지 않도록 사용하기 위해 캡쳐 리스트를 사용한다.
  - reference타입은 주로 weak 혹은 unowned를 통해 강한 순환 참조를 피하기 위해 캡쳐 리스트를 사용한다.

  

<br/>



## 출처

- https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html
- https://docs.swift.org/swift-book/ReferenceManual/Expressions.html#ID544
- https://www.hackingwithswift.com/articles/179/capture-lists-in-swift-whats-the-difference-between-weak-strong-and-unowned-references
- https://www.swiftbysundell.com/articles/swifts-closure-capturing-mechanics/
